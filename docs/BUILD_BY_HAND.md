# Build-by-Hand Exercises

These parts are deliberately **not** in the deployable metadata. Admins build them in Setup every day, and interviewers often ask you to explain exactly these steps. Do them in order after deploying the project.

When you finish an exercise, retrieve it into the repo so your GitHub shows your own work:
```bash
sf project retrieve start --metadata ApprovalProcess Report Dashboard DuplicateRule MatchingRule
git add . && git commit -m "Add approval process, reports and dashboard"
```

---

## Exercise 1: Add the Applications related list to the Student page
1. Open any Student record → gear icon → **Edit Object** (or Setup → Object Manager → **Student**).
2. **Page Layouts** → **Student Layout** → **Related Lists**.
3. Drag **Placement Applications** onto the layout. Click the wrench and add the columns **Company**, **Status**, **Package (LPA)** and **Applied Date**.
4. Save. Open a student and check that the applications now appear.

**You learn:** page layouts and related lists.

## Exercise 2: Customize the Student record page with Lightning App Builder
1. Open a Student → gear → **Edit Page**.
2. Add a **Highlights Panel**, and add **Path** using the Placement Status field.
3. Add a **Rich Text** component that shows "Not eligible yet" using **component visibility**, with the filter `Total Applications = 0`.
4. **Save** → **Activate** → **Org Default**.

**You learn:** Lightning App Builder, record pages, dynamic component visibility.

## Exercise 3: Create users, a profile and a role
1. Setup → **Profiles** → clone **Standard User** → name it **Placement User**.
2. Setup → **Users** → **New User** twice (Developer Edition gives you a couple of extra licences):
   - *Priya (Placement Officer)*: Profile **Placement User**, Role **Placement Officer**, then assign permission set **Placement Officer**.
   - *Test Student*: Profile **Placement User**, no role, then assign permission set **Placement Student**.
3. Log in as each user (Setup → Users → **Login**; first enable "Administrators Can Log in as Any User" in Login Access Policies). Compare what each user can see and edit.

**You learn:** the difference between profiles, roles and permission sets, and how they work together.

## Exercise 4: Approval process for accepting an offer
Goal: a student can't accept an offer until a Placement Officer approves it.
1. Setup → **Approval Processes** → object **Placement Application** → **Use Standard Setup Wizard**.
2. Name: `Offer Acceptance Approval`.
3. Entry criteria: `Status equals Offered`.
4. Approver: **Let the submitter choose the approver manually**.
5. **Final Approval Action** → Field Update → `Status = Offer Accepted`.
6. **Final Rejection Action** → Field Update → `Status = Rejected`.
7. Initial Submission Action: keep **Record Lock**.
8. **Activate** it. Add the **Submit for Approval** button to the application layout if it's missing.
9. Test: set an application to *Offered* → Submit for Approval → approve it as the officer. Check that the Flow then marks the student **Placed**.

**You learn:** approval processes, field updates, record locking, and how an approval can trigger a Flow.

## Exercise 5: Reports
Setup isn't needed. Open the **Reports** tab → **New Report**.
1. **Applications by Company and Status:** report type *Placement Applications with Student*. Group rows by **Company** and columns by **Status** (this makes it a Matrix report).
2. **Placed vs Not Placed by Branch:** report type *Students*. Group rows by **Branch** and **Placement Status**, and add a **record count** chart.
3. **Top Packages Offered:** report type *Placement Applications*, filter `Status = Offer Accepted`, sorted by **Package (LPA)** descending. Show the Max of Package.

Save all of them in a new public folder called **Placement Reports**.

**You learn:** report types, tabular, summary and matrix reports, filters, groupings and summaries.

## Exercise 6: Dashboard
1. **Dashboards** tab → **New Dashboard** → `Placement Overview` → save it in the **Placement Reports** folder.
2. Add components:
   - **Donut:** Placed vs Not Placed (report 2)
   - **Stacked bar:** Applications by Company and Status (report 1)
   - **Metric:** highest package offered (report 3)
   - **Table:** Top Packages Offered (report 3)
3. Set **View dashboard as: Me**, then Save.

**You learn:** dashboards, components, and the running user setting.

## Exercise 7: Duplicate rule for students
1. Setup → **Matching Rules** → New → object **Student**, matching on **Email** (exact) → **Activate**.
2. Setup → **Duplicate Rules** → New → **Student** → use that matching rule → Action on create: **Block** → **Activate**.
3. Test by creating a student with an existing email.

**You learn:** data quality with matching and duplicate rules.

## Exercise 8: Import data with Data Import Wizard
1. Make a CSV with 5 new students (Name, Roll Number, Email, Branch, CGPA).
2. Setup → **Data Import Wizard** → Custom objects → **Students** → Add new records → map fields → Start import.

**You learn:** Data Import Wizard, and when to use it instead of Data Loader (more than 50,000 records, or exports and deletes, need Data Loader).

## Exercise 9 (stretch): Email alert in the Flow
Open the Flow in **Flow Builder** → **Save As New Version** → after "Mark Student Placed", add a **Send Email** action that congratulates the student (use `$Record.Student__r.Email__c`). Then **Activate** the new version.

**You learn:** editing and versioning flows, and flow actions.
