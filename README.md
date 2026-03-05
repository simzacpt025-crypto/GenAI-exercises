# GenAI-exercise

# DISCOVERY-NOTES


# Task Manager Codebase Exploration Summary

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
