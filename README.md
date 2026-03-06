# DISCOVERY-NOTES


## Task Manager Codebase Exploration Summary

## 1. Initial vs. Final Understanding of the Codebase

### Initial Understanding

When I first reviewed the project, my understanding was limited to what I could gather from the README and a quick scan of the project files. I believed the project was a simple command-line application written in Python that allows users to manage tasks. From the directory structure, it seemed that the application was organized into a few key files that handled the command-line interface, data models, and storage.

At this stage, my assumptions were:

* The project uses Python with only the standard library.
* The application is executed through a CLI using commands like `create` and `list`.
* Tasks are stored in a local JSON file.
* The code structure appeared simple but I was unsure how responsibilities were divided between files.

However, I was uncertain about how the business logic was organized and how the different parts of the system communicated with each other.

### Final Understanding

After analyzing the code and using AI prompts to explore the project structure, feature implementation, and domain model, I developed a clearer understanding of the architecture and responsibilities of each component.

The project follows a **simple layered architecture**:

CLI Layer → Business Logic Layer → Storage Layer → JSON Data File

The key components are:

* **cli.py**: Acts as the entry point of the application. It parses command-line arguments and calls the appropriate methods in the TaskManager class.
* **TaskManager class**: Contains the core business logic of the application. It handles operations such as creating tasks, updating tasks, filtering tasks, and generating statistics.
* **models.py**: Defines the domain entities used in the application, including `Task`, `TaskStatus`, and `TaskPriority`.
* **storage.py**: Responsible for reading and writing task data to the `tasks.json` file.
* **tasks.json**: Serves as the local data store for all tasks.

The `Task` entity is the central object in the system and represents a unit of work with attributes such as title, description, status, priority, due date, and tags.

---

## 2. Most Valuable Insights from Each Prompt

### Project Structure Prompt

The project structure prompt helped me quickly identify the architecture and entry points of the application. I learned that even in a small codebase, it is important to recognize how responsibilities are separated between layers such as the CLI interface, business logic, and data storage. This made it easier to understand where new features should be implemented.

### Feature Location Prompt

The feature location prompt helped me understand how to trace a feature through the codebase. Instead of randomly reading files, I learned to follow the execution flow from the CLI command to the business logic and then to the storage layer. This approach made it easier to identify where changes would need to be made when implementing a new feature.

### Domain Understanding Prompt

The domain model prompt helped clarify the business concepts represented in the code. I learned that the system is built around the `Task` entity and uses enumerations for `TaskStatus` and `TaskPriority` to enforce valid states and priority levels. Understanding the domain model made it easier to connect the code to the actual functionality experienced by users.

---

## 3. Approach to Implementing the New Business Rule

The new business rule states that tasks overdue for more than seven days should automatically be marked as **abandoned**, unless they have a high priority.

To implement this rule, I would take the following approach:

1. **Update the Domain Model**

   * Add a new status called `ABANDONED` to the `TaskStatus` enum in `models.py`.

2. **Add Business Logic**

   * Implement logic in the `TaskManager` class to check for tasks that are overdue by more than seven days.
   * Ensure that tasks with high or urgent priority are excluded from this rule.

3. **Update Task Processing**

   * Modify the logic used when listing or retrieving tasks so that overdue tasks can automatically be updated to `ABANDONED`.

4. **Persist Changes**

   * Ensure that updates to task status are saved using the `TaskStorage` class so the changes are written to `tasks.json`.

Before implementing the rule, I would ask the team several questions, such as whether abandoned tasks should still be editable, how they should appear in statistics, and whether the rule should run automatically or only when tasks are retrieved.

---

## 4. Strategies for Approaching Unfamiliar Codebases

Through this exercise, I developed several strategies for exploring unfamiliar projects effectively.

First, I learned to start by examining the **directory structure and configuration files** to understand the overall architecture before reading individual lines of code.

Second, identifying the **entry point of the application** is important because it helps trace how user input flows through the system.

Third, understanding the **domain model** is critical since the entities and their relationships often reveal the main purpose of the application.

Fourth, tracing a **single feature from start to finish** is an effective way to understand how different components interact.

Finally, using structured AI prompts helped guide my investigation by encouraging me to form hypotheses, ask targeted questions, and validate my understanding.

In the future, I plan to apply this structured exploration process whenever I encounter a new codebase to build an accurate mental model more quickly.

# CODE UNDERSTANDING JOURNAL - TASK MANAGER

Language: Python
Date: 2026-03-05
Author: [Simphiwe]

## PART 1: UNDERSTANDING TASK CREATION and STATUS UPDATES

Key Files:

File	Purpose
task_manager.py	Handles task creation, updates, and business logic
models.py	Defines domain entities: Task, TaskStatus, TaskPriority
storage.py	Persists tasks to tasks.json

### RELEVANT CODE SNIPPET – TASK CREATION:

def create_task(self, title, description="", priority_value=2,
               due_date_str=None, tags=None):
    priority = TaskPriority(priority_value)
    due_date = datetime.strptime(due_date_str, "%Y-%m-%d") if due_date_str else None
    task = Task(title, description, priority, due_date, tags)
    task_id = self.storage.add_task(task)
    return task_id

RELEVANT CODE SNIPPET – STATUS UPDATE:

def update_task_status(self, task_id, new_status_value):
    new_status = TaskStatus(new_status_value)
    task = self.storage.get_task(task_id)
    if new_status == TaskStatus.DONE:
        task.mark_as_done()
    self.storage.update_task(task_id, status=new_status)
    self.storage.save()

### EXECUTION FLOW – TASK CREATION:

CLI Command → TaskManager.create_task() → Task object created → TaskStorage.add_task() → tasks.json updated

EXECUTION FLOW – TASK CREATION:

CLI Command → TaskManager.update_task_status() → Task.mark_as_done() → TaskStorage.save()

DESIGN PATTERNS OBSERVED:

Service Layer: TaskManager separates business logic from CLI and storage

Repository Pattern: TaskStorage abstracts persistence

Domain Model: Task, TaskStatus, TaskPriority encapsulate core entities

## PART 2: TASK PRIORITIZATION INSIGHTS

### Initial Understanding: Priority seemed numeric; now clarified as an enum (LOW, MEDIUM, HIGH, URGENT) that influences filtering and business rules.

KEY INSIGHTS:

Enums enforce valid values

Priority affects auto-abandon logic and task listing order

Misconception corrected: priority is not just cosmetic, it drives behavior

## PART 3: DATA FLOW – TASK COMPLETION

### DATA FLOW DIAGRAM:

User CLI Command
        ↓
   TaskManager
        ↓
   Task Object
        ↓
TaskStorage
        ↓
   tasks.json

State Changes:

Field	Before	After
status	IN_PROGRESS	DONE
completed_at	None	Current timestamp

Potential Failure Points:

Invalid task ID

Corrupted JSON file

Storage save failure

## PART 4: REFLECTION and INSIGHTS

### Architecture Overview:

CLI / Entry → TaskManager (Service Layer) → Task / Domain Entities → TaskStorage → tasks.json

Three Key Features:

Task Creation: validated, persisted, added to storage

Task Prioritization: enum-based, drives filtering and business rules

Task Completion: updates status and timestamp, persists changes

Interesting Design Pattern: Layered architecture separates CLI, business logic, and storage, improving maintainability.


# Understanding Complex Function Analysis

## Function Signature Analysis

Purpose: Identify the inputs, outputs, and high-level purpose of the function.

Example Visual:

Function: calculate_task_score(task)
-----------------------------------
Input: task (object with priority, due_date, status, tags, updated_at)
Output: integer score
High-level purpose: quantify task importance

### Insights:

Always start with what goes in and what comes out.

The function name often gives a clue about its primary responsibility.

## Split Function into Logical Sections

Purpose: Break the code into meaningful blocks to understand each step.

Diagram:

+---------------------+
| Initialization      |
| (setup variables)   |
+---------------------+
           |
           v
+---------------------+
| Core Computation    |
| (loops, calculations)|
+---------------------+
           |
           v
+---------------------+
| Conditionals        |
| (if/else, rules)    |
+---------------------+
           |
           v
+---------------------+
| Aggregation/Output  |
| (sorting, scoring)  |
+---------------------+
           |
           v
+---------------------+
| Return Statement    |
+---------------------+

### Insights:

Visualizing the flow from initialization → computation → return helps track data transformations.

Each block usually has a single responsibility, making it easier to debug or optimize.

## Annotate Each Section in Plain Language

Example:

1. Base score from priority
2. Adjust score for due date
3. Penalize completed/reviewed tasks
4. Boost for special tags
5. Bonus for recent updates
6. Return total score

### Insights:

Translating code into plain language steps helps identify what the algorithm is actually doing, not just what Python syntax says.

Makes it easier to explain to others or write documentation.

## Identify Optimizations or Tricks

### Example Observations from AI Explanation:

Using sets for tag membership → faster than lists.

Storing datetime.now() once → avoids repeated calls.

.days vs .total_seconds() → more accurate time comparisons.

Sorting with key=calculate_task_score → avoids building extra tuples.

### Insights:

Small tweaks can improve performance and accuracy.

Look for hidden assumptions, like task.tags always existing or task.updated_at not being None.

## Learning Points Summary

High-level purpose first: Know inputs, outputs, and intended effect.

Break into blocks: Identify initialization, core computation, branching, aggregation, and return.

Translate to plain language: Clarifies algorithm’s intent.

Spot optimizations: Recognize subtle tricks for speed, accuracy, or clarity.

Edge cases matter: Null values, overdue dates, or empty lists can break assumptions.

Reusability: This structured approach works for any complex function.

### Optional Flow Diagram of Analysis Process
[Start: Function]
      |
      v
[Identify Signature]
      |
      v
[Split into Sections]
      |
      v
[Annotate in Plain Language]
      |
      v
[Check for Tricks/Optimizations]
      |
      v
[Document Insights & Edge Cases]
      |
      v
[End: Full Understanding]
