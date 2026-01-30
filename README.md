# MS Planner Powerbi Hierarchy Report
Microsoft Planner → Dataverse → Power BI analytics solution featuring multi‑level task hierarchy (up to 6 levels), project progress tracking, user workload insights, and real‑time dashboards powered by DirectQuery. Ideal for PMO teams, task management, and Planner-based project tracking.

>This repository contains a dynamic Power BI report built on top of Microsoft Planner data stored in Dataverse. The report enables users to track tasks, hierarchies, progress, assignments, and project timelines across multiple MS Planner projects — all in real time through DirectQuery.

# Detail Description 
### A real‑time Power BI analytics solution for Microsoft Planner using Dataverse and DirectQuery.
>This report visualizes multi‑level task hierarchies, user assignments, project progress, and Gantt‑style scheduling. Designed for organizations using Planner for project management, it enables dynamic dashboards, workload insights, and hierarchy navigation up to 6 levels — all without importing data.
Features include:

* Multi‑level parent–child task hierarchy (Levels 1–6)  
* Real‑time task status (Complete, On Track, Late)
* Automatic user‑based project filtering
* Resource assignment & workload analytics
* Calendar/Gantt visualization
* DirectQuery-optimized SQL for hierarchy handling
* Dataverse integration with Planner, Tasks, Assignments & Users

<img width="1536" height="1024" alt="Designer" src="https://github.com/user-attachments/assets/7dcba84b-41a6-4616-a65b-8099e3cd21c7" />

# 🚀 Overview
Many organizations use Microsoft Planner for team collaboration, but Planner’s native reporting is limited. This BI report solves that by enabling:

* Real‑time task monitoring
* Parent–child hierarchical structures (up to 6 levels)
* Project‑wise progress tracking
* Assignments per user
* Calendar & Gantt‑style visualizations
* Task-On-Time vs Late vs Completed analysis

### All visuals dynamically filter based on the projects the user is assigned to.

# 🗂 Data Source
This report uses Dataverse tables (through DirectQuery), specifically:
```
├── msdyn_projecttask 
 └── Tasks, Parenttasks, Project, Progress & Start&End Date  
├── msdyn_resourceassignment 
 └── User-to-task mapping
```

### These tables are joined to create a complete view of Planner activity.

# 🔧 Key Features
## 1. Dynamic Planner Dashboard Shows:
* Total tasks
* Tasks per project
* Completed vs On Track vs Late
* Task age & schedule

### 2. Hierarchy Visualization (Levels 1–6)
Since DirectQuery does not support CTEs or recursive SQL, hierarchy levels are generated using simple SQL CASE logic, allowing:  
* Parent tasks 
* Subtasks 
* Deep nested structures  

### 3. Gantt‑Style Calendar Page
A calendar view showing:

* Start dates 
* Due dates 
* Task progress 

>Note: Calendar view works only for tasks with valid start/end dates.

### 4. User-Based Access
Every user sees only:
* Projects they are assigned to 
* Tasks they own 
* Subtasks underneath their assigned tasks 

This provides natural row-level filtering without RLS configuration.

# 📐 Technical Structure
```
Model Highlights
├──DirectQuery mode for real-time data  
├──Normalized tables: Tasks → Assignments → Users  
├──Additional SQL computed columns:  
 └──Level_01, Level_02, … Level_06  
 ├──Child_Task (leaf detection)   
 ├──Task_Track_Status (Complete / On Track / Late)  
```
# Performance Considerations

* No recursive SQL  
* No CTE  
* No complex calculated tables  
* Lightweight DAX for DirectQuery compatibility  

# 🖼 Pages in the Report

### Dashboard 
High-level project and task KPIs

### Project Task 
DetailsDrill-down hierarchy with visual indicators

### Assignments
Tasks per user, workload overview

### Gantt
Project Calendar layout based on schedule dates with Assignee
