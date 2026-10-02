# Car Rental Request Automation in ServiceNow
### Complete 7-Phase Project Documentation

**Prepared by:** Pothanaboyina Kanaka Durga Rao
**Institution:** Chalapathi Institute of Engineering and Technology (CIET), Guntur
**Project Name:** Car Rental Request Automation using Service Catalog and Workflows
**Team ID:** _[fill in your assigned Team ID]_

---

# PHASE 1: Ideation Phase

## 1.1 Define the Problem Statements

| Problem Statement (PS) | I am (Customer) | I'm trying to | But | Because | Which makes me feel |
|---|---|---|---|---|---|
| PS-1 | An employee needing business travel | Book a rental car quickly for an upcoming trip | There's no standard process to request one | Requests go through scattered emails/calls to admin and transport staff | Frustrated and unsure if the car will be arranged on time |
| PS-2 | A manager | Review and approve employee car requests | I have no central place to see pending requests or audit past approvals | Approvals happen informally over email or verbally | Worried about accountability and policy compliance |
| PS-3 | A Transport Team coordinator | Know exactly which cars are needed, when, and for whom | I receive requests late or with incomplete details (dates, locations) | There's no structured data capture at request time | Overworked chasing missing information instead of arranging vehicles |

## 1.2 Empathy Map Canvas — "Employee Requesting a Car Rental"

| Section | Details |
|---|---|
| **Says** | "I need a car for my client visit next Tuesday." / "Why hasn't anyone confirmed my pickup time?" |
| **Thinks** | "Is there an official way to request this, or do I just email someone?" / "Will this get approved in time?" |
| **Does** | Emails their manager informally, calls the admin/transport desk, follows up repeatedly for confirmation |
| **Feels** | Anxious about travel readiness, frustrated by lack of visibility, unsure who to escalate to if there's no response |
| **Pain Points** | No standard request form; no tracking number; approval status is invisible; transport team gets incomplete trip details |
| **Gains (desired outcome)** | A single place to submit the request, instant confirmation of submission, visible approval status, confirmed vehicle details before travel |

## 1.3 Brainstorming & Idea Prioritization

**Step 1 — Team Gathering & Problem Selection:** Selected PS-1 (no standardized request process) as the primary problem to solve, since it's the root cause behind PS-2 and PS-3 as well.

**Step 2 — Brainstormed Ideas:**
- Build a shared spreadsheet for requests (rejected — no approval tracking, no automation)
- Build a custom mobile app from scratch (rejected — high cost, long timeline, duplicates existing ITSM investment)
- Use the organization's existing ServiceNow instance to create a Service Catalog item with an automated approval and fulfillment workflow (selected)
- Route everything through a shared email inbox (rejected — no audit trail, no SLA tracking)

**Step 3 — Idea Prioritization (Impact vs. Effort):**

| Idea | Impact | Effort | Priority |
|---|---|---|---|
| ServiceNow Catalog Item + Flow Designer automation | High | Medium | ✅ Selected |
| Custom mobile app | High | Very High | Rejected |
| Shared spreadsheet | Low | Low | Rejected |
| Email-based routing | Low | Low | Rejected |

**Chosen Solution:** Build a **Car Rental Request** Service Catalog item in ServiceNow with Flow Designer automation handling manager approval and Transport Team task assignment, all packaged and deployed via Update Set.

---

# PHASE 2: Requirement Analysis

## 2.1 Functional & Non-Functional Requirements

**Functional Requirements:**

| FR No. | Functional Requirement (Epic) | Sub Requirement (Story/Sub-Task) |
|---|---|---|
| FR-1 | Catalog Item Submission | Employee can select Car Rental Request from the Service Catalog; fills in Requested Date, Pickup Location, Drop Location, Duration, Car Type, Reason |
| FR-2 | Approval Routing | Request automatically routes to the designated approver (manager/Transport Team) for review |
| FR-3 | Task Automation | Upon approval, a Catalog Task (sc_task) is automatically created and assigned to Transport Team |
| FR-4 | Notifications | Requester receives an email when their request is approved |
| FR-5 | Exception Handling | If a car is unavailable, an Incident record is automatically created for follow-up |
| FR-6 | Configuration Tracking | All configuration (catalog item, variables, flow, business rule) is captured in an Update Set for deployment |

**Non-Functional Requirements:**

| NFR No. | Requirement | Description |
|---|---|---|
| NFR-1 | Usability | Catalog form should be simple enough for any employee to complete without training |
| NFR-2 | Performance | Approval and task creation should complete within seconds of approval action |
| NFR-3 | Auditability | Every approval decision and task creation must be logged and traceable |
| NFR-4 | Portability | Entire configuration must migrate cleanly between instances via Update Set |
| NFR-5 | Scalability | Design should allow adding new car types, locations, or approval tiers without redesigning the flow |

## 2.2 Technology Stack

| S.No | Component | Description | Technology |
|---|---|---|---|
| 1 | User Interface | Employee-facing request form | ServiceNow Service Catalog (Service Portal) |
| 2 | Application Logic | Approval routing, task creation, conditional branching | ServiceNow Flow Designer |
| 3 | Backend Logic | Exception handling (unavailable car → Incident) | ServiceNow Business Rules (Server-side JavaScript / GlideRecord) |
| 4 | Database | Request, approval, and task data storage | ServiceNow native tables (sc_req_item, sysapproval_approver, sc_task, incident) |
| 5 | Notification | Email alerts to requester | ServiceNow Notification/Email action within Flow Designer |
| 6 | Deployment | Configuration packaging and migration | ServiceNow Update Sets |

## 2.3 Data Flow Diagram (Description)

```
Employee → [Submits Car Rental Request Catalog Item]
            → Creates REQ + RITM record
                → Flow Designer Trigger fires
                    → Ask For Approval (Manager / Transport Team)
                        ├─ Approved → Create Catalog Task (sc_task → Transport Team)
                        │              → Send Email Notification (Requester)
                        └─ Rejected → RITM closed, no task created
                → Business Rule checks RITM state
                    → If "car unavailable" state reached → Create Incident record
```

## 2.4 User Stories

| User Type | Functional Requirement (Epic) | User Story Number | User Story / Task | Acceptance Criteria | Priority | Release |
|---|---|---|---|---|---|---|
| Employee (Requester) | Catalog Item Submission | USN-1 | As an employee, I can submit a car rental request with pickup/drop location, dates, car type, and reason | Request is saved as a RITM with all fields populated | High | Sprint-1 |
| Employee (Requester) | Notifications | USN-2 | As an employee, I receive an email once my request is approved | Email arrives with correct request details | High | Sprint-1 |
| Manager/Approver | Approval Routing | USN-3 | As a manager, I can approve or reject a car rental request from my Approvals list | Approval state updates the RITM accordingly | High | Sprint-1 |
| Transport Team | Task Automation | USN-4 | As a Transport Team member, I receive a task with full trip details once a request is approved | Catalog Task is visible in my assigned tasks with correct info | High | Sprint-1 |
| Transport Team | Exception Handling | USN-5 | As a Transport Team member, an Incident is automatically logged if a car can't be arranged | Incident appears with correct short description and linked request number | Medium | Sprint-2 |
| Admin | Configuration Tracking | USN-6 | As an admin, I can migrate the entire solution to another instance via Update Set | All catalog, flow, and rule artifacts transfer and function correctly | High | Sprint-2 |

---

# PHASE 3: Project Design Phase

## 3.1 Problem – Solution Fit

| Problem | Solution |
|---|---|
| No standardized way to request a car rental | Service Catalog item with structured variables |
| No visibility into approval status | Native ServiceNow Approval records, visible in Approvals list and RITM timeline |
| Manual, error-prone task handoff to Transport Team | Flow Designer auto-creates a Catalog Task with full trip details |
| No handling for unavailable cars | Business Rule auto-creates an Incident for proactive follow-up |
| Configuration hard to move between environments | All artifacts bundled in a single trackable Update Set |

## 3.2 Proposed Solution

A ServiceNow-native, no-code/low-code automation that:
1. Gives employees a self-service Car Rental Request catalog item.
2. Automatically routes the request for manager approval.
3. On approval, creates a fulfillment task for the Transport Team with all trip details pre-filled.
4. Notifies the requester by email once approved.
5. Raises an Incident automatically if the car can't be arranged.
6. Is fully packaged in an Update Set for repeatable deployment across Dev/Test/Prod.

## 3.3 Solution Architecture

```
┌─────────────────────────┐
│   Service Catalog Item   │   ← Employee submits request
│   "Car Rental Request"   │
└───────────┬──────────────┘
            │ creates
            ▼
┌─────────────────────────┐
│  Requested Item (RITM)   │
│     sc_req_item table    │
└───────────┬──────────────┘
            │ triggers
            ▼
┌─────────────────────────┐
│     Flow Designer        │
│  "Car Rental Req" Flow   │
│  - Ask For Approval      │
│  - If/Else Branch        │
│  - Create Catalog Task   │
│  - Send Email            │
└───────────┬──────────────┘
            │ on approval
            ▼
┌─────────────────────────┐        ┌──────────────────────────┐
│   Catalog Task (sc_task) │        │   Business Rule            │
│   → Transport Team       │        │   (checks RITM state)      │
└─────────────────────────┘        │   → Creates Incident if     │
                                    │     car unavailable         │
                                    └──────────────────────────┘
            │
            ▼
┌─────────────────────────┐
│       Update Set         │
│  Packages: Catalog Item, │
│  Variables, Flow, Rule,  │
│  Transport Team group    │
└─────────────────────────┘
```

---

# PHASE 4: Project Planning Phase

## 4.1 Product Backlog & Sprint Schedule

| Sprint | Functional Requirement (Epic) | User Story Number | User Story / Task | Story Points | Priority | Team Members |
|---|---|---|---|---|---|---|
| Sprint-1 | Catalog Item Submission | USN-1 | Create catalog item + 6 variables | 3 | High | Pothanaboyina Kanaka Durga Rao |
| Sprint-1 | Approval Routing | USN-3 | Build Flow Designer trigger + Ask For Approval action | 3 | High | Pothanaboyina Kanaka Durga Rao |
| Sprint-1 | Task Automation | USN-4 | Build If/Approved branch + Create Catalog Task action | 3 | High | Pothanaboyina Kanaka Durga Rao |
| Sprint-1 | Notifications | USN-2 | Add Send Email action to flow | 2 | High | Pothanaboyina Kanaka Durga Rao |
| Sprint-2 | Exception Handling | USN-5 | Create and test Business Rule for unavailable car → Incident | 3 | Medium | Pothanaboyina Kanaka Durga Rao |
| Sprint-2 | Configuration Tracking | USN-6 | Finalize, complete, and migrate Update Set | 2 | High | Pothanaboyina Kanaka Durga Rao |

## 4.2 Velocity

- Sprint duration: ~5 working days each
- Sprint-1 total story points: 3+3+3+2 = **11**
- Sprint-2 total story points: 3+2 = **5**
- Average Velocity (AV) = (11+5) / 2 sprints = **8 story points/sprint**

## 4.3 Burndown Chart (Description)

Plot **Story Points Remaining** (Y-axis) against **Days** (X-axis) for each sprint:
- Sprint-1 starts at 11 points, ideally trending to 0 by day 5 (catalog item + core flow complete).
- Sprint-2 starts at 5 points, trending to 0 by day 3–4 (business rule + deployment complete).

*(Use Excel/Google Sheets or a burndown chart tool to plot this visually for your submission — a simple line chart with "Ideal" vs. "Actual" lines is sufficient.)*

---

# PHASE 5: Project Development Phase

This phase is the hands-on build — fully covered in the step-by-step guides I built earlier in our conversation. Reference order:

1. **Update Set creation** (tracks everything from here)
2. **Catalog Item + Variables** (Requested Date, Pickup Location, Drop Location, Duration, Car Type, Reason)
3. **Transport Team group creation**
4. **Flow Designer build** (trigger → Ask For Approval → If/Else → Create Catalog Task → Send Email)
5. **Business Rule** (Car Unavailable Incident Creation script)
6. **End-to-end testing** (approved path + rejected path)

## 5.1 Performance / UAT Testing Checklist

| Test Case | Steps | Expected Result | Status |
|---|---|---|---|
| TC-1 | Submit catalog item with valid data | RITM created with all variable values populated | ☐ |
| TC-2 | Manager approves request | Approval state = Approved; flow continues | ☐ |
| TC-3 | Catalog Task created on approval | sc_task exists, assigned to Transport Team, correct trip details | ☐ |
| TC-4 | Email sent on approval | Requester receives email with correct subject/body | ☐ |
| TC-5 | Manager rejects request | RITM does not get a task created; appropriately closed | ☐ |
| TC-6 | Business Rule fires on unavailable state | Incident created with short description starting "Car unavailable..." | ☐ |
| TC-7 | Update Set migration | All artifacts migrate and function identically on second instance | ☐ |

Use this table directly as your **UAT Report**.

---

# PHASE 6: Project Documentation

Adapting the FSD format (originally for MERN) to this ServiceNow project:

## 1. Introduction
- **Project Title:** Car Rental Request Automation in ServiceNow Using Service Catalog and Workflows
- **Team Members:** Pothanaboyina Kanaka Durga Rao

## 2. Project Overview
- **Purpose:** Automate the intake, approval, and fulfillment of employee car rental requests.
- **Features:** Self-service catalog item, automated manager approval, automated Transport Team task creation, email notifications, automatic incident creation for unavailable cars, Update Set–based deployment.

## 3. Architecture
- **Catalog Layer:** Service Catalog item + Variable Set (6 variables)
- **Automation Layer:** Flow Designer flow (trigger, approval, branching, task creation, email)
- **Exception Layer:** Business Rule on sc_req_item for Incident creation
- **Data Layer:** Native ServiceNow tables — sc_req_item, sysapproval_approver, sc_task, incident

## 4. Setup Instructions
- **Prerequisites:** Active ServiceNow PDI (Personal Developer Instance), admin access
- **Installation:** Import the project's Update Set (XML) via Retrieved Update Sets → Preview → Commit

## 5. Folder Structure (for GitHub — see Part 2 below)
```
car-rental-request-servicenow/
├── docs/                 (all 7-phase documentation, this file, screenshots)
├── update-sets/          (exported Update Set XML files)
├── scripts/              (business rule script, flow logic notes)
└── README.md
```

## 6. Running the Application
- Navigate to Service Catalog → Car Rental Request → Order Now

## 7. "API"/Integration Documentation
- No external APIs; all native ServiceNow table interactions (GlideRecord)

## 8. Authentication
- Standard ServiceNow role-based access; approvals restricted to Manager/Transport Team roles

## 9. User Interface
- Screenshot: Catalog item form
- Screenshot: Approval list
- Screenshot: Flow Designer canvas
- Screenshot: Catalog Task assigned to Transport Team

## 10. Testing
- Covered in Phase 5's UAT table above

## 11. Screenshots / Demo
- See Phase 7 below

## 12. Known Issues
- Approver is currently hardcoded to a test user; should be made dynamic (Requested for → Manager) for production use
- State value `3` in the Business Rule should be confirmed against the instance's actual choice list before production use

## 13. Future Enhancements
- Dynamic manager-based approval routing
- Car Inventory table integration for real-time availability checking
- Performance Analytics dashboard for approval time, fulfillment time, and request volume

---

# PHASE 7: Project Demonstration

## 7.1 Demo Script

1. **Portal walkthrough:** Home → Service Catalog → Car Rental Request
2. **Live submission:** Fill form with sample trip details, click Order Now
3. **Show RITM:** Open the generated RITM, point out the Approval related list
4. **Approve as manager:** Impersonate/approve the request
5. **Show automation fire:** Open Flow Execution Details, show each step turning green
6. **Show Catalog Task:** Open the sc_task assigned to Transport Team
7. **Show email:** Display the notification sent to the requester
8. **Show exception path:** Manually trigger the "unavailable" state and show the auto-created Incident
9. **Show Update Set:** Open the completed Update Set, briefly show its contents, and show the commit on a second instance

## 7.2 What to Capture for Submission
- Screen recording (5–7 minutes) following the script above
- Annotated screenshots of: Flow Designer canvas, Approval record, Catalog Task, Email log, Incident record, Update Set (Complete state)

---

# PART 2: Pushing This Project to GitHub

## Step-by-step

1. **Create the repository**
   - Go to github.com → click **New repository**
   - Name it: `car-rental-request-servicenow`
   - Set visibility (Public if your course wants it reviewable, Private otherwise)
   - Do **not** initialize with a README yet (you'll add your own)
   - Click **Create repository**

2. **Organize your local files first**
   - On your computer, create a folder: `car-rental-request-servicenow/`
   - Inside it, create subfolders: `docs/`, `update-sets/`, `scripts/`
   - Put this documentation file in `docs/`
   - Export your ServiceNow Update Set as XML (from the Update Set record → right-click header → **Export to XML**) and save it into `update-sets/`
   - Save your Business Rule script as a `.js` file (e.g. `car-unavailable-incident.js`) into `scripts/`
   - Save your demo screenshots into `docs/screenshots/`

3. **Initialize Git locally**
   Open a terminal in your project folder and run:
   ```bash
   git init
   git add .
   git commit -m "Initial commit: Car Rental Request Automation documentation and Update Set"
   ```

4. **Connect to your GitHub repository**
   ```bash
   git remote add origin https://github.com/<your-username>/car-rental-request-servicenow.git
   git branch -M main
   git push -u origin main
   ```

5. **Add a README.md** (if you haven't already) summarizing the project — you can reuse the "Project Overview" section from Phase 6 above.

6. **For future updates** (e.g., adding more screenshots or an updated Update Set):
   ```bash
   git add .
   git commit -m "Add updated Update Set after business rule testing"
   git push
   ```

> Note: ServiceNow Update Set XML files and scripts are just text/data — there's nothing sensitive about pushing them to GitHub as long as they don't contain real employee emails, passwords, or production instance URLs. Double-check your exported XML for any live data before pushing, and scrub it if needed.

---

## What I still need from you to finalize this

If you want this 100% filled in (not just structured), please share:
- Your actual **Team ID**
- Any **screenshots** you've already taken (catalog item, flow, approvals, tasks, incident) so I can help you organize them into the Documentation and Demonstration sections
- Confirmation of your **actual State field value** for "car unavailable" on your instance, so Phase 5/6's known-issues note can be finalized
