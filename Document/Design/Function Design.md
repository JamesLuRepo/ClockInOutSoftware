Excellent — you’re now at the **core design stage** 👏.
Below is a clear and scalable **Data Model Design** for your *Employee Clock In/Out System* built in **Power Apps + Dataverse (or SharePoint)**.

This model supports your current goals — multi-role employees, time classification, and integration with Xero — and can scale for future reporting and automation.

---

# 🧩 Data Model Design

## **Overview**

| Table           | Purpose                                                | Key Relationships                           |
| --------------- | ------------------------------------------------------ | ------------------------------------------- |
| **Employee**    | Stores employee info and available positions.          | 1 → Many with ClockRecord, Timesheet        |
| **ClockRecord** | Each individual clock in/out event.                    | Many → 1 Employee, Many → 1 Timesheet       |
| **Timesheet**   | Weekly summary of each employee’s total working hours. | 1 → Many ClockRecord                        |
| **HolidayList** | Contains official and custom public holidays.          | Used by ClockRecord for time classification |

---

## 1️⃣ **EmployeeInfo Table**

**Purpose:** Maintain employee profiles and available positions.

| Field Name                | Type                | Example                                     | Description                                   |
| ------------------------- | ------------------- | ------------------------------------------- | --------------------------------------------- |
| **EmployeeID**            | GUID (Primary Key)  | EMP-001                                     | Unique system identifier.                     |
| **FullName**              | Text                | John Smith                                  | Employee’s full name.                         |
| **PhoneLast6**            | Text                | 123456                                      | Used as login credential in app.              |
| **Positions**             | Multi-select choice | Cook, Cashier                               | Employee’s available roles in the restaurant. |
| **Status**                | Choice              | Active / Inactive                           | Used to hide ex-employees.                    |
| **StartDate**             | Date                | 2023-05-01                                  | Employment start date.                        |
| **EndDate**               | Date (nullable)     | —                                           | For terminated staff.                         |
| **Email (optional)**      | Text                | [john@wokitup.com](mailto:john@wokitup.com) | Optional field for notifications.             |
| **HourlyRateWeekDay (optional)** | Number       | 25.5                                        | For future payroll integration.               |
| **HourlyRateWeekend (optional)** | Number       | 25.5                                        | For future payroll integration.               |
| **HourlyRatePublicHoliday (optional)** | Number | 25.5                                        | For future payroll integration.               |
| **EmployeeClockStatus**   | Choice              | Clock In/Clock Out                            | Flag employee's clocking status.              |

🧠 **Notes:**

* If one employee can have multiple positions, store positions as a *multi-select choice column* (or link to a `Position` lookup table if you need structured roles later).
* Index `PhoneLast6` for quick authentication lookup.

---

## 2️⃣ **ClockRecord Table**

**Purpose:** Track all clock-in and clock-out events.

| Field Name          | Type               | Example                                | Description                                     |
| ------------------- | ------------------ | -------------------------------------- | ----------------------------------------------- |
| **ClockRecordID**   | GUID (Primary Key) | CLK-000123                             | Unique record ID.                               |
| **Name**            | Text  | EmployeeName & ClockInTime(Now())      | create a Name for this record including the employee name and the clockin time|
| **EmployeeName**    | Lookup(Employee)   | John Smith                             | Link to employee who clocked in/out.            |
| **ClockInTime**     | DateTime           | 2025-11-02 10:03                       | Clock-in timestamp.                             |
| **ClockOutTime**    | DateTime           | 2025-11-02 18:27                       | Clock-out timestamp.                            |
| **ShiftType**       | Choice             | Weekday / Weekend / Public Holiday     | Determined automatically.                       |
| ~~**BreakMinutes**~~    | Number         | 30                                 | Auto or manual deduction.                 |
| ~~**RoundedHours**~~| Decimal            | 8.0                                    | After applying rounding rules.                  |
| ~~**RawHours**~~    | Decimal            | 8.4                                    | Actual calculated hours.                        |
| **DurationInHours** | Decimal            | 8.4                                    | Actual calculated hours.                        |
| **ClockRecordStatus**          | Choice         | Active / Completed | Label the record.  Active: When the employee clock in but they haven't clock out. Completed: When the employee clock out and this record is finished. We can start a new record.                    |
| **TimesheetID**     | Lookup(Timesheet)  | —                                      | Link to weekly summary.                         |
| **Notes**           | Text               | Missed clock-out; corrected by manager | Optional remarks.                               |
| **CreatedBySystem** | Boolean            | true                                   | For audit tracking (manual vs automated entry). |

🧠 **Logic:**

* Each time an employee clocks in/out, Power Automate or Power Apps formula updates this table.
* If `ClockOutTime` is missing at day-end, flag record as “Pending” for manager correction.

---

## 3️⃣ **Timesheet Table**

**Purpose:** Summarize weekly work hours per employee.

| Field Name                  | Type               | Example                       | Description                             |
| --------------------------- | ------------------ | ----------------------------- | --------------------------------------- |
| **TimesheetID**             | GUID (Primary Key) | TS-2025W44-EMP01              | Unique week+employee identifier.        |
| **EmployeeID**              | Lookup(Employee)   | John Smith                    | The employee this timesheet belongs to. |
| **WeekStartDate**           | Date               | 2025-10-27                    | Start of week (Monday).                 |
| **WeekEndDate**             | Date               | 2025-11-02                    | End of week (Sunday).                   |
| **TotalWeekdayHours**       | Decimal            | 32.5                          | Total weekday hours worked.             |
| **TotalWeekendHours**       | Decimal            | 10.0                          | Total weekend hours worked.             |
| **TotalPublicHolidayHours** | Decimal            | 0                             | Total public holiday hours.             |
| **TotalHours**              | Decimal            | 42.5                          | Total of all categories.                |
| **ApprovalStatus**          | Choice             | Pending / Approved / Exported | Workflow tracking.                      |
| **ApprovedBy**              | Lookup(Employee)   | Manager                       | Approval record.                        |
| ~~**ExportedToXero**~~          | Boolean            | false                         | Mark after successful export.           |
| ~~**ExportDate**~~              | Date               | —                             | Timestamp for export.                   |

🧠 **Logic:**

* Generated automatically at week’s end via Power Automate (group by employee + week).
* Supports audit trail and Xero export integration.

---

## 4️⃣ **HolidayList Table**

**Purpose:** Store official and shop-specific public holidays.

| Field Name          | Type               | Example         | Description                              |
| ------------------- | ------------------ | --------------- | ---------------------------------------- |
| **HolidayID**       | GUID (Primary Key) | HOL-2025-01     | Unique holiday record.                   |
| **Date**            | Date               | 2025-01-01      | Public holiday date.                     |
| **Name**            | Text               | New Year’s Day  | Holiday name.                            |
| **State/Territory** | Choice             | ACT / NSW / VIC | To handle regional differences.          |
| ~~**IsActive**~~        | Boolean            | true            | For easy filtering.                      |
| ~~**Source**~~          | Text               | Manual / API    | Origin of record (manager or auto-sync). |

🧠 **Logic:**

* Power Automate can update this table monthly from **data.gov.au** API.
* Used by `ClockRecord` creation logic to determine if a day is a public holiday.

---

## 📊 **Entity Relationship Diagram (ERD)**

```
Employee (1) ───< ClockRecord >───(1) Timesheet
     │                             │
     │                             └── Summarizes hours per week
     │
     └── Uses HolidayList (for shift classification)
```

---

## ⚙️ **Implementation Notes**

* **Primary Storage:** Microsoft Dataverse
  (supports relationships, choice fields, auditing, and Power Automate integration natively).
* **Alternative:** SharePoint Lists (if cost-sensitive)

  * Create separate lists for Employee, ClockRecord, Timesheet, and HolidayList.
  * Use Power Automate to link and calculate summaries.
* **Data Access:**

  * Employees: filtered view (`EmployeeID = User()` or phone login).
  * Manager: full access, edit + export permissions.

---

## 💡 1. Can I directly use `PhoneLast6` as primary key in the **Employee** table?

**Not recommended**.

### Why:

* The **last 6 digits of a phone number** are not guaranteed to be unique (especially if you hire new staff or people change phones).
* It also exposes part of their personal data as an identifier, which isn’t ideal for privacy.

### ✅ Recommended:

* Use an auto-generated **EmployeeID** (GUID or integer) as the **primary key**.
* Store `PhoneNumber` and `PhoneLast6` as **columns**.
* When logging in, Power Apps can look up the employee record with:

  ```powerfx
  LookUp(Employees, PhoneLast6 = TextInput_Phone.Text)
  ```

  and then store the EmployeeID in context for later operations.

---

## 🕒 2. Can I round the hours to 1 minute in **ClockRecord** table?

**Yes — and 1 minute rounding is perfectly fine**.

### Best practice:

* Store the raw timestamp for **ClockInTime** and **ClockOutTime** (with seconds).
* For display or calculation, use Power Fx or Power Automate to round:

  ```powerfx
  RoundDown(DateDiff(ClockInTime, ClockOutTime, Minutes), 1)
  ```

  or

  ```powerfx
  Round(DateDiff(ClockInTime, ClockOutTime, Minutes)/1,0)*1
  ```
* You can also round to 5 or 15 minutes if required by policy — just don’t modify the original data, only the computed field.

---

## ⚠️ 3. What happens if there’s a ClockInTime but **no ClockOutTime**?

This is a **common real-world issue** (e.g., staff forget to clock out).

### Suggested business rule:

* If there’s no ClockOutTime by the end of the day:

  * Option 1: Auto-fill `ClockOutTime = 23:59` that day (and flag it as *“auto-clockout”*).
  * Option 2 (better): Keep it blank and mark `Status = “Incomplete”`.
  * Option 3: Let managers correct it manually in the **Admin Portal**.
I have to do those steps:
If there is no ClockOutTime by the end of the day, I'll do
    1. Keep it blank and mark `Status = “Incomplete”` at the end of the day
    2. Send an email to let manager know and correct it. 

### Implementation tip:

* Add a **Status** column in `ClockRecord`:

  ```
  Status (Choice): ClockedIn / ClockedOut / Incomplete / Adjusted
  ```
* Add a Power Automate daily flow that checks for records with `ClockOutTime = null` and sends a reminder to the manager.

---

## 📅 4. Relationship between **ClockRecord** and **Timesheet**

This is a critical structural choice.

### Concept:

* Each **ClockRecord** is an event (a pair of In/Out times).
* Each **Timesheet** summarizes a week per employee.

### Options:

#### ❌ Option A — Create Timesheet before ClockRecord

This is **not recommended** because it’s unnecessary complexity. Employees don’t know their timesheet before working.

#### ✅ Option B — Create ClockRecords first

Then, when generating a weekly summary (via Power Automate or app logic):

* Group `ClockRecords` by Employee + WeekNumber.
* If no `Timesheet` exists for that employee/week, create one.
* Store total hours (weekday/weekend/public holiday) in `Timesheet`.

This can be done in a **Power Automate weekly job**:

1. Get all ClockRecords where WeekNumber = currentWeek.
2. For each Employee, summarize hours.
3. Create or update that week’s Timesheet record.

### Data model link:

* **ClockRecord**

  * EmployeeID (lookup)
  * TimesheetID (lookup, optional – may be null until processed)
* **Timesheet**

  * EmployeeID (lookup)
  * WeekStartDate
  * TotalWeekdayHours
  * TotalWeekendHours
  * TotalPublicHolidayHours

---

## ✅ Summary of Best Practices

| Design Area            | Recommendation                                 |
| ---------------------- | ---------------------------------------------- |
| Employee Key           | Use GUID or auto number, not phone digits      |
| PhoneLast6             | Use for login lookup only                      |
| Clock rounding         | Round to 1 minute at calculation time          |
| Missing clock-out      | Keep blank + flag incomplete                   |
| Timesheet relationship | Generate after ClockRecords weekly             |
| Automation             | Use Power Automate to create/update Timesheets |



---

## 🕒 1. How to find the record that the employee “just clocked in”?

You want the system to identify whether an employee:

* is currently *clocked in* (but not out yet), or
* has already clocked out and can start a new shift.

### ✅ Best practice (simple & reliable)

When an employee clicks **“Clock In”**, run a lookup like this in Power Apps:

```powerfx
Set(
    varActiveRecord,
    LookUp(
        ClockRecords,
        EmployeeID = varEmployee.EmployeeID && IsBlank(ClockOutTime)
    )
);
```

* If `varActiveRecord` **is blank**, it means there’s no open shift → ✅ allow clock in.
* If `varActiveRecord` **exists**, it means they already clocked in → ❌ prevent another “Clock In” and show message (“You haven’t clocked out yet.”).

### On **Clock Out**:

Use the same logic to find that active record, then patch the ClockOutTime:

```powerfx
Patch(
    ClockRecords,
    LookUp(ClockRecords, EmployeeID = varEmployee.EmployeeID && IsBlank(ClockOutTime)),
    { ClockOutTime: Now(), Status: "ClockedOut" }
)
```

That way you always know which record is the current shift.

### Optional Enhancements:

* You can also store a **ShiftDate** (DateValue of ClockInTime) to make grouping easier.
* Or include `IsActive` column (Yes/No) and set it to true on ClockIn and false on ClockOut.

---

## 📅 2. Timesheet relationship — when and how to populate `TimesheetID`

You’re absolutely right:
If you create ClockRecord first (as the employee clocks in/out), you **don’t yet know which Timesheet record** it will belong to.
So yes, you should **leave `TimesheetID` blank initially**.

---

### 🧭 Step-by-step recommended process:

#### (1) During daily operations

* When an employee clocks in/out, create or update a **ClockRecord**.
* Columns filled:
  `EmployeeID`, `ClockInTime`, `ClockOutTime`, maybe `ShiftDate`.
* Leave `TimesheetID` **empty/null** for now.

#### (2) At end of week (via Power Automate or manual trigger)

Run a scheduled **Power Automate flow** (e.g., Sunday night):

1. **Find all employees** with ClockRecords in that week.
2. For each employee:

   * Check if a Timesheet already exists for that week.

     * If not, **create** it (record `EmployeeID`, `WeekStartDate`, etc.).
3. **Update ClockRecords** for that week:

   * Add the TimesheetID (using `Update a row` or `Update item` action in Power Automate).
   * Calculate total hours grouped by weekday/weekend/public holiday.
   * Patch the Timesheet totals.

That flow can look like this (conceptually):

```
For each Employee:
   Get ClockRecords (WeekStartDate <= ClockInTime <= WeekEndDate)
   If Timesheet not exists:
      Create Timesheet (output TimesheetID)
   For each ClockRecord in week:
      Update ClockRecord.TimesheetID = TimesheetID
   Summarize hours and update Timesheet
```

---

### ⚙️ Implementation Tip

If you’re using **Dataverse**:

* TimesheetID column in ClockRecord should be a **Lookup column** (to Timesheet).
* Updating that via Power Automate is easy using the **Relate row** action.

If you’re using **SharePoint**:

* TimesheetID can be a **Lookup column** or simply a text ID.
* Use “Update item” in Power Automate to populate it.

---

## ✅ Summary

| Question                               | Best Practice                                                                      |
| -------------------------------------- | ---------------------------------------------------------------------------------- |
| How to find “active” ClockRecord       | Lookup `ClockRecord` where `EmployeeID = current user` and `ClockOutTime` is blank |
| When to create Timesheet               | End of week (via Power Automate)                                                   |
| Should TimesheetID be blank initially? | Yes                                                                                |
| How to fill TimesheetID later          | Weekly Power Automate job that updates each ClockRecord with TimesheetID           |
| Data source                            | Works in both Dataverse and SharePoint, Dataverse preferred for clean lookups      |

---

Would you like me to show you an example **Power Automate flow structure** (trigger, actions, logic) for generating and linking weekly Timesheets automatically?


# Sketch wireframes (main screen, admin screen, summary report)

# Design Power automate flow for Weekly summary

Below is a **clear, step-by-step Power Automate design** for generating a **Weekly Summary** (Timesheet) from ClockRecord data.
This design works perfectly with the data model we created earlier (Employee, ClockRecord, Timesheet).

---

# ✅ **Goal of the Flow**

Every week (e.g., Monday at 3 AM), Power Automate should:

1. Get all employees.
2. For each employee:

   * Find all ClockRecords in the previous week.
   * Calculate weekday/weekend/public holiday hours.
   * Sum total hours.
3. Create a **Timesheet** record for each employee.
4. Link each ClockRecord to its TimesheetID.
5. (Optional) Email or store the summary.

---

# 🧩 **Before building the Flow**

Make sure your ClockRecord table includes:

| Column                                  | Type                            |
| --------------------------------------- | ------------------------------- |
| EmployeeID                              | Lookup                          |
| ClockInTime                             | DateTime                        |
| ClockOutTime                            | DateTime                        |
| WorkDurationMinutes / Hours             | Number                          |
| DayType (Weekday/Weekend/PublicHoliday) | Choice                          |
| TimesheetID                             | Lookup (can be blank initially) |

You do *not* need to create weekly Timesheets ahead of time.
They will be created automatically by this weekly flow.

---

# 🔧 **Power Automate Flow: Step-by-Step Design**

## **TRIGGER**

### **1. Recurrence trigger**

* Frequency: **Weekly**
* Day: **Monday**
* Time: e.g. **03:00 AM**

This will calculate the hours for the previous Monday–Sunday.

---

# 🔁 **MAIN FLOW STEPS**

## **2. Initialize Variables**

```
varWeekStart = startOfWeek(utcNow(), 'Monday') - 7 days
varWeekEnd   = startOfWeek(utcNow(), 'Monday')
```

In Power Automate expression:

**Week Start (previous Monday 00:00):**

```fx
startOfWeek(addDays(utcNow(), -7), 'Monday')
```

**Week End (previous Sunday 23:59):**

```fx
addSeconds(startOfWeek(utcNow(), 'Monday'), -1)
```

---

## **3. Get All Employees**

Use **List Rows** (Dataverse / SharePoint).

---

## **4. Apply to Each: Employee**

Loop through all employees.

Inside the loop:

---

## **4.1 Get ClockRecords for this employee in the week**

Filter:

```text
EmployeeID eq @{items('Apply_to_each')?['employeeid']}
and ClockInTime ge varWeekStart
and ClockInTime le varWeekEnd
and Status eq 'ClockedOut'
```

(We exclude incomplete shifts.)

---

## **4.2 Initialize summary variables**

Inside the loop, create these variables:

```
varWeekdayMinutes = 0
varWeekendMinutes = 0
varHolidayMinutes = 0
varTotalMinutes = 0
```

---

## **4.3 Loop through each ClockRecord**

Add a nested **Apply to each ClockRecord**:

For each item:

### **Add minutes to different buckets**

```
If DayType == 'Weekday' → add to varWeekdayMinutes
If DayType == 'Weekend' → add to varWeekendMinutes
If DayType == 'Public Holiday' → add to varHolidayMinutes
Always add to varTotalMinutes
```

Expressions:

```fx
add(variables('varWeekdayMinutes'), items('Apply_to_each_record')?['WorkDurationMinutes'])
```

Repeat for other categories.

---

# 📄 **5. Create Timesheet Record**

After finishing the loops for an employee:

Create new Timesheet row with:

| Field             | Value                  |
| ----------------- | ---------------------- |
| Employee          | employee ID            |
| WeekStart         | varWeekStart           |
| WeekEnd           | varWeekEnd             |
| TotalWeekdayHours | varWeekdayMinutes / 60 |
| TotalWeekendHours | varWeekendMinutes / 60 |
| TotalHolidayHours | varHolidayMinutes / 60 |
| TotalHours        | varTotalMinutes / 60   |

Store the **TimesheetID** output for linking records.

---

# 🔗 **6. Update ClockRecords to include TimesheetID**

Now update all ClockRecords from step 4.1:

Use **Update Row**:

```
TimesheetID = @{outputs('Create_Timesheet')?['TimesheetID']}
```

This links every shift record to the weekly timesheet.

---

# 📧 **7. (Optional) Notify Manager**

Send email:

* Total hours per employee
* Public holiday hours
* Weekend hours
* Missing clock-outs (if any)

---

# 🎉 **Flow is complete!**

This gives you:

✔ Automatically generated weekly summaries
✔ Automatic calculation of weekday/weekend/public holiday
✔ Clean link between ClockRecords → Timesheet
✔ Easy export to Xero

---

# 🔍 Want me to generate a **diagram** for this flow?

I can create:

* A **Mermaid flowchart**
* A **step-by-step blueprint diagram**
* Or a **Power Automate-style pseudo-JSON**

Just tell me which format you prefer!
