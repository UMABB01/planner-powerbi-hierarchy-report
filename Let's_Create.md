# LET's create 

## MS Planner BI Report

> Requirements
* PowerBI Desktop
* MS Planner Premium (MS Project) - Plan
* Dataverse
* PowerBI Service (for File upload - consume)

---
## DATA uploading
> Note that you must check whether MS Planner data is present in your Organization, if its on your work id you can check by directly connecting to dataverse from Power BI and Search for `msdyn_projecttask` and `msdyn_resourceassignment` and also get your Organizational SQL Server Link, it will be something like this `[YourOrganization].crm4.dynamics.com` and Database which is normally `dbo` (might be different in some cases).

> Login with your work/personel Microsoft Id depending where is MS Planner DATA (In PowerBI or if Prompted)

### Part 1/2 of data uploading. 

* Add Data source in PowerBI  
  └── `SQL Server database`
* use Server as `[YourOrganization].crm4.dynamics.com`  
* use Database as `dbo` (might be different in some cases)  
use below SQL 
```
SELECT
    t.msdyn_project,
    t.msdyn_projectname,  
    t.msdyn_projecttaskid,
    t.msdyn_subject, 
    t.msdyn_parenttask,
    t.msdyn_parenttaskname, 
    t.msdyn_outlinelevel,
    t.msdyn_progress, 
    t.msdyn_descriptionplaintext,
    t.msdyn_scheduledstart,
    t.msdyn_scheduledend, 
    t.msdyn_duration, 
    t.msdyn_effort, 
    t.msdyn_effortcompleted, 
    t.msdyn_effortremaining, 
    t.msdyn_priority,

    /* Level_01 (topmost/root) */
    CASE 
        WHEN p6.msdyn_projecttaskid IS NOT NULL THEN p6.msdyn_subject
        WHEN p5.msdyn_projecttaskid IS NOT NULL THEN p5.msdyn_subject
        WHEN p4.msdyn_projecttaskid IS NOT NULL THEN p4.msdyn_subject
        WHEN p3.msdyn_projecttaskid IS NOT NULL THEN p3.msdyn_subject
        WHEN p2.msdyn_projecttaskid IS NOT NULL THEN p2.msdyn_subject
        WHEN p1.msdyn_projecttaskid IS NOT NULL THEN p1.msdyn_subject
        ELSE t.msdyn_subject
    END AS Level_01,

    /* Level_02 */
    CASE 
        WHEN p6.msdyn_projecttaskid IS NOT NULL THEN p5.msdyn_subject
        WHEN p5.msdyn_projecttaskid IS NOT NULL THEN p4.msdyn_subject
        WHEN p4.msdyn_projecttaskid IS NOT NULL THEN p3.msdyn_subject
        WHEN p3.msdyn_projecttaskid IS NOT NULL THEN p2.msdyn_subject
        WHEN p2.msdyn_projecttaskid IS NOT NULL THEN p1.msdyn_subject
        WHEN p1.msdyn_projecttaskid IS NOT NULL THEN t.msdyn_subject
        ELSE NULL
    END AS Level_02,

    /* Level_03 */
    CASE 
        WHEN p6.msdyn_projecttaskid IS NOT NULL THEN p4.msdyn_subject
        WHEN p5.msdyn_projecttaskid IS NOT NULL THEN p3.msdyn_subject
        WHEN p4.msdyn_projecttaskid IS NOT NULL THEN p2.msdyn_subject
        WHEN p3.msdyn_projecttaskid IS NOT NULL THEN p1.msdyn_subject
        WHEN p2.msdyn_projecttaskid IS NOT NULL THEN t.msdyn_subject
        ELSE NULL
    END AS Level_03,

    /* Level_04 */
    CASE 
        WHEN p6.msdyn_projecttaskid IS NOT NULL THEN p3.msdyn_subject
        WHEN p5.msdyn_projecttaskid IS NOT NULL THEN p2.msdyn_subject
        WHEN p4.msdyn_projecttaskid IS NOT NULL THEN p1.msdyn_subject
        WHEN p3.msdyn_projecttaskid IS NOT NULL THEN t.msdyn_subject
        ELSE NULL
    END AS Level_04,

    /* Level_05 */
    CASE 
        WHEN p6.msdyn_projecttaskid IS NOT NULL THEN p2.msdyn_subject
        WHEN p5.msdyn_projecttaskid IS NOT NULL THEN p1.msdyn_subject
        WHEN p4.msdyn_projecttaskid IS NOT NULL THEN t.msdyn_subject
        ELSE NULL
    END AS Level_05,

    /* Level_06 */
    CASE 
        WHEN p6.msdyn_projecttaskid IS NOT NULL THEN p1.msdyn_subject
        WHEN p5.msdyn_projecttaskid IS NOT NULL THEN t.msdyn_subject
        ELSE NULL
    END AS Level_06,

    /* --- Task tracking status & color --- */
    CASE
        WHEN t.msdyn_progress >= 1 THEN 'Complete'
        WHEN t.msdyn_scheduledend >= CAST(GETDATE() AS date) THEN 'ON_Track'
        ELSE 'LATE'
    END AS Task_Track_Status,

    CASE
        WHEN t.msdyn_progress >= 1 THEN '#bfdef6'  -- Complete
        WHEN t.msdyn_scheduledend >= CAST(GETDATE() AS date) THEN '#85ceb3'  -- ON_Track
        ELSE '#ea9d9a'  -- LATE
    END AS Task_Track_Status_Color,

    /* --- Child_Tasks: 1 if task has a parent AND has no children --- */
    CASE 
        WHEN t.msdyn_parenttask IS NOT NULL
             AND NOT EXISTS (
                 SELECT 1
                 FROM dbo.msdyn_projecttask AS c
                 WHERE c.msdyn_parenttask = t.msdyn_projecttaskid
             )
        THEN CAST(1 AS bit)
        ELSE CAST(0 AS bit)
    END AS Child_Tasks

FROM dbo.msdyn_projecttask AS t

/* Self-joins to traverse up to 6 ancestor levels */
LEFT JOIN dbo.msdyn_projecttask AS p1
    ON t.msdyn_parenttask = p1.msdyn_projecttaskid
LEFT JOIN dbo.msdyn_projecttask AS p2
    ON p1.msdyn_parenttask = p2.msdyn_projecttaskid
LEFT JOIN dbo.msdyn_projecttask AS p3
    ON p2.msdyn_parenttask = p3.msdyn_projecttaskid
LEFT JOIN dbo.msdyn_projecttask AS p4
    ON p3.msdyn_parenttask = p4.msdyn_projecttaskid
LEFT JOIN dbo.msdyn_projecttask AS p5
    ON p4.msdyn_parenttask = p5.msdyn_projecttaskid
LEFT JOIN dbo.msdyn_projecttask AS p6
    ON p5.msdyn_parenttask = p6.msdyn_projecttaskid;
```

### Part 2/2 of data uploading
* Add Data source in PowerBI  
  └── SQL Server database
* use Server as `[YourOrganization].crm4.dynamics.com`  
* use Database as `dbo` (might be different in some cases)  
use below SQL   
```
SELECT
msdyn_taskid,
msdyn_bookableresourceidname
FROM dbo.msdyn_resourceassignment
```

## DATA Uploading Completed
----

## DATA Modeling 

* Create a `one-to-many` relationship from column `msdyn_projecttaskid` of Table `msdyn_projecttask` to column `msdyn_taskid` of Table `msdyn_resourceassignment`  
* Rename Table `msdyn_projecttask` to `Tasks_Hierarchy`  
* Rename Table `msdyn_resourceassignment` to `Assignee_Users`  

# Create Measures

> Hierarchy_Max_Depth : for defining Final depth of Hierarchy
```
Hierarchy_Max_Depth = MAX(Tasks_Hierarchy[msdyn_outlinelevel])
```
  
    
> Hierarchy_Depth :  finding each task depth
```
Hierarchy_Depth = 
ISINSCOPE(Tasks_Hierarchy[Level_06])+
ISINSCOPE(Tasks_Hierarchy[Level_05])+
ISINSCOPE(Tasks_Hierarchy[Level_04])+
ISINSCOPE(Tasks_Hierarchy[Level_03])+
ISINSCOPE(Tasks_Hierarchy[Level_02])+
ISINSCOPE(Tasks_Hierarchy[Level_01])
```
  
  
> All_Hierarchy_Task_Progress : I only wanted the Leaf Tasks, that define's the whole Parent Duration and progress
```
All_Hierarchy_Task_Progress = 
VAR _Progress_val =
DIVIDE(
    SUMX(
        FILTER(
            Tasks_Hierarchy,
            Tasks_Hierarchy[msdyn_progress] = 1
                && Tasks_Hierarchy[Child_Tasks] = TRUE()
        ),
        IF(Tasks_Hierarchy[msdyn_duration]=0,1,Tasks_Hierarchy[msdyn_duration])
    ),
    SUMX(
        FILTER(
            Tasks_Hierarchy,
            Tasks_Hierarchy[Child_Tasks] = TRUE()
        ),
        IF(Tasks_Hierarchy[msdyn_duration]=0,1,Tasks_Hierarchy[msdyn_duration])
    ),
    0
)

RETURN
IF(ISBLANK(_Progress_val),0,_Progress_val)
```

>Hierarchy_Task_Progress : Progress 
```
Hierarchy_Task_Progress = 
VAR _val = [All_Hierarchy_Task_Progress]
VAR _show = [Hierarchy_Depth] <= [Hierarchy_Max_Depth]
VAR _result = IF(_show,_val)
RETURN
_result
```


>Hierarchy_Total_Task : Tasks Count with all the black sections removed
```
Hierarchy_Total_Task = 
VAR _val = COUNTX(FILTER(Tasks_Hierarchy,Tasks_Hierarchy[Child_Tasks]=TRUE()),Tasks_Hierarchy[msdyn_projecttaskid])
VAR _show = [Hierarchy_Depth] <= [Hierarchy_Max_Depth]
VAR _result = IF(_show,_val)
RETURN
_result
```


>Pending_Task_Progress : for Donut Chart
```
Pending_Task_Progress = 1-[Hierarchy_Task_Progress]
```


>Projects_Count : Total Projects
```
Projects_Count = DISTINCTCOUNTNOBLANK(Tasks_Hierarchy[msdyn_projectname])
```


>Task_DurationDays : Days for Each Tasks with with all the black sections removed
```
Task_DurationDays = 
VAR _DateStart = FIRSTNONBLANK(Tasks_Hierarchy[msdyn_scheduledstart],DATEVALUE(Tasks_Hierarchy[msdyn_scheduledstart]))
VAR _DateEnd = LASTNONBLANK(Tasks_Hierarchy[msdyn_scheduledend],DATEVALUE(Tasks_Hierarchy[msdyn_scheduledend]))

VAR _val = DATEDIFF(_DateStart,_DateEnd,DAY)
VAR _show = [Hierarchy_Depth] <= [Hierarchy_Max_Depth]
VAR _result = IF(_show,_val)
RETURN
_result
```


>Task_EndDate : EndDate of each Task with all the black sections removed
```
Task_EndDate = 
VAR _val = LASTNONBLANK(Tasks_Hierarchy[msdyn_scheduledend],DATEVALUE(Tasks_Hierarchy[msdyn_scheduledend]))
VAR _show = [Hierarchy_Depth] <= [Hierarchy_Max_Depth]
VAR _result = IF(_show,_val)
RETURN
_result
```
  
  
>Task_StartDate : StartDate of each Task with all the black sections removed
```
Task_StartDate = 
VAR _val = FIRSTNONBLANK(Tasks_Hierarchy[msdyn_scheduledstart],DATEVALUE(Tasks_Hierarchy[msdyn_scheduledstart]))
VAR _show = [Hierarchy_Depth] <= [Hierarchy_Max_Depth]
VAR _result = IF(_show,_val)
RETURN
_result
```


>Tasks_Completed :  for multi Card 
```
Tasks_Completed = COUNTX(FILTER(Tasks_Hierarchy,Tasks_Hierarchy[msdyn_progress]>=1),Tasks_Hierarchy[msdyn_projecttaskid])
```


>Tasks_Pending : for multi Card 
```
Tasks_Pending = COUNTX(FILTER(Tasks_Hierarchy,Tasks_Hierarchy[msdyn_progress]<1),Tasks_Hierarchy[msdyn_projecttaskid])
```


>Tasks_Pending_Late : Tasks with passed due date - for multi Card 
```
Tasks_Pending_Late = COUNTX(FILTER(Tasks_Hierarchy,Tasks_Hierarchy[Task_Track_Status]="LATE"),Tasks_Hierarchy[msdyn_projecttaskid])
```


>Tasks_Pending_OnTrack : Tasks having time till due date - for multi Card 
```
Tasks_Pending_OnTrack = COUNTX(FILTER(Tasks_Hierarchy,Tasks_Hierarchy[Task_Track_Status]="On_Track"),Tasks_Hierarchy[msdyn_projecttaskid])
```


>TaskTrack_Status : For Gantt Table
```
TaskTrack_Status = 
VAR _Today = TODAY()
VAR _DateEnd = LASTNONBLANK(Tasks_Hierarchy[msdyn_scheduledend],DATEVALUE(Tasks_Hierarchy[msdyn_scheduledend]))
VAR _Progress = [All_Hierarchy_Task_Progress]

VAR _val = SWITCH(TRUE(),
_Progress>= 1,"Complete",
 _DateEnd<_Today, "LATE",
 _DateEnd>=_Today,"ON_Track",0)
VAR _show = [Hierarchy_Depth] <= [Hierarchy_Max_Depth]
VAR _result = IF(_show,_val)
RETURN
_result
````


>TaskTrack_Status_Color : For Gantt Table
````
TaskTrack_Status_Color = 
VAR _Today = TODAY()
VAR _DateEnd = LASTNONBLANK(Tasks_Hierarchy[msdyn_scheduledend],DATEVALUE(Tasks_Hierarchy[msdyn_scheduledend]))
VAR _Progress = [All_Hierarchy_Task_Progress]

VAR _val = SWITCH(TRUE(),
_Progress>= 1,1, //Complete
 _DateEnd<_Today, -1, // LATE
 _DateEnd>=_Today,0,//ON_Track
 0) 
VAR _show = [Hierarchy_Depth] <= [Hierarchy_Max_Depth]
VAR _result = IF(_show,_val)
RETURN
_result
````

Watch this video for better understanding of BI Hierarchy `https://youtu.be/iwRqSl-_zvU`

## DATA Modeling Completed
----

## Report View

> Note that the data has been removed from pbix file as its DirectQuery, Add the above data files and measures by same names to get results.

<img width="1307" height="733" alt="image" src="https://github.com/user-attachments/assets/af5ac4e8-44bc-4e11-8bb2-4a41c59624a3" />

<img width="1303" height="733" alt="image" src="https://github.com/user-attachments/assets/92ca066b-8535-426c-9828-cafe4c78b3c9" />

<img width="1307" height="737" alt="image" src="https://github.com/user-attachments/assets/3c7aaa80-36a3-402c-ba4f-58881b20daff" />


