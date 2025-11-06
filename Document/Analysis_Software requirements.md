# 🧾 Functional Requirements Document (FRD)

**Project Name:** Employee Clock In/Out System
**Developed in:** Microsoft Power Apps + Power Automate + Dataverse
**Purpose:** To digitize employee attendance tracking, automate weekly timesheets, and simplify manager reporting for the restaurant.

---

## 1. Employee Login

**Objective:**
Allow employees to securely access the app using the last 6 digits of their phone number.

**Description:**

* When the app opens, users see a **login screen** with a single input field for the **last 6 digits of their phone number**.
* On submission, the app validates the input against the `Employee` table.
* If a match is found, the system loads that employee’s session.
* If no match is found, an error message appears (“Invalid PIN number”).
* The system auto-logs out and returns to the login screen after each Clock In/Out action.

**Acceptance Criteria:**

* The app must validate the last 6 digits only (not full number).
* Only active employees can log in.
* Invalid or duplicate phone numbers must be rejected.
* After 30 seconds of inactivity, the app should auto-return to login screen.

**Optional Enhancements:**

* PIN code or QR code login in future.
* Role-based access (Manager vs Employee).

---

## 2. Clock In / Clock Out Functions

**Objective:**
Allow employees to record their working time accurately and easily.

**Description:**

* After login, the employee sees two buttons: **Clock In** and **Clock Out**.
* On tap:

  * The system records:

    * Employee ID
    * Date/time (`Now()`)
    * Action type (In or Out)
    * Position (if selected)
    * Device location (optional)
  * Displays a confirmation message: “Clock In recorded at 10:02 AM.”
* If the employee attempts to Clock In twice in a row without Clock Out, show a warning and block the action.
* Similarly, Clock Out cannot occur without a previous Clock In.

**Acceptance Criteria:**

* Each Clock In must have a corresponding Clock Out.
* The system must prevent duplicate or overlapping sessions.
* Time entries must be saved even if device temporarily goes offline (sync later).
* A “Missed Clock Out” flag should appear for any session not closed within 24 hours.

**Optional Enhancements:**

* Break tracking (Start/End Break).
* Capture device GPS for verification.

---

## 3. Time Classification

**Objective:**
Automatically categorize each recorded work session as **Weekday**, **Weekend**, or **Public Holiday**.

**Description:**

* The system checks the date of each clock-in session.
* Uses an internal reference table `HolidayList` that lists all Australian public holidays.
* Based on the date:

  * Monday–Friday → **Weekday**
  * Saturday/Sunday → **Weekend**
  * Matches `HolidayList` → **Public Holiday**
* Classification is done automatically when Clock Out is recorded (or via a Power Automate flow if done later).

**Acceptance Criteria:**

* All sessions must have exactly one classification type.
* If a date is both a weekend and a public holiday, “Public Holiday” takes priority.
* Holiday list must be configurable by the manager.

**Optional Enhancements:**

* Add shift-based rules (e.g., night shift spanning two days).
* Sync holiday list automatically from government API.

---

## 4. Weekly Summary

**Objective:**
Generate a weekly summary of total working hours per employee, broken down by category.

**Description:**

* Every Sunday night, a Power Automate flow runs to:

  * Aggregate all timesheet records from the previous Monday–Sunday.
  * Calculate total hours for each employee, grouped by:

    * Weekday hours
    * Weekend hours
    * Public holiday hours
  * Identify missing Clock Outs or incomplete sessions.
  * Email summary to the manager (and optionally each employee).
* Summary stored in `TimesheetSummary` table for reporting.

**Acceptance Criteria:**

* Weekly summary must include at least: Employee, Week Start, Week End, Total Hours (by type).
* Email must be delivered automatically every week.
* Rounding to nearest 5 or 15 minutes must be consistent (define business rule).
* Flow should skip inactive employees.

**Optional Enhancements:**

* Export summary as Excel file or sync with payroll (e.g., Xero).
* Include total wage estimate if hourly rates exist.

---

## 5. Admin Portal for Manager

**Objective:**
Provide a manager-only view to monitor attendance, correct errors, and manage employee records.

**Description:**

* Accessible via separate tab or screen in the same app.
* Features include:

  * View all Clock In/Out records.
  * Edit or delete incorrect entries (e.g., missing Clock Out).
  * Manage employees (add, deactivate, update phone numbers and positions).
  * Upload or modify Public Holiday list.
  * View Weekly Summary dashboard.

**Acceptance Criteria:**

* Only users with “Manager” flag = true in Employee table can access admin screen.
* All edits must log “Last Modified By” and “Last Modified On.”
* Deactivated employees must not appear in login screen.
* Changes in Admin portal should update Dataverse instantly.

**Optional Enhancements:**

* Add Power BI report dashboard for trends (total hours, attendance rate).
* Allow exporting to CSV.

---

# 🧱 Non-Functional Requirements (NFR)

## 1. **Ease of Use**

| ID      | Requirement           | Description                                                                                                          |
| ------- | --------------------- | -------------------------------------------------------------------------------------------------------------------- |
| NFR-1.1 | Simple user interface | The application shall provide a clean, intuitive interface suitable for use by employees with minimal training.      |
| NFR-1.2 | Minimal user input    | Employees shall only need to enter their phone number (last 6 digits) and tap a button to clock in/out.              |
| NFR-1.3 | Accessibility         | The app shall be optimized for tablet and desktop use, with large buttons and readable fonts for quick interactions. |
| NFR-1.4 | Fast navigation       | The system shall complete the clock-in or clock-out process within **5 seconds** from input to confirmation.         |
| NFR-1.5 | Error feedback        | Any input error (e.g., invalid phone number) shall be clearly displayed with user-friendly guidance.                 |

---

## 2. **Security (Limit Access to In-Shop Use)**

| ID      | Requirement             | Description                                                                                                                                        |
| ------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| NFR-2.1 | Restricted access       | The app shall only be accessible from **the shop’s designated device(s)** (e.g., via static IP, device ID, or kiosk login).                        |
| NFR-2.2 | Limited data visibility | Employees shall only see their own clock-in/out records; administrative data shall only be visible to managers.                                    |
| NFR-2.3 | Authentication          | Employee access shall be verified via the **last 6 digits of their phone number**, matched against a pre-defined employee list.                    |
| NFR-2.4 | Session timeout         | The system shall automatically return to the login screen after **30 seconds of inactivity** to prevent misuse.                                    |
| NFR-2.5 | Data protection         | All data (clock times, employee information) shall be stored securely in **Microsoft Dataverse** or **SharePoint** with role-based access control. |
| NFR-2.6 | Audit logging           | All user actions (login, clock in/out, data updates) shall be timestamped and auditable by the manager.                                            |

---

## 3. **Performance and Reliability**

| ID      | Requirement         | Description                                                                                                                 |
| ------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| NFR-3.1 | System uptime       | The application shall be available **99% of business hours** excluding planned maintenance.                                 |
| NFR-3.2 | Data integrity      | All clock-in/out transactions shall be recorded accurately and synchronized with the database within **10 seconds**.        |
| NFR-3.3 | Offline resilience  | If the app temporarily loses internet connectivity, it shall store records locally and sync automatically once reconnected. |
| NFR-3.4 | Scalability         | The system shall support at least **50 employees** and up to **500 transactions per day** without performance degradation.  |
| NFR-3.5 | Backup and recovery | Data backups shall occur daily, and recovery shall be possible within **1 hour** of a data loss event.                      |
| NFR-3.6 | Response time       | Page transitions and button actions shall respond within **2 seconds** on standard shop hardware.                           |



Perfect — these are exactly the kind of **Business Rules** that make a clock-in/out system truly useful and accurate for payroll and reporting.

Here’s a professional **Business Rules** section you can add right after your Functional and Non-Functional Requirements.

---

# ⚖️ Business Rules

## 1. **Missed Clock-Out Handling**

| ID     | Rule                   | Description                                                                                                                                                                  |
| ------ | ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| BR-1.1 | Automatic detection    | The system shall automatically detect if an employee has clocked in but not clocked out by the end of the day.                                                               |
| BR-1.2 | Default clock-out time | If a clock-out entry is missing, the system shall temporarily assign a **default clock-out time (e.g., store closing time)** for reporting purposes, marked as “Unverified.” |
| BR-1.3 | Manager correction     | The manager shall have the ability to manually update or confirm the correct clock-out time through the Admin Portal.                                                        |
| BR-1.4 | Notification           | Optionally, Power Automate can send a daily notification (email or Teams message) listing all employees with missing clock-out entries.                                      |

---

## 2. **Break Time Calculation** (Ignore this one currently)

| ID     | Rule                              | Description                                                                                                                                                                                      |
| ------ | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| BR-2.1 | Automatic break deduction         | For each shift longer than a defined duration (e.g., **6 hours**), the system shall automatically deduct a standard break time (e.g., **30 minutes**) unless a manual entry indicates otherwise. |
| BR-2.2 | Manual break recording (optional) | If the app later supports it, employees can clock out for “Break” and clock back in to resume work; total break time will be subtracted from total hours.                                        |
| BR-2.3 | Validation                        | Total break time per shift shall not exceed **90 minutes** without manager approval.                                                                                                             |
| BR-2.4 | Visibility                        | Break time deduction or manual breaks shall be visible in the weekly summary for transparency.                                                                                                   |

---

## 3. **Work Hour Rounding Rules**

| ID     | Rule               | Description                                                                                                        |
| ------ | ------------------ | ------------------------------------------------------------------------------------------------------------------ |
| BR-3.1 | Rounding precision | All clock-in and clock-out times shall be rounded to the **nearest 5 minutes** for reporting and payroll purposes. |
| BR-3.2 | Rounding logic     | Rounding shall follow standard midpoint rules:  * 0–2 minutes → round down * 3–5 minutes → round up |
| BR-3.3 | Storage | Raw timestamps (exact times) shall still be stored in the database for audit purposes, while the rounded values are used in reports. |
| BR-3.4 | Display | The weekly summary shall show **both actual and rounded** total hours, so the manager can verify calculations if needed. |

---

## 4. **Public Holiday and Weekend Classification**

| ID     | Rule                     | Description                                                                                                                |
| ------ | ------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| BR-4.1 | Classification logic     | The system shall automatically classify each work entry as **Weekday, Weekend, or Public Holiday** based on the work date. |
| BR-4.2 | Holiday table            | Public holidays shall be maintained in a **reference table** (editable by admin).                                          |
| BR-4.3 | Overtime flag (optional) | If an employee works on a public holiday or exceeds **8 hours per day**, the system can flag the entry for manager review. |

---

## 5. **Data Correction and Validation**

| ID     | Rule                 | Description                                                                                           |
| ------ | -------------------- | ----------------------------------------------------------------------------------------------------- |
| BR-5.1 | Edit control         | Only managers can modify time records; employees cannot edit past clock-in/out entries.               |
| BR-5.2 | Change tracking      | All edits must record the editor’s name, timestamp, and reason for audit purposes.                    |
| BR-5.3 | Duplicate prevention | The system shall prevent two active clock-ins for the same employee without an intervening clock-out. |


---

# 🔗 Integration Requirements

## 1. **Power Automate Integration**

| ID      | Integration                   | Description                                                                                                                                                 | Purpose                                             |
| ------- | ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| INT-1.1 | Daily sync of attendance data | Power Automate flow retrieves all clock-in/out records daily and compiles a summary per employee.                                                           | Generate daily reports and catch missed clock-outs. |
| INT-1.2 | Weekly report generation      | A scheduled flow runs every Sunday night to calculate total hours (weekday, weekend, public holiday) and sends a summary to the manager via email or Teams. | Automate weekly reporting and review.               |
| INT-1.3 | Missed clock-out alert        | Flow checks for employees still clocked in after business hours and sends an alert to the manager.                                                          | Prevent incorrect or unclosed shifts.               |
| INT-1.4 | Backup and archival           | Flow exports records weekly to Excel or SharePoint for long-term storage and backup.                                                                        | Data retention and audit history.                   |
| INT-1.5 | Admin notification automation | When a manager corrects time entries or adds new employees, Power Automate logs the changes and optionally notifies HR or payroll contact.                  | Maintain visibility and compliance.                 |

---

## 2. **Public Holiday List Integration**

| ID      | Integration                    | Description                                                                                                                                                                                                                                                        | Purpose                                   |
| ------- | ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------- |
| INT-2.1 | Static table                   | A “Public Holidays” table in Dataverse or SharePoint contains all holidays for the current year.                                                                                                                                                                   | Classify public holiday work.             |
| INT-2.2 | Dynamic data import (optional) | Power Automate retrieves official Australian public holiday data via government API (e.g., [data.gov.au public holiday feeds](https://data.gov.au/dataset/ds-dga-080f6b59-6481-4e8e-8b9c-9dd71d9f34d5/details?q=public%20holidays)) and updates the table monthly. | Keep holiday data accurate automatically. |
| INT-2.3 | Manager override               | Admin portal allows the manager to manually add or edit holiday dates for local or store-specific events.                                                                                                                                                          | Custom holiday flexibility.               |

---

## 3. **Payroll System Integration (Xero Export)**

| ID      | Integration                              | Description                                                                                                                                                 | Purpose                        |
| ------- | ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| INT-3.1 | Weekly hours export                      | Power Automate compiles total hours by employee and category (weekday/weekend/holiday) and exports to CSV/Excel format compatible with **Xero Timesheets**. | Simplify payroll input.        |
| INT-3.2 | Xero API integration (optional advanced) | For future automation, integrate directly with **Xero Payroll API** using Power Automate HTTP connector to push approved timesheets automatically.          | End-to-end payroll automation. |
| INT-3.3 | Data validation                          | Before export, flow validates that all records have both clock-in and clock-out times and are approved by the manager.                                      | Ensure payroll accuracy.       |
| INT-3.4 | Export log                               | All exports are logged (date, total entries, export file name) for auditing.                                                                                | Traceability of payroll data.  |

---

## 4. **Data Source and Storage**

| ID      | Integration                        | Description                                                                                                         | Purpose                                           |
| ------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| INT-4.1 | Microsoft Dataverse (preferred)    | Store employee, attendance, and configuration data in Dataverse with role-based access.                             | Reliability, security, and relational data model. |
| INT-4.2 | Microsoft SharePoint (alternative) | If Dataverse license is unavailable, use SharePoint lists with defined columns (Employee, ClockIn, ClockOut, etc.). | Low-cost implementation.                          |
| INT-4.3 | Backup to OneDrive                 | Weekly export of attendance data to OneDrive for Business via Power Automate.                                       | Prevent data loss.                                |

---

## 5. **Optional Future Integrations**

| ID      | Integration                      | Description                                                                                                                      | Purpose                |
| ------- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ---------------------- |
| INT-5.1 | Microsoft Teams notification bot | Notify employees or managers via Teams when their hours summary or corrections are available.                                    | Improve communication. |
| INT-5.2 | Power BI dashboard               | Connect to Dataverse data for visual analytics (hours by day, peak staff times, overtime tracking).                              | Management insights.   |
| INT-5.3 | Employee onboarding sync         | When new employees are added to Xero or a staff spreadsheet, a flow automatically adds them to the employee table in Power Apps. | Reduce double entry.   |

---

## ✅ Summary

| Integration Area      | Key Tool                  | Business Value                    |
| --------------------- | ------------------------- | --------------------------------- |
| Attendance automation | **Power Automate**        | Saves manual reporting time       |
| Holiday management    | **API / SharePoint**      | Accurate rate classification      |
| Payroll export        | **Xero + Power Automate** | Streamlined weekly payroll        |
| Data visualization    | **Power BI (optional)**   | Performance and staffing insights |

---


# 🔒 Data Privacy & Retention Policy

## 1. **Purpose**

This policy defines how employee data collected through the *Clock In/Out System* is stored, accessed, retained, and disposed of to ensure compliance with the **Australian Privacy Act 1988**, the **Australian Privacy Principles (APPs)**, and Microsoft’s data protection standards.

---

## 2. **Data Collected**

| Data Type            | Description                                                      | Source                             | Use                                   |
| -------------------- | ---------------------------------------------------------------- | ---------------------------------- | ------------------------------------- |
| Employee Information | Name, phone number (last 6 digits), position(s)                  | Admin input during employee setup  | Identification for clock in/out       |
| Attendance Records   | Clock-in time, clock-out time, total hours, shift classification | Employee actions within Power Apps | Work hour tracking and payroll export |
| Manager Actions      | Manual edits, approvals, and corrections                         | Admin portal                       | Verification and compliance audit     |
| System Logs          | User access timestamps, data sync logs                           | Power Automate / Dataverse         | Troubleshooting and audit tracking    |

---

## 3. **Data Storage**

| Element                    | Location                                                     | Security                                                                                      |
| -------------------------- | ------------------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| Employee & Attendance Data | **Microsoft Dataverse** (preferred) or **SharePoint Online** | Encrypted at rest and in transit (TLS 1.2), stored within Microsoft’s Australian data centres |
| Reports and Backups        | **OneDrive for Business** or **SharePoint document library** | Restricted to manager-level users via Microsoft 365 access controls                           |
| Audit Logs                 | **Dataverse / Power Automate history**                       | Retained for accountability and compliance verification                                       |

All data is stored within the organisation’s Microsoft 365 tenant and is **not shared with external parties** unless explicitly required for payroll integration (e.g., Xero).

---

## 4. **Access Control**

| Role              | Access Rights                                                             |
| ----------------- | ------------------------------------------------------------------------- |
| **Employee**      | Can view their own attendance history (read-only). Cannot modify records. |
| **Manager/Admin** | Can view and edit all records, approve corrections, and export reports.   |
| **System Owner**  | Full control of environment, data retention, and integration management.  |

Access is authenticated via **Microsoft Entra ID (Azure AD)**, and permissions are enforced through **Dataverse security roles** or **SharePoint list permissions**.

---

## 5. **Data Retention**

| Data Type             | Retention Period                                                                    | Action After Retention Period                                      |
| --------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Attendance records    | **7 years** (aligned with Australian Fair Work and tax record-keeping requirements) | Archived to OneDrive or deleted upon manager approval              |
| Employee data         | **12 months** after termination                                                     | Anonymised or deleted unless required for ongoing payroll disputes |
| Audit and change logs | **2 years**                                                                         | Automatically purged using scheduled Power Automate cleanup flow   |
| Reports and backups   | **7 years**                                                                         | Archived to secure storage or deleted after business review        |

---

## 6. **Data Privacy Principles**

1. **Consent and transparency:** Employees shall be informed about the purpose of data collection during onboarding.
2. **Data minimisation:** Only essential personal data (name and phone number) shall be collected.
3. **Use limitation:** Data shall only be used for workforce management, payroll processing, and legal compliance.
4. **Right to access:** Employees may request access to their stored attendance records at any time.
5. **Right to correction:** Errors in attendance data can be corrected by a manager upon employee request.
6. **Right to deletion:** Upon leaving employment, employees may request data deletion subject to legal retention requirements.

---

## 7. **Data Disposal**

When data reaches the end of its retention period:

* Power Automate will identify eligible records for deletion or archival.
* Data shall be **permanently deleted** using Microsoft 365’s secure deletion process (cannot be recovered).
* Backup archives shall be encrypted and stored in secure OneDrive folders accessible only to management.

---

## 8. **Third-Party Integrations (Xero)**

* Only summary data (employee name, total hours, rate classification) will be exported to Xero for payroll.
* No personal identifiers (full phone numbers, personal email addresses) will be transmitted.
* Exports will occur via secure Power Automate connection using OAuth 2.0 authentication.
* Xero’s own privacy and data retention policies apply once data enters their system.

---

## 9. **Incident Management**

In the event of unauthorised data access or breach:

1. The system owner will notify affected employees and management within **72 hours**.
2. Access credentials will be reset, and all affected sessions invalidated.
3. A root-cause analysis will be conducted, and controls updated to prevent recurrence.

---

## 10. **Policy Review**

This policy shall be reviewed **annually** or after any significant system change to ensure compliance with current Australian privacy regulations and Microsoft platform standards.

---

Would you like me to help you add a **“Compliance & Risk Management”** section next?
It would cover risk identification (e.g. data loss, misuse, system downtime) and mitigation strategies — useful for your *Testing* and *Deployment* stage documentation in Microsoft Planner.
