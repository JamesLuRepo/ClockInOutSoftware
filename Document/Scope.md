# Project goal and expected outcomes
1. Create a clock in/out software to record employee's clock in and out time. 
2. At the end of each week. If all employees clocked out, email the result to my email address.
3. Calculate the working time and devide them into weekday, weekend and public holiday.
4. The report can also calculate the total salary based on their salary.

# Identify users
All users are employees. They should be classified into different positions. Each employee can have multiple positions. Only one account is required because I'll put this app in the shop and keeping open.

# Define key features (MVP + optional future features)
## ✅ Phase 1 — Essential (MVP) Features

At the beginning, focus on stability and easy daily use:

### 1. **Employee Login**

* Login by **phone number** (used as password).
* Optional PIN or biometric if you later deploy to company tablet.

### 2. **Clock In / Clock Out**

* Buttons to clock in/out.
* Record:

  * Employee
  * Date/time
  * Type (Clock In / Clock Out)
  * Device location (optional, to ensure it's done at shop)
  * Comments (optional: e.g., “covering for John”)

### 3. **Work Session Tracking**

* Automatically calculate **session duration** (Clock Out – Clock In).
* Save it into a “Timesheet” table.

### 4. **Time Classification**

Use Power Automate or Dataverse logic to auto-tag:

* **Weekday**
* **Weekend (Sat/Sun)**
* **Public Holiday** (use an Australian holiday list — can store in a table or pull via API).

### 5. **Weekly Summary**

* Automatically summarize total hours:

  * Weekday hours
  * Weekend hours
  * Public holiday hours
* Email a summary report to you every Sunday night (via Power Automate).
* Optional export to Excel.

---

## 🌱 Phase 2 — Useful Add-ons

Once the core system runs well, add features that boost management efficiency:

| Category                   | Feature         | Description                                                                       |
| -------------------------- | --------------- | --------------------------------------------------------------------------------- |
| **Shift management**       | Planned roster  | Manager sets planned shifts; employees see their schedule.                        |
| **Late / early detection** | Clock-in check  | If employee clocks in late (>10 mins) or early, flag in report.                   |
| **Break tracking**         | Start/End break | Adds accuracy for paid/unpaid time.                                               |
| **Dashboard**              | Manager view    | Summary dashboard: who’s currently working, hours this week, missing clock-outs.  |
| **Leave tracking**         | Request leave   | Employees request leave; manager approves in app.                                 |
| **Geofencing (optional)**  | Verify location | Limit clock-in to within restaurant geofence (via Power Apps latitude/longitude). |
| **Notifications**          | Reminder        | Auto-reminder to clock out after closing time if someone forgot.                  |
| **Data export**            | Integration     | Weekly CSV export for payroll software like Xero or MYOB.                         |

---

## 🧱 Suggested Tables (Dataverse or SharePoint)

| Table                   | Key Columns                                                                     | Description                  |
| ----------------------- | ------------------------------------------------------------------------------- | ---------------------------- |
| **Employee**            | Name, Phone (Password), Role, Hourly Rate                                       | Basic info                   |
| **ClockRecord**         | Employee, Type (In/Out), TimeStamp, Location, Note                              | Raw punch data               |
| **Timesheet**           | Employee, Date, Start, End, Duration, Category (Weekday/Weekend/Public Holiday) | Calculated results           |
| **HolidayList**         | Date, Description                                                               | Reference for classification |
| **Roster** *(optional)* | Employee, ShiftDate, StartTime, EndTime                                         | Planned shifts               |

---

## ⚙️ Tools Used

* **Power Apps Canvas App** — UI for employees to clock in/out.
* **Dataverse / SharePoint** — store employee and attendance data.
* **Power Automate** — weekly summary flow + classification logic.
* **Power BI (optional)** — dashboard of attendance trends and wage costs.

---

## 💡 Pro Tips

* Keep interface **one-screen simple** (Clock In / Out only). Use role-based access for admin features.
* Use `Now()` to record timestamps server-side for consistency.
* Build a small “Admin Portal” (second app or admin screen) for editing records and viewing reports.
* Log “Missed clock out” errors automatically (no out after certain hours).
* If using a shared device, add **auto-logout** after each operation.


# Define constraints (device, internet connection, user tech level)
## 🧩 Option 1 — **Use Power Apps on the shop computer (via browser)**

✅ **Simple, zero extra cost, works right away.**

### 🔧 How it works

1. You build the Canvas App (responsive layout).
2. Publish it in your Microsoft environment (via Power Apps web).
3. In your shop, open a browser (e.g., Edge or Chrome) and go to
   👉 [https://make.powerapps.com/play](https://make.powerapps.com/play)
4. Choose your app, and keep it open on that computer or tablet.

### 👨‍🍳 Employees’ experience

* Each employee walks to the computer.
* Enters their **phone number** (used as password/login).
* Clicks “Clock In” or “Clock Out.”
* App records the time and stores it in Dataverse/SharePoint.

### 🔒 Tips for shared-computer setup

* Create a **shared user account** (e.g., “WokitupErindaleClock”) that’s always logged in to Power Apps.
* Inside your app, add your **own login screen** (phone number field + “Continue” button).
  → This avoids logging in/out of Microsoft accounts every time.
* Optionally auto-log out after each action (navigate back to login screen).

### 🖥️ Requirements

* Internet connection (Power Apps is cloud-based).
* Computer or tablet with browser access.

---

## 💡 Option 2 — **Dedicated tablet at the front counter**

✅ Most restaurants prefer this setup.

### Why

* It feels like a punch clock terminal.
* Employees just tap, clock in, and walk away.
* You can use **Power Apps mobile app (iOS/Android)** for better UX.

### Setup

* Mount a cheap tablet (e.g., iPad, Surface Go, Samsung Tab).
* Install **Power Apps mobile app** and pin your app to home screen.
* Enable “Kiosk Mode” (optional) to prevent using other apps.
* Keep it plugged in and always on the clock-in screen.

### Benefits

* Simple, fast, touch-friendly.
* No typing URL each time.
* Works offline for short periods (Power Apps caches data).

---

## ⚙️ Option 3 — **QR Code Clock-in (semi-automated)**

🚀 Slightly advanced, but slick for larger teams.

### How it works

* Each employee has a **QR code badge** (contains their phone number or ID).
* The Power Apps app uses the **camera** to scan QR code → instantly logs them in → records clock-in time.

### Benefits

* Fast and no typing errors.
* More secure (less chance of one person clocking in for another).
* Looks professional.

### Requirements

* Device with camera (tablet or phone).
* QR code generation for each employee (simple Power Automate flow can do this).

---

## 💡 Option 4 — **Use Microsoft Forms + Power Automate (lightweight fallback)**

⚡ Quick and reliable backup if Power Apps is offline.

### Setup

* Make a simple Microsoft Form:

  * Name
  * Phone number
  * Clock In / Clock Out choice
* Power Automate receives form submissions and logs to Dataverse or Excel.

### Use Case

* Useful if your shop has unreliable Wi-Fi or you want a “Plan B” for clocking.

---

## 🧠 Recommendation (for your restaurant)

Since you already use Power Platform and have staff on-site:

| Factor        | Recommendation                                   |
| ------------- | ------------------------------------------------ |
| Daily use     | **Tablet at counter** (Option 2)                 |
| Backup        | Keep browser version open on computer (Option 1) |
| Security      | Use in-app phone number login screen             |
| Later feature | Add optional QR code login for speed             |

# Choose data source (Dataverse or Sharepoint)
I'll use Dataverse.

# Define reporting frequency
Reporting frequency is weekly.

# Confirm app access method
Currently, we use PC. Powerapps desktop and also browser.

