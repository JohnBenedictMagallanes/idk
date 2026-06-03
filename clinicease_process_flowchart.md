# ClinicEase - Complete Process Flowchart

## System Overview
ClinicEase is a Clinic Management System for managing patient records, medical transactions, doctor schedules, billing, and follow-ups through Receptionist and Doctor portals.

---

## 🔐 AUTHENTICATION PROCESS

```
User Access
    ↓
[index.php] Session Check
    ├─→ YES: User ID exists? → Redirect to Dashboard
    └─→ NO: Redirect to Login
         ↓
    [login.php] Check if already logged in
         ├─→ YES: Redirect to Dashboard
         └─→ NO: Show Login Form
              ↓
         Form Submission (POST)
              ├─→ Validate: Email & Password filled?
              │   ├─→ NO: Show Error Message
              │   └─→ YES: Continue
              ├─→ User::login(email, password)
              │   ├─→ Check: Email exists in users table?
              │   │   ├─→ NO: Return "No account found"
              │   │   └─→ YES: Continue
              │   ├─→ Check: Password matches?
              │   │   ├─→ NO: Return "Incorrect password"
              │   │   └─→ YES: Continue
              │   └─→ Create Session Variables
              │       ├─ user_id
              │       ├─ name
              │       ├─ role (receptionist/doctor)
              │       └─ email
              └─→ Redirect to Dashboard
              
    [logout.php] Logout
         ↓
    User::logout()
         ├─→ Session destroy
         └─→ Redirect to Login
```

---

## 📊 DASHBOARD PROCESS

```
[dashboard.php] - Authentication Required
     ↓
Role-Based Dashboard Initialization
     ├─────────────────────────────────────────────┬─────────────────────────────────┐
     │ RECEPTIONIST DASHBOARD                      │ DOCTOR DASHBOARD                │
     └────────────────────┬────────────────────────┴────────────┬────────────────────┘
                          ↓                                     ↓
            Transaction::getUpcoming()             Transaction::getUpcomingByDoctor()
            FollowUp::getScheduled()               FollowUp::getUpcomingByDoctor()
                          ↓                                     ↓
            Display upcoming visits                Display doctor's upcoming visits
            Display scheduled follow-ups           Display follow-ups for doctor
            Display patient queue                  Display assigned patients
            Display billing alerts                 Display pending consultations
                          ↓                                     ↓
            Available Actions:                     Available Actions:
            ├─ Manage Patients                     ├─ View Patient Records
            ├─ Schedule Appointments               ├─ Complete Consultations
            ├─ Manage Doctors                      ├─ Add Prescriptions
            ├─ Create Billing                      ├─ View/Update Doctor Profile
            ├─ Export/Import Data                  ├─ View Scheduled Follow-ups
            └─ View Reports                        └─ View Billing Info
```

---

## 👥 PATIENT MANAGEMENT PROCESS

```
[patients/index.php] - Staff Access Only (Receptionist/Doctor)
     ↓
Authorization Check: isStaff()?
     ├─→ NO: Redirect to Dashboard
     └─→ YES: Continue
          ↓
Initialization:
     ├─ Patient::getAllWithSummary()
     ├─ Filter by Doctor (if Doctor viewing)
     └─ Sort patients alphabetically
          ↓
     ┌────────────────────────────────────────┬─────────────────────────────────┐
     │                                        │                                 │
     ↓ RECEPTIONIST ONLY                      ↓ BOTH RECEPTIONIST & DOCTOR     │
                                              │                                 │
[POST] Create Patient                        [GET] View Patients List          │
     ├─ Validate:                            ├─ Display table with:            │
     │  ├─ Name (required)                    │  ├─ Patient Name                │
     │  ├─ Gender (valid enum)                │  ├─ Contact Info                │
     │  ├─ Email (optional)                   │  ├─ Blood Type                  │
     │  ├─ Phone (optional)                   │  ├─ Recent Visit Date           │
     │  ├─ Address (optional)                 │  ├─ Total Visits Count          │
     │  ├─ Date of Birth (optional)           │  ├─ Last Transaction Status     │
     │  ├─ Blood Type (optional)              │  └─ Action buttons
     │  ├─ Allergies (optional)               │
     │  └─ Medical History (optional)         ├─ Search & Filter               │
     ├─ Patient::create(...)                  ├─ Pagination (if many)          │
     ├─ setFlash('success')                   └─ Sort by name                  │
     └─ Redirect to index.php                 │
                                              ↓                                 │
     [GET] Delete Patient                    [GET] View Patient Details        │
     ├─ Patient::delete(id)                   ├─ [patients/view.php]           │
     ├─ setFlash('success')                   ├─ Display:                      │
     └─ Redirect to index.php                 │  ├─ Demographics               │
                                              │  ├─ Medical History             │
                                              │  ├─ Blood Type                  │
                                              │  ├─ Allergies                   │
                                              │  ├─ Contact Details             │
                                              │  └─ Visit History               │
                                              ├─ Associated Transactions       │
                                              ├─ Associated Follow-ups         │
                                              └─ Edit/Manage buttons           │
```

---

## 📅 TRANSACTION (VISIT/APPOINTMENT) PROCESS

```
Transaction Workflow
     ↓
[GET] View All Transactions
     ├─ Transaction::getAll()
     ├─ Display with filters:
     │  ├─ By Status (Pending, Completed, Cancelled)
     │  ├─ By Date Range
     │  ├─ By Doctor
     │  └─ By Patient
     └─ Show table with:
        ├─ Patient Name
        ├─ Doctor Name
        ├─ Visit Date/Time
        ├─ Visit Type
        ├─ Status
        └─ Action buttons
          ↓
[GET] View Transaction Details
     ├─ Transaction::getById()
     └─ Display:
        ├─ Patient Information
        ├─ Doctor Information
        ├─ Visit Details (Date, Time, Type)
        ├─ Reason for Visit
        ├─ Doctor Notes
        ├─ Prescription (if completed)
        ├─ Requirements
        └─ Status History
          ↓
[RECEPTIONIST] Create Transaction (Schedule Appointment)
     ├─ [transactions/create.php]
     ├─ Validate:
     │  ├─ Patient ID (required, exists)
     │  ├─ Doctor ID (required, exists)
     │  ├─ Visit Date (required, valid date)
     │  ├─ Visit Time (required, valid time)
     │  ├─ Visit Type (optional: Consultation, Follow-up, Check-up)
     │  └─ Reason (optional)
     ├─ Check Doctor Schedule Availability
     ├─ Transaction::create()
     ├─ setFlash('success')
     └─ Redirect to list
          ↓
[DOCTOR] Complete Transaction
     ├─ [transactions/complete.php]
     ├─ Validate:
     │  ├─ Doctor Notes (required)
     │  ├─ Prescription (required)
     │  └─ Status = 'Completed'
     ├─ Transaction::update(status, notes, prescription)
     ├─ Update completed_at timestamp
     ├─ setFlash('success')
     └─ Redirect to dashboard
          ↓
[RECEPTIONIST] Cancel Transaction
     ├─ Verify Authorization
     ├─ Transaction::updateStatus('Cancelled')
     ├─ Notification (optional)
     ├─ setFlash('success')
     └─ Redirect
```

---

## 📝 FOLLOW-UP MANAGEMENT PROCESS

```
[follow-ups/index.php] - Staff Access Only
     ↓
Initialization:
     ├─ FollowUp::getAll() or getUpcoming()
     ├─ Filter by Doctor (if Doctor viewing)
     └─ Group by Status
          ↓
     ┌──────────────────────────────────────────────────────────┐
     │                                                          │
     ↓ RECEPTIONIST ONLY                    ↓ BOTH STAFF       │
                                            │                   │
[POST] Create Follow-up                    [GET] View List      │
     ├─ Link to completed transaction      ├─ Filter by:       │
     ├─ Validate:                          │  ├─ Status        │
     │  ├─ Patient ID (required)           │  ├─ Doctor        │
     │  ├─ Doctor ID (required)            │  ├─ Date Range    │
     │  ├─ Scheduled Date (required)       │  └─ Patient       │
     │  ├─ Notes (required)                └─ Table display    │
     │  ├─ Type (optional)                 │                   │
     │  └─ Priority (optional)             ↓                   │
     ├─ FollowUp::create()                 [GET] View Details  │
     ├─ setFlash('success')                ├─ Patient Info     │
     └─ Redirect                            ├─ Doctor Info      │
                                            ├─ Scheduled Date   │
     [GET] Delete Follow-up                ├─ Follow-up Notes  │
     ├─ FollowUp::delete(id)                ├─ Status           │
     ├─ setFlash('success')                ├─ Priority         │
     └─ Redirect                            └─ Completion Date  │
                                            │                   │
     [POST] Update Follow-up                ↓                   │
     ├─ Change Status                       [POST] Update       │
     ├─ Add notes                           ├─ Update Status    │
     ├─ Set completion date                 ├─ Add Notes        │
     └─ FollowUp::update()                  ├─ Mark Completed   │
                                            └─ Redirect         │
```

---

## 💳 BILLING PROCESS

```
[billing/index.php] - Receptionist Only
     ↓
Authorization: requireReceptionist()
     ├─→ NO: Redirect to Dashboard
     └─→ YES: Continue
          ↓
Initialization:
     ├─ Billing::getAll(filter)
     ├─ Billing::getBillableVisits()
     ├─ Billing::getBillableFollowUps()
     └─ Display "Awaiting Billing" section
          ↓
     ┌────────────────────────────────────┬─────────────────────────┐
     │                                    │                         │
     ↓ Awaiting Billing                   ↓ Existing Bills          │
                                          │                         │
Display unbilled completed transactions   Display all bills with:   │
Display unbilled completed follow-ups     ├─ Bill ID                │
                                          ├─ Patient Name           │
     ↓                                    ├─ Amount                 │
     │                                    ├─ Status                 │
[GET] Create New Bill                     ├─ Date Created           │
     └─ [billing/create.php]              ├─ Due Date               │
        ├─ Display available items:       └─ Action buttons         │
        │  ├─ Completed visits            │                         │
        │  └─ Completed follow-ups        ↓                         │
        ├─ Validate selection             [GET] View Bill Details   │
        ├─ Billing::create()              ├─ Bill items            │
        ├─ Billing::addItem()             ├─ Itemized breakdown    │
        ├─ Calculate total                ├─ Total Amount          │
        ├─ setFlash('success')            ├─ Status                │
        └─ Redirect to list               ├─ Payment Info (if paid) │
                                          └─ Print/Export options  │
     [GET] Delete Bill                    │                         │
     ├─ Billing::delete(id)               ↓                         │
     ├─ setFlash('success')               [POST] Update Bill        │
     └─ Redirect to list                  ├─ Mark as paid          │
                                          ├─ Update status         │
                                          ├─ Add payment note       │
                                          └─ Billing::update()     │
```

---

## 👨‍⚕️ DOCTOR MANAGEMENT PROCESS

```
[doctors/list.php] - Receptionist Only
     ↓
Authorization: requireReceptionist()
     ├─→ NO: Redirect
     └─→ YES: Continue
          ↓
Initialization:
     ├─ Doctor::getAll()
     └─ Display all doctors with:
        ├─ Name
        ├─ Specialization
        ├─ Bio
        ├─ Contact Info
        ├─ Schedule
        ├─ Active Patients Count
        └─ Action buttons
          ↓
     ┌─────────────────────────┬────────────────────────┐
     │                         │                        │
     ↓ Create Doctor           ↓ Edit Doctor Info       │
                               │                        │
[GET] doctors/create.php       [GET] doctors/edit.php   │
     ├─ Display form with:     ├─ Load current data     │
     │  ├─ Name                ├─ Display form          │
     │  ├─ Email               ├─ Allow update of:      │
     │  ├─ Phone               │  ├─ Name               │
     │  ├─ Address             │  ├─ Phone              │
     │  ├─ Specialization      │  ├─ Specialization    │
     │  ├─ Bio                 │  └─ Bio                │
     │  └─ Password            └─ Doctor::update()     │
     │                         │                        │
     ├─ [POST] Create         ↓                        │
     │  ├─ Validate            [GET] Delete Doctor      │
     │  ├─ User::create()       ├─ Verify no active     │
     │  ├─ Doctor::create()     │  patients/transactions │
     │  ├─ Doctor::setSchedule()│ ├─ Doctor::delete()  │
     │  ├─ setFlash()           ├─ setFlash()          │
     │  └─ Redirect            └─ Redirect             │
```

---

## 🗓️ DOCTOR SCHEDULE PROCESS

```
Doctor Schedule Management
     ↓
[schedules/list.php]
     ├─ Display doctor's schedule
     ├─ Show days and hours
     ├─ Display list of available time slots
     └─ Show booked appointments
          ↓
[POST] Schedule::create() - Receptionist
     ├─ Validate:
     │  ├─ Doctor ID (required, exists)
     │  ├─ Day of Week (required, valid enum)
     │  ├─ Start Time (required, valid format)
     │  └─ End Time (required, after start time)
     ├─ Check for conflicts
     ├─ Schedule::create()
     ├─ setFlash('success')
     └─ Redirect
          ↓
[GET] Schedule::getByDoctor()
     ├─ Display weekly view
     ├─ Show booked time slots
     ├─ Highlight available slots
     └─ Allow delete/edit
          ↓
[POST] Delete Schedule
     ├─ Verify no appointments scheduled
     ├─ Schedule::delete(id)
     └─ Redirect
```

---

## 👔 RECEPTIONIST MANAGEMENT PROCESS

```
[receptionists/list.php] - Admin/Manager Only
     ↓
Display all receptionists:
     ├─ Name
     ├─ Email
     ├─ Phone
     ├─ Status
     └─ Action buttons
          ↓
     ┌──────────────────────────┬────────────────────────┐
     │                          │                        │
     ↓ Add Receptionist         ↓ Edit Receptionist      │
                                │                        │
[GET] receptionists/create.php  [GET] receptionists/edit.php
     ├─ Display form:          ├─ Load current data     │
     │  ├─ Name                ├─ Display form          │
     │  ├─ Email               ├─ Allow edit:           │
     │  ├─ Phone               │  ├─ Name               │
     │  ├─ Address             │  ├─ Phone              │
     │  ├─ Password            │  └─ Address            │
     │  └─ Confirm Password    └─ User::update()       │
     │                         │                        │
     ├─ [POST] Create          ↓                        │
     │  ├─ Validate            [GET] Delete Receptionist
     │  ├─ User::create()       ├─ User::delete()       │
     │  ├─ setFlash()           ├─ setFlash()           │
     │  └─ Redirect            └─ Redirect             │
```

---

## 📤 DATA IMPORT/EXPORT PROCESS

```
[xml/export.php] - Receptionist Only
     ↓
[POST] Export Data
     ├─ Select export type:
     │  ├─ Patients
     │  ├─ Doctors
     │  ├─ Transactions
     │  ├─ Billing
     │  ├─ Follow-ups
     │  └─ All Data
     ├─ Fetch data from database
     ├─ Convert to XML format
     ├─ Generate downloadable file
     ├─ Set headers for download
     └─ Stream file to user
          ↓
[xml/import.php] - Receptionist Only
     ↓
[POST] Import Data
     ├─ Validate:
     │  ├─ File type is XML
     │  ├─ File size < limit
     │  ├─ XML structure valid
     │  └─ Required fields present
     ├─ Parse XML file
     ├─ Process records:
     │  ├─ Insert new records
     │  ├─ Update existing (by email/ID)
     │  └─ Skip duplicates
     ├─ Validate data integrity
     ├─ Log import operation
     ├─ setFlash() with results
     └─ Redirect to dashboard
```

---

## 📋 REQUIREMENTS TRACKING PROCESS

```
Requirements (for visits/transactions)
     ↓
[GET] View Requirements
     ├─ Display list filtered by:
     │  ├─ Status (Pending, Submitted, Verified)
     │  ├─ Due Date
     │  └─ Transaction
     └─ Show requirement details
          ↓
[POST] Create Requirement
     ├─ Link to transaction
     ├─ Validate:
     │  ├─ Requirement Type (required)
     │  ├─ Description (required)
     │  └─ Due Date (optional)
     ├─ Requirement::create()
     └─ Redirect
          ↓
[POST] Submit Requirement
     ├─ Update submission_date
     ├─ Status = 'Submitted'
     ├─ Requirement::update()
     └─ Redirect
          ↓
[POST] Verify Requirement
     ├─ Set verified_by (current user)
     ├─ Update verification_date
     ├─ Status = 'Verified'
     ├─ Add verification notes
     ├─ Requirement::update()
     └─ Redirect
```

---

## 🔑 KEY ROLE-BASED ACCESS CONTROL

```
RECEPTIONIST Can Access:
├─ Dashboard (Receptionist view)
├─ Patient Management (Create, Read, Update, Delete)
├─ Appointment/Transaction Management (Create, Read, Update, Cancel)
├─ Follow-up Management (Create, Read, Update, Delete)
├─ Doctor Management (Create, Read, Update, Delete)
├─ Doctor Schedule Management (Create, Read, Update, Delete)
├─ Receptionist Management (Create, Read, Update, Delete)
├─ Billing Management (Create, Read, Update, Delete)
├─ Data Import/Export
├─ Requirements Management (Create, Submit)
└─ Financial Reports

DOCTOR Can Access:
├─ Dashboard (Doctor view)
├─ Patient Records (View only, filtered to doctor's patients)
├─ Transaction Management (View, Complete, Add Notes & Prescriptions)
├─ Follow-up Management (View own follow-ups)
├─ Doctor Profile (View/Edit own profile only)
├─ Doctor Schedule (View own schedule)
└─ Requirements Management (View, Verify)

UNAUTHENTICATED Users Can:
└─ Access Login Page Only
```

---

## ⚠️ VALIDATION & ERROR HANDLING FLOW

```
User Input
     ↓
Form Submission
     ↓
Receive POST/GET Data
     ↓
Initial Data Sanitization
     ├─ trim()
     ├─ htmlspecialchars() (for display)
     └─ Type casting
          ↓
Business Logic Validation
     ├─ Required field checks
     ├─ Data format validation
     ├─ Relationship checks (FK exists)
     ├─ Authorization checks
     └─ Business rule validation
          ↓
     ┌─────────────────┬──────────────────┐
     │                 │                  │
     ↓ VALIDATION OK   ↓ VALIDATION FAILS │
                       │                  │
Database Operation    Store Error        │
     ↓                Message             │
Update/Insert/Delete  in $error var      │
     ↓                │                  │
Check Success        Re-display Form     │
     ↓                with error         │
setFlash()            and user input     │
redirect()                               │
     ↓                                   ↓
Success Response     Error Response
```

---

## 🔄 SESSION & STATE MANAGEMENT

```
User Session Lifecycle
     ↓
[login.php] - Session Created
     ├─ $_SESSION['user_id']
     ├─ $_SESSION['name']
     ├─ $_SESSION['role']
     ├─ $_SESSION['email']
     └─ Session ID cookie
          ↓
Session Active
     ├─ Persistent across page requests
     ├─ Used for authentication checks
     ├─ Used for role-based access
     └─ Available to all pages
          ↓
[logout.php] - Session Destroyed
     ├─ session_destroy()
     ├─ All session vars cleared
     ├─ Session cookie expires
     └─ Redirect to login
          ↓
Flash Messages (Temporary)
     ├─ Set by setFlash(type, msg)
     ├─ Type: success, danger, warning, info
     ├─ Retrieved by getFlash()
     ├─ Displayed once then deleted
     └─ Used for feedback messages
```

---

## 📊 DATA FLOW OVERVIEW

```
                          ┌─────────────┐
                          │   MySQL DB  │
                          └──────┬──────┘
                                 │
                 ┌───────────────┼───────────────┐
                 │               │               │
         ┌──────▼────────┐ ┌────▼────────┐ ┌───▼──────────┐
         │ Users/Doctors │ │ Patients    │ │ Transactions │
         └──────┬────────┘ └────┬────────┘ └───┬──────────┘
                │               │              │
         ┌──────▼────────────────┼──────────────▼────┐
         │   ClinicEase Classes  │                   │
         ├───────────────────────┼─────────────────┤
         │ • User                │                   │
         │ • Patient             │                   │
         │ • Transaction         │                   │
         │ • Doctor              │                   │
         │ • FollowUp            │                   │
         │ • Billing             │                   │
         │ • Requirements        │                   │
         │ • Schedule            │                   │
         │ • Database (Singleton)│                   │
         └──────┬────────────────┼─────────────────┘
                │                │
         ┌──────▼──────────────────▼────┐
         │   PHP Pages/Views             │
         ├───────────────────────────────┤
         │ • Authentication              │
         │ • Dashboards                  │
         │ • Forms & CRUD               │
         │ • Data Management            │
         │ • Reporting                  │
         └───────────┬───────────────────┘
                     │
             ┌───────▼──────────┐
             │   Browser (User) │
             └──────────────────┘
```

---

## 🎯 COMPLETE USER JOURNEY EXAMPLES

### Journey 1: RECEPTIONIST SCHEDULING A PATIENT APPOINTMENT

```
Receptionist Login
    ↓
View Dashboard (Upcoming Visits, Follow-ups)
    ↓
Navigate to "Patients" Menu
    ↓
View or Create Patient Record
    ↓
Navigate to "Appointments/Transactions"
    ↓
Click "Schedule Appointment"
    ↓
Form: Select Patient, Doctor, Date, Time, Type, Reason
    ↓
Validate form & check doctor availability
    ↓
Create transaction record
    ↓
setFlash('success') + Redirect
    ↓
View updated appointment list
```

### Journey 2: DOCTOR COMPLETING A PATIENT CONSULTATION

```
Doctor Login
    ↓
View Dashboard (Upcoming Visits for this Doctor)
    ↓
Click on scheduled appointment
    ↓
View Patient Details & Appointment Info
    ↓
Click "Complete Consultation"
    ↓
Form: Doctor Notes, Prescription, Optional Requirements
    ↓
Validate and save
    ↓
Update transaction status to 'Completed'
    ↓
Redirect to dashboard
    ↓
View updated visit status
    ↓
[Optional] Later: Receptionist bills the completed visit
```

### Journey 3: RECEPTIONIST CREATING & BILLING A CONSULTATION

```
Receptionist Login
    ↓
View Dashboard
    ↓
Appointment scheduling → Doctor completes → Transaction marked as 'Completed'
    ↓
Navigate to "Billing"
    ↓
View "Awaiting Billing" section showing completed visit
    ↓
Click "Create Bill"
    ↓
Select unbilled items (Consultation + any follow-ups)
    ↓
System calculates total
    ↓
Save Bill
    ↓
View billing list → Click bill detail
    ↓
[Optional] Mark as paid, print, or export
```

---

## 🔧 ERROR HANDLING PATHS

```
Authentication Error
    ├─ Missing user_id in session
    ├─ Invalid role
    └─ Unauthorized access
         ↓
    Redirect to login.php or dashboard
    Display appropriate error

Database Error
    ├─ Connection failure
    ├─ Query error
    └─ Constraint violation
         ↓
    Catch PDOException
    Display user-friendly error
    Log technical details

Validation Error
    ├─ Missing required field
    ├─ Invalid data type
    ├─ Business rule violation
    └─ Foreign key doesn't exist
         ↓
    Store error in $error variable
    Re-render form with error message
    Preserve user input for correction
```

---

## 📁 FILE STRUCTURE REFERENCE

```
clinicease/
├── index.php                 [Entry point & session check]
├── login.php                 [Authentication]
├── logout.php                [Destroy session]
├── config.php                [Global config & helpers]
├── classes/
│   ├── Database.php         [Singleton DB connection]
│   ├── User.php             [User login/logout]
│   ├── Patient.php          [Patient CRUD]
│   ├── Doctor.php           [Doctor management]
│   ├── Transaction.php      [Appointments/Visits]
│   ├── FollowUp.php         [Follow-up management]
│   ├── Billing.php          [Billing management]
│   └── Schedule.php         [Doctor schedules]
├── pages/
│   ├── dashboard.php        [Main dashboard]
│   ├── partials/sidebar.php [Navigation menu]
│   ├── patients/
│   │   ├── index.php        [Patient list]
│   │   └── view.php         [Patient detail]
│   ├── doctors/
│   │   ├── list.php         [Doctor list]
│   │   └── edit.php         [Doctor edit]
│   ├── billing/
│   │   ├── index.php        [Bills list]
│   │   ├── create.php       [Create bill]
│   │   └── view.php         [Bill detail]
│   ├── xml/
│   │   ├── import.php       [Import data]
│   │   └── export.php       [Export data]
│   ├── receptionists/
│   │   ├── list.php         [Receptionist list]
│   │   └── edit.php         [Edit receptionist]
│   └── transactions/        [Appointment management]
├── assets/css/style.css     [Styling]
└── sql/clinicease.sql       [Database schema]
```

---

**Created:** June 4, 2026
**System:** ClinicEase v1.0
**Last Updated:** Current
