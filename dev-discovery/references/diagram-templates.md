# Diagram Templates

Every dev discovery output MUST include rendered diagrams. Use **Mermaid** syntax (fenced with ` ```mermaid `) so diagrams render in GitHub, VS Code, and most markdown viewers.

## Entity Relationship Diagram

```mermaid
erDiagram
    branches ||--o{ employees : employ
    branches ||--o{ patients : serves
    employees ||--o{ attendance_logs : records
    employees ||--o{ leave_requests : files
    payroll_cutoffs ||--o{ payroll_entries : contains
    employees ||--o{ service_logs : performs
```

Include at module level and a master ERD in the final document.

## State Machine Diagram

Every entity with a `status` column that has more than 2 states MUST have a state machine:

```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Approved
    Pending --> Rejected
    Pending --> Cancelled
    Approved --> Cancelled
    Approved --> Completed
```

Include allowed transitions, side effects (notifications, audit logs, balance updates).

## Module Dependency Flowchart

```mermaid
flowchart LR
    Auth --> Employees
    Employees --> Attendance
    Employees --> Leave
    Employees --> Payroll
    Attendance --> Payroll
    Leave --> Payroll
    Loans --> Payroll
    ServiceLogs --> Commission --> Payroll
```

## Route / API Flow

```mermaid
flowchart TD
    A["POST /admin/employees"] --> B["CreateEmployeeRequest validates"]
    B --> C["EmployeeService::create"]
    C --> D["Create User (portal guard)"]
    C --> E["Assign Role"]
    C --> F["Create Leave Credits"]
    D --> G["Send Welcome Email"]
    F --> H["Create Audit Log"]
```

## Permission Hierarchy

```mermaid
flowchart TD
    Guard["guard: admin"] --> Module1["Module A"]
    Guard --> Module2["Module B"]
    Module1 --> P1["can-list-X"]
    Module1 --> P2["can-create-X"]
    Module1 --> P3["can-update-X"]
    Module2 --> P4["can-approve-Y"]
    Module2 --> P5["can-reject-Y"]

    Guard2["guard: portal"] --> Self["Self-Service"]
    Self --> S1["can-view-own-Z"]
    Self --> S2["can-file-own-Z"]
```

## Domain Islands (zoomed-in views)

For large systems, create focused ERDs for each domain island (auth, employment, payroll, assets) and link them from the master ERD.
