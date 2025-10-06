# Assignment 2

## Problem Statement

### Problem Domain: Task Management
Students often track to-do tasks across multiple platforms. For example, I use Notion or Reminders for to-do lists, as well as various learning management platforms for assignments. The general domain of this project is the concept of task management and how to make task management more seamless.

### Problem: Integration of Tasks Across Platforms

Coursework at universities including MIT commonly requires students to coordinate deadlines across multiple platforms: learning-management systems (eg. Canvas, Gradescope, Catsoop), code repositories (eg. GitHub), group chats, and personal calendars. This fragmentation creates cognitive load and missed/late submissions even for organized students. A focused tool that aggregates assignment deadlines across these sources and makes them easily accessible addresses a concrete productivity pain common to every course-taking student.

### Stakeholders

1. **Students**: *Primarily, students experience the identified problem, which is a fragmented task management workflow. A solution would positively impact students' task management because all to-dos are presented in one place, rather than being scattered across different websites.*
2. **Professors**: *Professors do not directly experience this problem, but a solution may cause them to see improved performance from students, and they may also feel less pressured to use a specific assignment/grading platform.*
3. **Existing Teaching Platforms**: *The disconnected nature of the landscape of existing task/learning management platforms contributes to the ientified problem. A solution that unifies these various platforms may reduce the ability of any specific platform to dominate the (perhaps highly profitable) teaching platform product landscape.*

### Evidence and Comparables
1. [Beyond fragmentation: integrating AI for a seamless learning experience](https://www.timeshighereducation.com/campus/beyond-fragmentation-integrating-ai-seamless-learning-experience) - This article notes that students "bounce between inconsistent experiences" across multiple systems and that fragmented learning systems negatively impact student performane.
2. [App Overload: Fragmented Digital Landscape Failing K-12 Education](https://www.eschoolnews.com/digital-learning/2025/04/28/too-many-apps-for-that-in-schools/) - This study of 100+ teachers and 125 parents reported 42% rated satisfaction at 5/10 or lower when using multiple educational apps. The article describes this as "app fatigue", which is decreased engagement due to needing to use too many different platforms.
3. [Impacts of digital technologies on education and transformation challenges](https://pmc.ncbi.nlm.nih.gov/articles/PMC9684747/) - This study shows that digital integration has an observable impact on student performance.
4. [State of Higher Ed LMS Market for US and Canada: Year-End 2024 Edition](https://onedtech.philhillaa.com/p/state-of-higher-ed-lms-market-for-us-and-canada-year-end-2024-edition) - A comparable showing that Canvas dominates the LMS platform market across institutions. Ideally, Canvas could act as a unifier, but it currently does not.
5. [LMS Integration: Enhancing Learning Management Systems](https://www.inkling.com/blog/2024/01/lms-integration-learning-management-systems/) - A comparable: Inkling attempts to integrate a LMS with other systems, such as allowing administrators to track student progress and providing SSO. However, it does not seem to be able to integrate tasks across multiple learning platforms.  

## Application Pitch

### Taskmate

Instead of losing time and sleep over assignment deadlines scattered across Canvas, Catsoop, GitHub repos, calendar invites, and group chats, studnets use Taskmate to gather everything into one clear timeline so they always know what to do next.

### Key Features

1. **Integrated Task Calendar**: This integrated calendar pulls tasks from Canvas, GitHub, Catsoop, Google Calendar and aggregates them into a single user interface.  
Why it helps: the integrated task calendar mitigates the identified problem because instead of hunting through different platforms, users get aggregated view of every task.
Impact on stakeholders: students reduce cognitive load and fewer deadlines slip through the cracks; instructors ensure deadlines are more accessible, existing calendar features on Canvas, etc. become more redundant.

2. **Daily To-Do List**: This checklist is drawn from the calendar daily and allows users to check off or snooze upcoming action items aggregated across different platforms.  
Why it helps: This conversts a scattered backlog of tasks into a easily understandable daily plan so students actually start tasks.  
Impact on stakeholders: students improve their planning, instructors see more on-time submissions, existing platforms' content becomes more unified.

3. **Task Prioritization**: Aggregates tasks across different platforms and ranks them by estimated time to complete, as well as user-chosen priority (such as importance to student, percentage of grade, etc.).  
Why it helps: Prioritization allows users to turn an overwhelming and scattered list of tasks into actionable steps and helps them improve time allocation.  
Impact on stakeholders: students are better able to make decisions on how to allocate time; instructors' recommended priorities or time estimations are more accessible, existing task management platforms may not have this feature, so this could be a competitive new feature.

## Concept Design

### Brief Overview

The key concepts of a task management system include Task, Assigner, TodoList, Priority, and Calendar. As a brief description of how the concepts play in the context of the app as a whole, Users are associated with TodoLists with scoped start and end dates (this is therefore reusable for both a daily to-do list and a longer calendar - A daily to-do list is a TodoList with the same start and end date, whereas a calendar is a TodoList within a day, week, or month). When an Assigner (eg. a course, or the user for a personal task) creates a Task, which represents a single action item, TodoLists are updated with the task and the associated Priority accordingly. Currently, I decided to base the Priority only on proximity to due date, but in later versions, it could incorporate percentage of grade, etc.

### Concepts

```
concept Task
purpose represents a single deliverable or action item that must be completed within a deadline
principle a task is the smallest useful unit for planning and is associated with the deliverable, due date, and completedness.

state
    a name String
    a description String
    a dueDate Time
    a completed Flag
    an overdue Flag

actions
    snoozeTask(taskName: String, newDate: Time): Task
        requires: requires: taskName is the same as the Task's name String and newDate is a time after the current dueDate
        effects: the dueDate associated with the Task is set to the new Time. return the updated Task
    
    completeTask(taskname: String): Task
        requires: taskName is the same as the Task's name String
        effects: the completed Flag is set to True. return the updated Task

    markLate(taskname: String): Task
        requires: requires: taskName is the same as the Task's name String and the completed Flag is False
        effects: the overdue Flag is set to True. return the updated Task
```

```
concept Assigner
purpose represents a category or source of tasks; for example, a course, a project, or an individual's personal to-dos.
principle each assigner is associated with a set of tasks, to which they can add or remove tasks

state
    a set of Tasks

actions
    addTask(name: String, description: String, dueDate: Time): (Task)
        effects: adds a Task with the specified name, description, and dueDate, along with the completed Task set to False, to the set of Tasks. returns the newly created Task
```

```
concept Priority
purpose assigns a numerical priority to a Task
principle a priority is assigned to a Task based on proximity to a given endDate and whether it is overdue.

state
    an endTime Time

actions
    getPriority(task: Task): Number
        effects: generates priority Number, which is based on the difference between the endTime and the Task's dueDate, as well as whether the Task's overdue Flag is True or False. If two priority Numbers were to be compared, it is guaranteed that the Number is greater when the time difference is smaller. The priority is also greater if the overdue Flag is True.
```

```
concept TodoList
purpose represents a list of tasks that need to be completed in a scoped time frame
principle tasks are added to the TodoList and can be marked as finished or unfinished by Users.

state
    a user User
    a startTime Time
    an endTime Time
    a uncompleted set of PrioritizedTasks with
        a Task
        a priority Number
    a completed set of Tasks
    a set of Assigners

actions
    addTask(user: User, assigner: Assigner, task: Task, priority: Number)
        requires: the user must be the same as the User associated with the TodoList; the Task's dueDate must occur after the TodoList's startTime and before the TodoList's endTime; the assigner must be in the set of Assigners associated with the TodoList; and the Task must be in the set of Tasks associated with the specified Assigner
        effects: adds the Task and Priority to the set of uncompleted Tasks

    updatePriority(user: User, task: Task, priority: Number)
        requires: the user must be the same as the User associated with the TodoList, the task must exist in the set of uncompletedTasks
        effects: the specified task's priority is updated with the new priority Number
    
    finishTask(user: User, task: Task)
        requires: the user must be the same as the User associated with the TodoList, the task must be in the set of uncompleted Tasks, and the completed Flag associated with the Task must be True
        effects: removes the task from the set of uncompleted PrioritizedTasks and adds the task to the set of completed Tasks

    clearCompleted(user: User):
        effects: clears set of completed Tasks

    getOverdue(user: User): (set(Task))
        requires: the user must be the same as the User associated with the TodoList
        effects: returns the set of uncompleted Tasks whose dueDate is at or before endTime.

    getSortedItems: list(Task)
        effects: returns a list of Tasks sorted from highest priority Number to lowest priority Number
```

### Basic Syncs

The first sync occurs when a new task created by a User appears, which is either a personal task, or pulled from an existing learning management system, eg. Canvas.
```
sync integrateNewTask
  when Request.newTask(source: Assigner, name, description, dueDate)
  then Assigner.addTask(name, description, dueDate):
```

Also, the priority is generated.
```
sync getTaskPriority
  when Request.newTask()
  when Assigner.addTask(): (Task)
  then Priority.getPriority(Task)
```

Then, the Task and the priority are added to the User's TodoList.
```
sync addToDoList
  when Request.newTask(User, source: Assigner)
  when Assigner.addTask(): (Task)
  when Priority.getPriority(): (priority: Number)
  then TodoList.addTask(User, Assigner, Task, priority)
```

If a user completes a task:
```
sync completeTask
  when Request.completeTask(taskName)
  then Task.completeTask(taskName)
```

Then, the User's TodoList is updated:
```
sync completeToDo
  when Request.completeTask(User)
  when Task.completeTask(): (Task)
  then TodoList.finishTask(User, Task)
```

At the end of every day, the TodoList's completed tasks are cleared.
```
sync endOfDayClear
  when Request.endOfDay(User)
  then TodoList.clearCompleted(User)
```

At the same time, the TodoList, a sync is performed for items on the todo-list that are incomplete and overdue.
```
sync overDueToDo
  when Request.endOfDay(User)
  then TodoList.getOverdue(User): (set(Task))
```

Each overdue Task is then marked overdue.
```
sync updateOverDue
  when Request.endOfDay()
  when TodoList.getOverdue(): (set(Task))
  then Task.markOverdue(taskname)
```

Their priority is recalculated.
```
sync handleOverduePriority
  when Request.endOfDay()
  when TodoList.getOverdue(): (set(Task))
  when Task.markOverdue(taskname)
  then Priority.getPriority(Task)
```

And the priority in the TodoList is updated accoridngly.
Their priority is recalculated.
```
sync handleOverduePriority
  when Request.endOfDay()
  when TodoList.getOverdue(): (set(Task))
  when Task.markOverdue(taskname)
  when Priority.getPriority(Task): Number
  then TodoList.updatePriority(Task, Number)
```

If the user requests to look at the TodoList, the items are returned in prioritized form:
```
sync requestTodoList
  when Request.viewTodo(User)
  when TodoList.getSortedItems(User)
```

## UI Sketches

Calendar View:
![Calendar View](https://media.discordapp.net/attachments/1182438564030578782/1424602700116529192/Note_Oct_5_2025.jpg?ex=68e48c27&is=68e33aa7&hm=d3ba08ee17d933ebd528846884ca9fbbc41a085bccb2ac7d2f0b09578ae0bda9&=&format=webp&width=1920&height=1182)

Daily To-Do List:
![To-Do List](https://media.discordapp.net/attachments/1182438564030578782/1424602716809728011/Fig_1.jpeg?ex=68e48c2b&is=68e33aab&hm=8d176931d64fe5163fde31920a4a89e2a5629412f5faabb37d3c032de7deef4e&=&format=webp&width=1892&height=1370)

## User Journey
A student realizes on a Sunday night that three task deadlines landed in the last week: a Github lab due Sunday, a Pset on Catsoop due last Thursday, and a soccer club scrimmage. He's tired of having to juggle five courses, scattered notifications, and a crowded calendar has started to cost his sleep and focus.

He opens **Taskmate** to sync his various assignment sources. Taskmate parses each feed, from Canvas to Google Calendar, and creates Tasks. It shows a combined view on the Calendar Dashboard and also displays a Daily To-Do List so he can see exactly what he needs to do and by when. 

The next morning Taskmate runs its Daily To-Do flow and presents the Daily To-Do: a short checklist of three items ranked by deadline Priority. The top item is an overdue Pset from last Thursday. Next is the 6.104 Pset due next Sunday, and the next IMS pset on Canvas, also due next Sunday. He decides to add a Laundry task and also crosses off 6.104 once he finishes it. 

The result is the student no longer misses small deadline changes, and his stress drops because Taskmate turned scattered feeds into a tiny, trustworthy daily plan. 
