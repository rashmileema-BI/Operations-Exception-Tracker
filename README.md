# Operations Exception & Approval Tracker Platform

An enterprise operations management and workflow automation platform built to log, triage, approve, and resolve operational exceptions. Powered by Python, SQL Server, Power Apps, and Power Automate, this system enforces audit-ready governance across multi-tiered risk incidents while providing operational visibility.

---

## 🎯 Project Overview

Business operations across financial processing, compliance, and systems infrastructure encounter daily disruptions and exceptions. Unmonitored incidents lead to missed SLAs, regulatory penalties, and unmanaged backlogs. This platform establishes an automated incident-to-resolution lifecycle to manage:

* **Automated Risk Triage:** Routing High-severity incidents to designated approvers while auto-logging standard exceptions.
* **SLA & Bottleneck Tracking:** Calculating resolution timeframes to surface overdue and unaddressed operational debt.
* **Audit Compliance & Accountability:** Enforcing mandatory sign-offs, manager decision rationale, and structured resolution notes.
* **Self-Service Operational Workflow:** Front-end submission, search, status inspection, and inline resolution handling.

---

## 🛠️ Tech Stack

| Tool / Technology | Purpose |
| :--- | :--- |
| **Python (`pandas`, `Faker`, `SQLAlchemy`)** | Synthetic data generation, realistic operational event simulation, and ETL ingestion pipeline |
| **Microsoft SQL Server / SSMS** | Relational data store hosting operational records, schema definitions, and audit history |
| **Microsoft Power Apps** | Low-code operational UI for exception creation, request inspection, filtering, and detail management |
| **Microsoft Power Automate** | Automated approval orchestration, asynchronous manager approval timeouts, and status notification emails |

---

## 📐 Data Pipeline & Architecture

```text
[ Python / Faker Generator ]
            │
            ▼ (ODBC / SQLAlchemy)
[ SQL Server: OpsExceptionsDB.dbo.Exceptions ]
            │
            ├───────────────┐
            ▼               ▼
[ Power Apps UI ]   [ Power Automate Flow ]
 (Intake & Audit)     (Approval Routing & Notifications)
```
---

### Database Schema (`OpsExceptionsDB.dbo.Exceptions`)

* **`ExceptionID`** (`INT`): Unique primary sequence identifier.
* **`ExceptionRef`** (`VARCHAR`): Standardized reference tag formatted by creation year (e.g., `EXC-2026-00003`).
* **`DateCreated`** (`DATETIME`): Incident timestamp.
* **`Category`** (`VARCHAR`): Domain classification (`Compliance Issue`, `Billing Discrepancy`, `SLA Breach`, `Data Error`, `System Downtime`, `Process Deviation`, `Vendor Delay`).
* **`Severity`** (`VARCHAR`): Priority tier (`Low`, `Medium`, `High`).
* **`Owner`** (`VARCHAR`): Assignee responsible for logging or managing the exception.
* **`Description`** (`VARCHAR`): Contextual summary of the operational breakdown.
* **`Status`** (`VARCHAR`): Lifecycle state (`Pending`, `Approved`, `Rejected`, `Closed/Resolved`).
* **`Approver`** (`VARCHAR`, Nullable): Escalation authority assigned to high-severity records.
* **`Approver_Comments`** (`VARCHAR`, Nullable): Audit feedback and sign-off rationale.
* **`Resolution`** (`VARCHAR`, Nullable): Root-cause action note executed upon case closure.
* **`DateResolved`** (`DATETIME`, Nullable): Resolution completion timestamp.
* **`ResolutionTimeHours`** (`FLOAT`, Nullable): Calculated time difference (`DateResolved - DateCreated`) in hours.

---

## ⚙️ Workflow Logic & Automation Flow

The solution leverages an integrated Power Automate flow (**Operations Exception Tracker Approval**) that coordinates with Power Apps:

```text
                  [ Exception Created / Submitted ]
                                  │
                       [ Severity == "High"? ]
                                  │
                 ┌────────────────┴────────────────┐
               Yes                                 No
                 │                                 │
        [ Create Approval ]                [ Auto-log Status ]
        [ Parallel Timeout/Wait ]                  │
                 │                         [ Send Logged Notice ]
        ┌────────┴────────┐
    Approved           Rejected
        │                  │
[ Update to Approved ]  [ Update to Rejected ]
[ Email Assignee ]      [ Request Justification ]
```

* **Trigger & Conditional Branching:** When an exception is logged, the system evaluates severity. Non-high records are logged directly as `Pending` or auto-routed for standard tracking.
* **Approval Handling:** High-severity items initiate an approval process with an automated escalation and delay-monitoring fallback.
* **Audit Trail Synchronization:** Approver decisions and review comments are pushed directly back into SQL Server, transitioning records to `Approved`, `Rejected`, or `Closed/Resolved` with corresponding notification dispatches.

---

## 📱 Power Apps Interface

The canvas application provides end-to-end exception lifecycle tracking:

### Executive Operations Hub
* **High-Level Metric Tiles:** Total Requests, Open Requests, High Risk Requests, and Overdue Requests.
* **Filterable Gallery:** Real-time gallery filtered by Category, Owner, and Status tags.

### Exception Inspection & Detail View
* **Segmented Information Panes:** Dedicated views showing Basic Information, Description, and Resolution.
* **Audit Visibility:** Clear tracking of manager review notes, assigned approvers, and calculated resolution turnaround hours.

### Standardized Incident Submission
* **Dynamic Form:** Fields capturing Category, Severity, Assignee, and Description with built-in validation rules.

---

## 💻 Sample Data Pipeline Execution

```python
# Synthetic generation logic for operational records
data = generate_rows(ROW_COUNT=2000)

# Calculate operational resolution time
df["ResolutionTimeHours"] = (
    (df["DateResolved"] - df["DateCreated"]).dt.total_seconds() / 3600
).round(1)

# Direct staging to SQL Server
push_to_sql(data)
# Output: Loaded 2000 rows into OpsExceptionsDB.dbo.Exceptions

---

## 📁 Repository Structure

```text
├── .env.example                          # Database environment variables template
├── requirements.txt                      # Python dependencies (pandas, faker, sqlalchemy, pyodbc)
├── notebooks/
│   └── exception_data_generator.ipynb   # Notebook for generating mock enterprise incident logs
├── sql/
│   └── schema_setup.sql                  # Table schemas, constraints, and operational views
├── power_platform/
│   └── flows/                            # Exported Power Automate workflow definitions
└── README.md                             # Project documentation
```
