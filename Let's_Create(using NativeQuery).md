### This is not recommended method but if you want to use PowerQuery Steps. here is the following ...  

# LET's create 

## MS Planner BI Report using Native Query

> Requirements
* PowerBI Desktop
* MS Planner Premium (MS Project) - Plan
* Dataverse
* PowerBI Service (for File upload - consume)

---
## DATA uploading
> Note that you must check whether MS Planner data is present in your Organization, if its on your work id you can check by directly connecting to dataverse from Power BI and Search for `msdyn_projecttask` and `msdyn_resourceassignment`.

> Login with your work/personel Microsoft Id depending where is MS Planner DATA (In PowerBI or if Prompted)

### Part 1/2 of data uploading 

* Add Data source in PowerBI  
  └── `DATAVERSE`
* Navigate to the environment of your organization where the MS Planner data located.
* Right click on it and `Transform Data` 
* Connection Settings to `Direct Query`
* Navigate to `fx` near code section, it would add new step on right side of `Applied Step`

<img width="1190" height="57" alt="Screenshot 2026-02-04 223238" src="https://github.com/user-attachments/assets/e15dfef7-d87a-440a-b533-90d29c731e8d" />

* Add below code
```
= Value.NativeQuery(Source,"SELECT
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
    ON p5.msdyn_parenttask = p6.msdyn_projecttaskid",null,[EnableFolding=true])
```

> Change the name of this Query `Tasks_Hierarchy`
* This would allow you to add Power Query Steps ...

### Part 2/2 of data uploading 

* Add Data source in PowerBI  
  └── `DATAVERSE`
* Navigate to the environment of your organization where the MS Planner data located.
* Navigate to `msdyn_resourceassignment` 
* Remove all columns except :`msdyn_taskid` and `msdyn_bookableresourceidname`

> Change the name of this Query `Assignee_Users`

## DATA Uploading Completed
----

# The rest is same as "LET's Create"
