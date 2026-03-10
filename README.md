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


# AI Code Documentation

## Overview

This project demonstrates how AI-assisted documentation can be used to understand and explain existing code. The goal of the exercise was to analyze a piece of Python code, generate documentation using AI prompts, identify improvements, and produce a refined version of the documentation.

The selected code implements a two-way synchronization algorithm used to merge task lists from a local system and a remote system.

This type of algorithm is commonly used in:

Offline-first applications

Distributed systems

Multi-device productivity apps

Cloud synchronization services

The exercise demonstrates how AI can assist developers in:

Understanding unfamiliar code

Generating documentation quickly

Identifying potential improvements

Creating clearer explanations of complex logic

Repository Structure
AI-Documentation-Exercise
│
├── original_code.py
├── prompt1_documentation.md
├── prompt2_insights.md
└── README.md
File	Description
original_code.py	The original function selected for documentation
prompt1_documentation.md	Documentation generated using Prompt 1
prompt2_insights.md	Insights and improvements identified using Prompt 2
README.md	Final combined documentation and summary

### 1. Original Code

The following function merges task lists from local and remote sources and determines synchronization actions.

def merge_task_lists(local_tasks, remote_tasks):
    merged_tasks = {}
    to_create_remote = {}
    to_update_remote = {}
    to_create_local = {}
    to_update_local = {}

    all_task_ids = set(local_tasks.keys()) | set(remote_tasks.keys())

    for task_id in all_task_ids:
        local_task = local_tasks.get(task_id)
        remote_task = remote_tasks.get(task_id)

        if local_task and not remote_task:
            merged_tasks[task_id] = local_task
            to_create_remote[task_id] = local_task

        elif not local_task and remote_task:
            merged_tasks[task_id] = remote_task
            to_create_local[task_id] = remote_task

        else:
            merged_task, update_local, update_remote = resolve_task_conflict(
                local_task, remote_task
            )

            merged_tasks[task_id] = merged_task

            if update_local:
                to_update_local[task_id] = merged_task

            if update_remote:
                to_update_remote[task_id] = merged_task

    return (
        merged_tasks,
        to_create_remote,
        to_update_remote,
        to_create_local,
        to_update_local
    )
## 2. Documentation Generated Using Prompt 1
### Function Purpose
The `merge_task_lists` function merges tasks from two data sources:

Local task storage

Remote task storage

The function creates a single merged view of tasks while determining which updates must occur to keep both sources synchronized.

### Inputs
Parameter	Type	Description
`local_tasks`	Dictionary	Tasks stored locally
`remote_tasks`	Dictionary	Tasks stored on the server

Each dictionary uses the format:

task_id → Task object

### Outputs

The function returns five dictionaries.

Output	Description
`merged_tasks`	Final combined task list
`to_create_remote`	Tasks that must be created on the server
`to_update_remote`	Tasks that must be updated on the server
`to_create_local`	Tasks that must be created locally
`to_update_local`	Tasks that must be updated locally

Algorithm Workflow

<img width="1241" height="1700" alt="mermaid-diagram (2)" src="https://github.com/user-attachments/assets/a1474064-3c14-4621-b875-7c972b8eebe1" />


## 3. Insights and Improvements (Prompt 2)

After analyzing the generated documentation, several insights and improvements were identified.

### Improvement 1 — Explicit `None` Checks

Original code:

if local_task and not remote_task

Improved version:

if local_task is not None and remote_task is None

This avoids errors if objects evaluate to `False`.

### Improvement 2 — Safer Tag Handling

If a task contains `tags = None`, converting it to a set will fail.

Improved implementation:

set(local_task.tags or [])

This ensures tags are always processed safely.

### Improvement 3 — Timestamp Synchronization Issues

The algorithm assumes:

The latest timestamp wins

Potential issue:

If system clocks differ, incorrect merges may occur.

Possible improvements:

Server-authoritative timestamps

Vector clocks

Conflict history tracking

### Improvement 4 — Field-Level Conflict Resolution

Current approach:

Newest version replaces entire task

Better approach:

Merge individual fields.

Example:

Local Change	Remote Change
Title updated	Priority updated

Field-level merging preserves both updates.

## 4. Final Combined Documentation
### System Overview

The `merge_task_lists` function performs two-way task synchronization between local and remote data sources.

It generates a merged task list and a set of actions required to synchronize both systems.

### High-Level Architecture

<img width="2084" height="519" alt="mermaid-diagram (1)" src="https://github.com/user-attachments/assets/f146a420-5af1-4fae-8d61-bd39d1304589" />

### Synchronization Rules
Scenario	Action
Task only exists locally	Create remotely
Task only exists remotely	Create locally
Task exists in both	Resolve conflict

### Conflict Resolution Strategy
The `resolve_task_conflict` function uses the following rules:

Latest Update Wins

The task with the newest `updated_at` timestamp overwrites older data.

### Completion Status Overrides

If one task is marked as completed, the merged result will also be completed.

### Tags Are Merged

Tags from both tasks are combined.

Example:

Local tags: {work, urgent}
Remote tags: {urgent, client}

Merged tags: {work, urgent, client}
Complexity Analysis
Type	Complexity
Time Complexity	O(n)
Space Complexity	O(n)

Where n IS THE TOTAL NUMBER OF UNIQUE TASKS.

# Documentatiom API

### 1. Original API Endpoint Code

This is the endpoint that was documented.

@app.route('/api/users/register', methods=['POST'])
def register_user():
    """Register a new user"""
    data = request.get_json()

    # Validate required fields
    required_fields = ['username', 'email', 'password']
    for field in required_fields:
        if field not in data:
            return jsonify({
                'error': 'Missing required field',
                'message': f'{field} is required'
            }), 400

    # Check if username or email already exists
    if User.query.filter_by(username=data['username']).first():
        return jsonify({
            'error': 'Username taken',
            'message': 'Username is already in use'
        }), 409

    if User.query.filter_by(email=data['email']).first():
        return jsonify({
            'error': 'Email exists',
            'message': 'An account with this email already exists'
        }), 409

    # Validate email format
    if not re.match(r"^[^@]+@[^@]+\.[^@]+$", data['email']):
        return jsonify({
            'error': 'Invalid email',
            'message': 'Please provide a valid email address'
        }), 400

    # Validate password strength
    if len(data['password']) < 8:
        return jsonify({
            'error': 'Weak password',
            'message': 'Password must be at least 8 characters long'
        }), 400

    try:
        password_hash = generate_password_hash(data['password'])

        new_user = User(
            username=data['username'],
            email=data['email'].lower(),
            password_hash=password_hash,
            created_at=datetime.utcnow(),
            role='user'
        )

        db.session.add(new_user)
        db.session.commit()

        confirmation_token = generate_confirmation_token(new_user.id)

        try:
            send_confirmation_email(new_user.email, confirmation_token)
        except Exception as e:
            app.logger.error(f"Failed to send confirmation email: {str(e)}")

        user_data = {
            'id': new_user.id,
            'username': new_user.username,
            'email': new_user.email,
            'created_at': new_user.created_at.isoformat(),
            'role': new_user.role
        }

        return jsonify({
            'message': 'User registered successfully',
            'user': user_data
        }), 201

    except Exception as e:
        db.session.rollback()
        app.logger.error(f"Error registering user: {str(e)}")
        return jsonify({
            'error': 'Server error',
            'message': 'Failed to register user'
        }), 500
### 2. Comprehensive Endpoint Documentation (Prompt 1)
Endpoint

POST /api/users/register

Description

Registers a new user account. The endpoint validates user input, checks for duplicate usernames or emails, hashes the password securely, and stores the new user in the database. After successful registration, a confirmation email is sent to verify the account.

Request Body
Field	Type	Required	Description
username	string	Yes	Unique username for the account
email	string	Yes	Valid email address
password	string	Yes	Password with minimum length of 8 characters
Example Request
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecurePass123"
}
Response
Success Response

Status Code: 201 Created

{
  "message": "User registered successfully",
  "user": {
    "id": 15,
    "username": "john_doe",
    "email": "john@example.com",
    "created_at": "2026-03-10T12:30:00",
    "role": "user"
  }
}
Error Responses
Missing Required Field

Status: 400 Bad Request

{
  "error": "Missing required field",
  "message": "username is required"
}
Invalid Email

Status: 400 Bad Request

{
  "error": "Invalid email",
  "message": "Please provide a valid email address"
}
Weak Password

Status: 400 Bad Request

{
  "error": "Weak password",
  "message": "Password must be at least 8 characters long"
}
Username Already Exists

Status: 409 Conflict

{
  "error": "Username taken",
  "message": "Username is already in use"
}
Email Already Exists

Status: 409 Conflict

{
  "error": "Email exists",
  "message": "An account with this email already exists"
}
Server Error

Status: 500 Internal Server Error

{
  "error": "Server error",
  "message": "Failed to register user"
}
### 3. Converted Documentation Format (Prompt 2)
OpenAPI / Swagger Format
paths:
  /api/users/register:
    post:
      summary: Register a new user
      description: Creates a new user account and sends a confirmation email.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - username
                - email
                - password
              properties:
                username:
                  type: string
                  example: john_doe
                email:
                  type: string
                  format: email
                  example: john@example.com
                password:
                  type: string
                  example: SecurePass123
      responses:
        "201":
          description: User registered successfully
        "400":
          description: Invalid request
        "409":
          description: Duplicate username or email
        "500":
          description: Server error

High-Level System Architecture Diagram

This shows how the frontend, API, database, and email service interact.

                    +----------------------+
                    |      Frontend        |
                    | (Web / Mobile App)   |
                    +----------+-----------+
                               |
                               | HTTPS Request
                               | POST /api/users/register
                               ▼
                    +----------------------+
                    |      Flask API       |
                    |   (REST Backend)     |
                    +----------+-----------+
                               |
             +-----------------+------------------+
             |                                    |
             ▼                                    ▼
   +--------------------+              +----------------------+
   |  Authentication &  |              |  Email Service       |
   |  Validation Layer  |              |  (SMTP / Mail API)   |
   +----------+---------+              +----------+-----------+
              |                                   |
              | Validate Input                    |
              ▼                                   |
   +--------------------+                         |
   |  Business Logic    |-------------------------+
   |  User Registration |
   +----------+---------+
              |
              | Store User Data
              ▼
      +--------------------+
      |     Database       |
      |  (Users Table)     |
      +--------------------+

Explanation for the report:

*The frontend client sends a registration request.

*The Flask API processes the request.

*The validation layer checks required fields and formats.

*The business logic layer handles password hashing and user creation.

*The database stores user information.

*The email service sends the confirmation email.

### 4. Developer Usage Guide (Prompt 3)
Registering a User

To register a new user, send a POST request to:

/api/users/register
Example Using cURL
curl -X POST http://localhost:5000/api/users/register \
-H "Content-Type: application/json" \
-d '{
"username":"john_doe",
"email":"john@example.com",
"password":"SecurePass123"
}'
Example Using JavaScript (Fetch)
fetch("/api/users/register", {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    username: "john_doe",
    email: "john@example.com",
    password: "SecurePass123"
  })
})
.then(res => res.json())
.then(data => console.log(data));
Expected Workflow

Client sends registration request.

API validates input fields.

API checks for duplicate username/email.

Password is securely hashed.

User record is stored in the database.

Confirmation email is sent.

API returns user details.

 # API Sequence Diagram

This is commonly used in technical documentation to show the order of interactions.

Client            API Server            Database           Email Service
  |                    |                     |                     |
  | POST /register     |                     |                     |
  |------------------->|                     |                     |
  |                    | Validate Input      |                     |
  |                    |-------------------->|                     |
  |                    | Check Username      |                     |
  |                    |<--------------------|                     |
  |                    | Check Email         |                     |
  |                    |-------------------->|                     |
  |                    |<--------------------|                     |
  |                    | Hash Password       |                     |
  |                    | Store User          |                     |
  |                    |-------------------->|                     |
  |                    |<--------------------|                     |
  |                    | Generate Token      |                     |
  |                    | Send Email          |-------------------->|
  |                    |                     |                     |
  | 201 Created        |                     |                     |
  |<-------------------|                     |                     |

Explanation:

This diagram demonstrates the chronological flow of events when a user registers.

### API Endpoint Component Diagram

This diagram shows internal backend components.

                 +----------------------+
                 |   Flask Application  |
                 +----------+-----------+
                            |
        +-------------------+-------------------+
        |                                       |
        ▼                                       ▼
+---------------+                     +-------------------+
|  Route Layer  |                     |  Utility Services |
| /api/register |                     |                   |
+-------+-------+                     | - Email Sender    |
        |                             | - Token Generator |
        ▼                             +---------+---------+
+---------------+                               |
| Validation    |                               |
| - Fields      |                               |
| - Email       |                               |
| - Password    |                               |
+-------+-------+                               |
        |                                       |
        ▼                                       ▼
+---------------+                     +-------------------+
| Business Logic|                     | Security Layer    |
| Create User   |                     | Password Hashing  |
+-------+-------+                     +---------+---------+
        |                                       |
        ▼                                       |
+----------------------+
|      Database        |
|       Users          |



## 5. Reflection Questions
Which parts of the API were most challenging to document?

The most challenging parts were identifying all possible error responses and understanding the validation logic implemented in the endpoint. Documenting edge cases such as duplicate usernames, invalid email formats, and weak passwords required careful examination of the code.

How did you adjust your prompts to get better results?

Initially, the prompts produced very high-level documentation. I improved the results by making the prompts more specific, asking the AI to include:

Request and response examples

HTTP status codes

Error scenarios

Field descriptions

This produced more structured and complete documentation.

Which documentation format was most effective?

The OpenAPI/Swagger format was the most effective because it can be integrated into tools like Swagger UI and Postman. It allows developers to automatically generate interactive documentation and test API endpoints directly.

How would you incorporate this approach into your workflow?

I would incorporate AI-assisted documentation into the development workflow by:

Generating API documentation immediately after writing endpoints.

Converting documentation into OpenAPI format for automated API docs.

Including documentation generation as part of code reviews.

Keeping API documentation synchronized with backend changes.

This approach would improve developer productivity and ensure APIs remain well documented
