# Learning Guide

Work through this file next to the org. For each piece, open it in **Setup → Object Manager** (or **Flows**, **Permission Sets** and so on), read the explanation, then try the interview question out loud.

---

## 1. Custom objects
**Where:** Object Manager → Student / Company / Placement Application

- An **object** is a table. A **record** is a row. A **field** is a column.
- *Student* and *Company* use a **Text** name field. *Placement Application* uses an **Auto Number** name (`APP-0001`), because applications have no natural name.
- Track Field History is enabled, and CGPA, Package and Status are tracked. Look at the **History** related list after editing a record.

**Interview:** *What's the difference between a standard and a custom object?* Standard objects (Account, Contact) come with Salesforce. Custom objects end in `__c` and are created by admins.

## 2. Relationships
**Where:** Placement Application → Fields → Student, Company

| | Master-Detail (Student) | Lookup (Company) |
|---|---|---|
| Child required? | Always | Optional (made required here) |
| Delete parent | Deletes the children (cascade) | Blocked here (`Restrict`) |
| Sharing | Child inherits the parent's (Controlled by Parent) | Independent |
| Roll-up summary on parent | **Yes** | No (needs a Flow or Apex) |
| Owner on child | No owner field | Has its own owner |

**Interview:** *Why did you use master-detail for Student but lookup for Company?* An application can't exist without a student, and I wanted roll-up counts on Student. A company should never be deleted while it has applications, so I used a lookup with a restrict delete.

## 3. Roll-up summary fields
**Where:** Student → Total Applications, Offers Accepted

- `COUNT` of child records. **Offers Accepted** adds a filter: `Status = Offer Accepted`.
- These only work on the master side of a master-detail relationship.

## 4. Formula fields
**Where:** Placement Application → Student CGPA, Package (LPA), Eligible

- **Cross-object formulas** read values from parent records: `Student__r.CGPA__c`.
- `__c` is the field and `__r` is the relationship you travel through.
- *Eligible* is a checkbox formula: `Student__r.CGPA__c >= Company__r.Minimum_CGPA__c`.
- Formula values are calculated when the record is read and never stored. That's why they can't trigger a Flow.

## 5. Validation rules
**Where:** each object → Validation Rules

| Rule | Formula | Idea |
|---|---|---|
| CGPA 0–10 | `OR(CGPA__c < 0, CGPA__c > 10)` | Basic data quality |
| CGPA cut-off | `AND(ISNEW(), Student__r.CGPA__c < Company__r.Minimum_CGPA__c)` | `ISNEW()` runs it only on create; cross-object check |
| Accepted offer locked | `AND(ISCHANGED(Status__c), ISPICKVAL(PRIORVALUE(Status__c), "Offer Accepted"))` | `PRIORVALUE()`, `ISPICKVAL()` for picklists |

**Remember:** the formula describes the **error**. If it's TRUE, the save is blocked.

**Interview:** *Why do picklists need `ISPICKVAL`?* Picklists aren't text, so you can't compare them with `=`. Use `ISPICKVAL(field, "value")` or `TEXT(field)`.

## 6. Record-triggered Flow
**Where:** Setup → Flows → *Application Offer Accepted – Mark Student Placed* → open it in Flow Builder

- **Trigger:** Placement Application is created or updated.
- **Entry condition:** `Status = Offer Accepted`, and *only when a record is updated to meet the condition*. This stops it from running again on every later edit.
- **Optimized for:** *Actions and Related Records*, which is an **after-save** flow. It has to be after-save because it updates a *different* record (the Student).
- **Element:** Update Records → Student where `Id = {!$Record.Student__c}` → `Placement Status = Placed`.

**Interview:** *Before-save vs after-save flow?* Before-save ("Fast Field Updates") only changes the record that triggered it, and it's very fast. After-save can update other records, create records, send emails and call actions.

**Interview:** *Workflow Rules and Process Builder?* They're retired. Salesforce recommends Flow for all new automation.

## 7. Security model (most asked in interviews)
Salesforce access is layered:

```
Org-Wide Defaults (the most restrictive baseline)
  → Role Hierarchy (managers see their subordinates' records)
    → Sharing Rules (open access to groups or roles)
      → Manual Sharing
Profiles + Permission Sets decide WHAT you can do (object, field and app permissions)
OWD + Roles + Sharing decide WHICH records you can see
```

**In this project:**
- **OWD:** Student = *Private*, Company = *Public Read Only*, Application = *Controlled by Parent*.
- **Roles:** Placement Head → Placement Officer. The Head automatically sees what Officers see.
- **Sharing rule:** a criteria-based rule shares *every* Student with the Placement Officer role (Read/Write).
- **Permission sets:** *Placement Officer* gets full CRUD and View All. *Placement Student* gets read-only access to Company and Student, and can create and edit Applications.

**Interview:** *Profile vs role vs permission set?*
- **Profile:** a user's baseline permissions. Every user has exactly one.
- **Permission set:** extra permissions added on top. A user can have many. Salesforce is moving permissions off profiles and onto permission sets.
- **Role:** decides record visibility through the hierarchy, not what you can do.

**Interview:** *A user can't see a record. How do you troubleshoot?* Check the object permission (profile or permission set), the OWD, the role hierarchy and the sharing rules. Then use the record's **Sharing Hierarchy** button to see why the user does or doesn't have access.

## 8. Lightning app, tabs and page layouts
- **Placement Tracker** app: tabs for Students, Companies, Applications, Reports and Dashboards.
- **Page layouts** control which fields appear and whether they're required or read-only. Formula and roll-up fields are always read-only.
- **Lightning record pages** (App Builder) control the arrangement of components. See BUILD_BY_HAND, Exercise 2.

## 9. Data management
- `Roll_Number__c` is **Unique** and an **External ID**, so imports can *upsert* by roll number without creating duplicates.
- **Data Import Wizard:** up to 50,000 records, standard and custom objects, with duplicate checking.
- **Data Loader:** up to 5 million records, and it can also export and delete.

---

## Talking about it in an interview (sample answer)
> "I built a Student Placement Tracker in a Developer org. It has three custom objects: Student, Company and Placement Application, linked by master-detail and lookup relationships. Roll-up summaries count each student's applications and offers, and cross-object formulas show eligibility. Validation rules block applications below the company's CGPA cut-off and lock accepted offers. A record-triggered flow marks the student as Placed when an offer is accepted, and an approval process lets a placement officer approve offer acceptance. For security, Student records are private by OWD, officers get access through a role and a criteria-based sharing rule, and two permission sets separate officer and student access. I finished with reports and a dashboard showing placements by branch and company."

Only say this once you've deployed it and done the BUILD_BY_HAND exercises yourself.
