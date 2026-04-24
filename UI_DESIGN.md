# UI Design Specification

## Overview

This document contains detailed UI/UX designs for the Dominion School Management System. All designs follow **Ant Design** component library principles for a clean, professional, and consistent look.

---

## Design System

### Color Palette

| Color | Hex Code | Usage |
|-------|----------|-------|
| Primary Blue | `#1677ff` | Buttons, links, active states |
| Success Green | `#52c41a` | Paid status, positive values |
| Warning Orange | `#faad14` | Partial payment, pending status |
| Error Red | `#ff4d4f` | Debt, overdue, void status |
| Neutral Gray | `#f5f5f5` | Backgrounds, borders |
| Text Dark | `#262626` | Primary text |
| Text Light | `#8c8c8c` | Secondary text, placeholders |

### Typography

| Element | Font Size | Weight | Example |
|---------|-----------|--------|--------|
| H1 (Page Title) | 24px | Bold | "Dashboard" |
| H2 (Section) | 20px | Bold | "Recent Payments" |
| H3 (Card Title) | 16px | Bold | "Total Students" |
| Body | 14px | Regular | Standard text |
| Small | 12px | Regular | Secondary info, timestamps |
| Large Numbers | 28px | Bold | Dashboard stats |

### Spacing

| Element | Value |
|---------|-------|
| Card Padding | 24px |
| Form Item Gap | 16px |
| Table Row Height | 54px |
| Modal Max Width | 720px |
| Page Margin | 24px |
| Grid Gutter | 16px |

---

## Global Layout

### Structure

```
+------------------------------------------------------------------+
|  SIDEBAR (256px)           |  TOPBAR (64px)                      |
|  +-------------------+     |  +--------------------------------+ |
|  | [LOGO]            |     |  | [Search]  [Sync Status] [Notif] [User] |
|  | Dominion School   |     |  +--------------------------------+ |
|  +-------------------+     +------------------------------------+
|  | Dashboard         |     |  MAIN CONTENT AREA                  |
|  | Students          |     |  +--------------------------------+ |
|  | Payments          |     |  |                                | |
|  | Reports           |     |  |  Page-specific content         | |
|  | Inventory         |     |  |                                | |
|  | Settings          |     |  +--------------------------------+ |
|  +-------------------+     +------------------------------------+
```

### Sidebar Navigation

```
+-------------------+
| DOMINION SCHOOL   |
+-------------------+
| Dashboard         |
| Students          |
| Payments          |
| Reports           |
| Inventory         |
| Settings          |
+-------------------+
```

**Behavior**:
- Collapses to icons only on tablet (768px)
- Hamburger menu on mobile (< 768px)
- Active item highlighted with blue background
- **Sync Status**: Visual indicator (Cloud icon) in topbar showing if local data is synced to server.

---

## 1. Dashboard

### Purpose
Quick overview for daily operations - big numbers, actionable alerts.

### ASCII Wireframe

```
+------------------------------------------------------------------+
|  DASHBOARD                                                        |
+------------------------------------------------------------------+
|                                                                   |
|  +----------------+  +----------------+  +----------------+       |
|  | Total          |  | Today''s         |  | Outstanding    |       |
|  | Students       |  | Collection     |  | Debt           |       |
|  |                |  |                |  |                |       |
|  |    350         |  |  1,200,000 TZS |  |    800,000     |       |
|  |   +12 this     |  |   +45 payments |  |   28 students  |       |
|  |   month        |  |   today        |  |   owing        |       |
|  +----------------+  +----------------+  +----------------+       |
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | Collection Trends (Last 30 Days)                          |    |
|  |                                                           |    |
|  |      |           /                                       |    |
|  |      |         /                                         |    |
|  |      |       /                                           |    |
|  |      |     /                                             |    |
|  |      |___/                                               |    |
|  |         1    10    20    30 (Days)                        |    |
|  +-----------------------------------------------------------+    |
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | Recent Payments                        [Export] [View All] |    |
|  +-----------------------------------------------------------+    |
|  | Student Name        | Amount      | Method | Date          |    |
|  |---------------------|-------------|--------|---------------|    |
|  | Amaira Mapalala     | 150,000 TZS | Cash   | 14 Apr 2026   |    |
|  | Daniel Mjema        | 236,500 TZS | Mobile | 14 Apr 2026   |    |
|  +-----------------------------------------------------------+    |
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | Alerts                                [Dismiss All]         |    |
|  +-----------------------------------------------------------+    |
|  | 15 students with debt > 100,000 TZS      [View]             |    |
|  | 8 payments pending verification          [Review]            |    |
|  +-----------------------------------------------------------+    |
+------------------------------------------------------------------+
```

### Components

| Section | Ant Design Component |
|---------|---------------------|
| Stat Cards | `Card` + `Statistic` |
| Chart | `LineChart` (Recharts) |
| Payments Table | `Table` (sortable, paginated) |
| Alerts | `Alert` with type (error/warning/info) |

### Interactions

- Click stat card → Navigate to filtered list
- Click table row → View payment details
- Click alert → Navigate to relevant page
- Hover chart → Show tooltip with exact values

---

## 2. Student List Page

### Purpose
Fast search, filtering, and access to student profiles.

### ASCII Wireframe

```
+------------------------------------------------------------------+
|  STUDENTS                                          [Add Student]  |
+------------------------------------------------------------------+
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | Search by name, admission no, ID pass no...               |    |
|  |                                                           |    |
|  |  Filter: [Class v]  [Status v]  [Term v]  [Reset]         |    |
|  +-----------------------------------------------------------+    |
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | [ ] | Adm No | Name          | Class | Balance   | Status |    |
|  |-----|--------|---------------|-------|-----------|--------|    |
|  | [ ] | 001    | Amaira M.     | Baby  | 350,000   | Orange |    |
|  | [ ] | 002    | Daniel M.     | Baby  | 0         | Green  |    |
|  | [ ] | 003    | Teddy P.      | Baby  | 0         | Green  |    |
|  | [ ] | 004    | John D.       | Mid   | 50,000    | Red    |    |
|  +-----------------------------------------------------------+    |
|                                                                   |
|  Showing 1-20 of 350    < 1 2 3 4 5 ... 18 >                     |
|                                                                   |
|  [Select All] [Export Selected] [Bulk Payment] [Bulk SMS/Remind]  |
+------------------------------------------------------------------+
```

### Components

| Element | Ant Design Component |
|---------|---------------------|
| Search Bar | `Input.Search` with enter button |
| Filters | `Select` (multiple) |
| Table | `Table` with row selection |
| Pagination | `Pagination` (show size changer) |
| Status Badge | `Badge` (green/orange/red) |

### Status Colors

| Status | Color | Meaning |
|--------|-------|--------|
| Green | `#52c41a` | Fully paid (balance = 0) |
| Orange | `#faad14` | Partial payment (0 < balance < 50%) |
| Red | `#ff4d4f` | High debt (balance >= 50%) |

### Interactions

- **Search:** Debounced (300ms), triggers on Enter
- **Filters:** Auto-apply on change, "Reset" clears all
- **Row Click:** Navigate to Student Profile
- **Checkbox Select:** Enable bulk actions
- **Sort:** Click column header to sort (asc/desc)

---

## 3. Student Profile (CORE PAGE)

### Purpose
Complete student view - all information, all actions in one place.

### ASCII Wireframe

```
+------------------------------------------------------------------+
|  STUDENT PROFILE                                  [Edit] [Print]  |
+------------------------------------------------------------------+
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | [Photo]  AMAIRA RAMADHAN MAPALALA                    [Edit]|
|  |          Admission: ADM-2026-001  |  ID Pass: 411          |
|  |          Class: Baby Class        |  Gender: Female        |
|  |          Guardian: Ramadhan M.    |  Phone: +255 XXX XXX   |
|  |          Status: Active           |  Joined: Jan 2026      |
|  +-----------------------------------------------------------+    |
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | FEE SUMMARY                                                 |   |
|  |                                                             |   |
|  |    Total Fee:      665,000 TZS                              |   |
|  |    Paid:           315,000 TZS                              |   |
|  |    Balance:        350,000 TZS  [Red]                       |   |
|  |                                                             |   |
|  |    [======== 47% Paid ========]                             |   |
|  +-----------------------------------------------------------+    |
|                                                                   |
|  [ Record Payment ]  [ Print Statement ]  [ Promote ]            |
|                                                                   |
+------------------------------------------------------------------+
|  [Overview] [Payments] [Debt] [Transport] [Inventory] [Comments]|
+------------------------------------------------------------------+
```

### TAB 1: OVERVIEW

```
+------------------------------------------------------------------+
|  OVERVIEW                                                         |
+------------------------------------------------------------------+
|  +-------------------+  +-------------------+                     |
|  | Academic          |  | Medical           |                     |
|  | Term 1: B+        |  | Blood Type: O+    |                     |
|  | Attendance: 92%   |  | Allergies: None   |                     |
|  +-------------------+  +-------------------+                     |
|  +-------------------+  +-------------------+                     |
|  | Stationery        |  | Transport         |                     |
|  | Reams: 1 (Pend)   |  | Status: Active    |                     |
|  | Files: 1 (Paid)   |  | Rate: 35,000/mo   |                     |
|  +-------------------+  +-------------------+                     |
+------------------------------------------------------------------+
```

### TAB 2: PAYMENTS

```
+------------------------------------------------------------------+
|  PAYMENTS                                   [Add Payment]         |
+------------------------------------------------------------------+
|  Filter: [Term v]  [Date Range v]  [Method v]  [Export]          |
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | Date     | Amount    | Method | Term  | Receipt | Actions |    |
|  |----------|-----------|--------|-------|---------|---------|    |
|  | 08 Apr   | 150,000   | Cash   | 2nd   | RCP-411 | View    |    |
|  | 15 Jan   | 165,000   | Mobile | 1st   | RCP-205 | View    |    |
|  +-----------------------------------------------------------+    |
+------------------------------------------------------------------+
```

### TAB 3: DEBT

```
+------------------------------------------------------------------+
|  DEBT / ARREARS (DENI)                                            |
+------------------------------------------------------------------+
|  +-----------------------------------------------------------+    |
|  | Term     | Amount Owed | Due Date   | Status   | Actions  |    |
|  |----------|-------------|------------|----------|----------|    |
|  | 2nd 2026 | 350,000 TZS | 05 Jun 26  | Overdue  | [Pay]    |    |
|  | 1st 2026 | 0           | 31 Jan 26  | Paid     | [History]|    |
|  +-----------------------------------------------------------+    |
|  Total Outstanding: 350,000 TZS                                   |
|  [ Payment Plan ]  [ Request Extension ]                          |
+------------------------------------------------------------------+
```

### TAB 4: TRANSPORT

```
+------------------------------------------------------------------+
|  TRANSPORT SERVICE                                  [Edit]        |
+------------------------------------------------------------------+
|  Status: Active                               [Start/Stop]        |
|                                                                   |
|  Monthly Rate:    35,000 TZS                                      |
|  Vehicle:         Bus A-12                                        |
|  Route:           City Center - School                            |
|  Next Billing:    01 May 2026                                     |
+------------------------------------------------------------------+
```

### TAB 5: INVENTORY

```
+------------------------------------------------------------------+
|  INVENTORY / STATIONERY                           [Add Item]      |
+------------------------------------------------------------------+
|  +-----------------------------------------------------------+    |
|  | Item    | Qty | Cost       | Status       | Actions        |    |
|  |---------|-----|------------|--------------|----------------|    |
|  | Ream    | 1   | 10,000 TZS | [pkg] Pend   | [Fulfill]      |    |
|  | File    | 1   | 3,500 TZS  | [chk] Paid   | [View]         |    |
|  | Uniform | 1   | 25,000 TZS | [user] Issued| [View]         |    |
|  +-----------------------------------------------------------+    |
+------------------------------------------------------------------+
```

### TAB 6: COMMENTS

```
+------------------------------------------------------------------+
|  COMMENTS                                     [Add Comment]       |
+------------------------------------------------------------------+
|  +-----------------------------------------------------------+    |
|  | Accounts Officer                         14 Apr 2026        |    |
|  | Parent requested payment extension until end of May.       |    |
|  | Approved by Madam.                                          |    |
|  +-----------------------------------------------------------+    |
|  +-----------------------------------------------------------+    |
|  | Director (Madam)                         10 Apr 2026        |    |
|  | In talks with parent regarding transport fee adjustment.   |    |
|  +-----------------------------------------------------------+    |
+------------------------------------------------------------------+
```

---

## 4. Payment Entry UI (CRITICAL)

### Purpose
Fast, error-free payment recording - the most used feature.

### ASCII Wireframe (Modal - 720px wide)

```
+------------------------------------------------------------------+
|  RECORD PAYMENT                                           [X]     |
+------------------------------------------------------------------+
|                                                                   |
|  Student: [ Search Amaira Mapalala (ADM-2026-001)       (X) ]    |
|           (Auto-complete search)                                  |
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | Current Balance: 350,000 TZS [Red]                        |    |
|  +-----------------------------------------------------------+    |
|                                                                   |
|  Amount: [ 150,000                  ] TZS                         |
|          (Min: 1,000 | Max: 10,000,000)                          |
|                                                                   |
|  Payment Method:                                                  |
|  (X) Cash   ( ) Mobile Money   ( ) Bank Transfer                 |
|                                                                   |
|  Term: [ 2nd Term 2026 v ]                                        |
|                                                                   |
|  Date: [ 14/04/2026 ]  <- Auto-filled (editable)                 |
|                                                                   |
|  Reference/Note: [___________________________] (Optional)        |
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | ALLOCATION PREVIEW                                          |    |
|  |                                                             |    |
|  | Debt (1st Term):      100,000 TZS  <- Will be cleared       |    |
|  | Tuition (2nd Term):    50,000 TZS  <- Remaining             |    |
|  | Transport:                  0 TZS                           |    |
|  | -------------------------------------                       |    |
|  | Total:                150,000 TZS                           |    |
|  |                                                             |    |
|  | New Balance: 200,000 TZS [Orange]                           |    |
|  +-----------------------------------------------------------+    |
|                                                                   |
|  [x] Print receipt after payment                                 |
|                                                                   |
|  [ Cancel ]                    [ Submit Payment ]                 |
|                                                                   |
+------------------------------------------------------------------+
```

### Components

| Element | Ant Design Component |
|---------|---------------------|
| Modal | `Modal` (centered, closable) |
| Student Search | `Select` with showSearch |
| Amount Input | `InputNumber` with formatter |
| Payment Method | `Radio.Group` |
| Term Select | `Select` |
| Date Picker | `DatePicker` |
| Allocation Preview | `Card` + `Statistic` |

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Enter` | Submit payment |
| `Esc` | Close modal |
| `Ctrl+P` | Toggle print receipt |

### Smart Features

1. **Auto-allocation:** System suggests best allocation (Debt → Tuition → Transport)
2. **Balance preview:** Shows new balance before submitting
3. **Recent students:** Quick-select from last 5 students paid
4. **Validation:** Real-time inline validation
5. **Offline mode:** Queue payment if offline, sync when online
6. **Keyboard-Driven Entry:** Logical `tabIndex` order ensures full keyboard-only recording (Search → Amount → Submit).

---

## 5. Receipt Print View

### Purpose
Clean, professional receipt for parents.

### ASCII Wireframe (A4 Paper Format)

```
+----------------------------------------------------------+
|                                                          |
|     DOMINION PRE & PRIMARY SCHOOL                        |
|     P.O. Box XXX, City, Tanzania                         |
|     Phone: +255 XXX XXX XXX                              |
|                                                          |
|     ==================================================   |
|                                                          |
|               PAYMENT RECEIPT                            |
|                                                          |
|     Receipt No: RCP-2026-411                             |
|     Date: 14 April 2026                                  |
|                                                          |
|     --------------------------------------------------   |
|                                                          |
|     Received From: Ramadhan Mapalala                     |
|                                                          |
|     Student: Amaira Mapalala (ADM-2026-001)              |
|                                                          |
|     Amount: ONE HUNDRED FIFTY THOUSAND ONLY              |
|             (150,000 TZS)                                |
|                                                          |
|     For: 2nd Term 2026 Fees                              |
|                                                          |
|     Payment Method: Cash                                 |
|                                                          |
|     --------------------------------------------------   |
|                                                          |
|     Allocation:                                          |
|     - Debt (1st Term):     100,000 TZS                   |
|     - Tuition (2nd Term):   50,000 TZS                   |
|                                                          |
|     --------------------------------------------------   |
|                                                          |
|     Remaining Balance: 200,000 TZS                       |
|                                                          |
|     _________________________                            |
|     Authorized Signature                                 |
|                                                          |
|     ==================================================   |
|     Thank you for your payment!                          |
+----------------------------------------------------------+
```

### Features

- School logo at top
- Receipt number unique per academic year
- Amount in words + numbers
- Allocation breakdown shown
- Print-optimized (no colors, clean layout)
- **Print Context Toggle:** Quick switch between 'Copy for School' and 'Copy for Parent' versions.

---

## 6. Reports Page

### Purpose
Generate and export financial reports.

### ASCII Wireframe

```
+------------------------------------------------------------------+
|  REPORTS                                                          |
+------------------------------------------------------------------+
|  Report Type: [ Daily Collections v ]                             |
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | FILTERS                                                     |    |
|  | Date Range: [ 01/04/2026 ] to [ 14/04/2026 ]               |    |
|  | Class: [ All Classes v ]                                    |    |
|  | Payment Method: [ All Methods v ]                           |    |
|  | [ Apply Filters ]  [ Reset ]                                |    |
|  +-----------------------------------------------------------+    |
|                                                                   |
|  [ Generate Report ]  [ Export PDF ]  [ Export Excel ]            |
|                                                                   |
|  +-----------------------------------------------------------+    |
|  | RESULTS (Preview)                                           |    |
|  +-----------------------------------------------------------+    |
|  | Date     | Student    | Amount    | Method | User          |    |
|  |----------|------------|-----------|--------|---------------|    |
|  | 14 Apr   | Amaira M.  | 150,000   | Cash   | John D.       |    |
|  | 14 Apr   | Daniel M.  | 236,500   | Mobile | John D.       |    |
|  +-----------------------------------------------------------+    |
|  Total: 1,200,000 TZS | Payments: 45                              |
+------------------------------------------------------------------+
```

### Available Reports

| Report | Filters | Output |
|--------|---------|--------|
| Daily Collections | Date range, method, user | PDF, Excel |
| Outstanding Balances | Class, term, balance range | PDF, Excel |
| Transport Revenue | Term, vehicle, route | PDF, Excel |
| Debtors List | Due date, class, amount | PDF, Excel |
| Student Ledger | Student, term | PDF (printable) |
| Family Summary | Family name | PDF, Excel |
| Audit Log | User, action type, date | PDF, Excel |

---

## 7. Settings Page

### Purpose
Configure system parameters.

### ASCII Wireframe

```
+------------------------------------------------------------------+
|  SETTINGS                                                         |
+------------------------------------------------------------------+
|  [ Fee Structures ] [ Transport Rates ] [ Academic Terms ] [Users]|
+------------------------------------------------------------------+
|  FEE STRUCTURES                                                   |
+------------------------------------------------------------------+
|  +-----------------------------------------------------------+    |
|  | Class      | Annual Fee | Term Fee | Actions               |    |
|  |------------|------------|----------|----------------------|    |
|  | Baby Class | 350,000    | 150,000  | [Edit] [Delete]      |    |
|  | Middle     | 500,000    | 200,000  | [Edit] [Delete]      |    |
|  +-----------------------------------------------------------+    |
|  [ Add Fee Structure ]                                            |
+------------------------------------------------------------------+
|  ACADEMIC TERMS                                                   |
+------------------------------------------------------------------+
|  +-----------------------------------------------------------+    |
|  | Year | Term | Start Date | End Date   | Status             |    |
|  |------|------|------------|------------|--------------------|    |
|  | 2026 | 1st  | 08 Jan 26  | 28 Mar 26  | Completed          |    |
|  | 2026 | 2nd  | 08 Apr 26  | 05 Jun 26  | Active             |    |
|  | 2026 | 3rd  | 01 Jul 26  | 30 Sep 26  | Upcoming           |    |
|  +-----------------------------------------------------------+    |
+------------------------------------------------------------------+
```

---

## 8. Void Payment Modal

### Purpose
Cancel a payment with audit trail (cannot delete).

### ASCII Wireframe

```
+------------------------------------------------------------------+
|  VOID PAYMENT                                           [X]       |
+------------------------------------------------------------------+
|  This action cannot be undone.                                    |
|                                                                   |
|  Payment Details:                                                 |
|  +-----------------------------------------------------------+    |
|  | Receipt No: RCP-2026-411                                   |    |
|  | Student: Amaira Mapalala                                   |    |
|  | Amount: 150,000 TZS                                        |    |
|  | Date: 14 Apr 2026                                          |    |
|  +-----------------------------------------------------------+    |
|                                                                   |
|  Reason for Voiding: * Required                                  |
|  +-----------------------------------------------------------+    |
|  |                                                           |    |
|  | [_______________________________________________________] |    |
|  |                                                           |    |
|  +-----------------------------------------------------------+    |
|                                                                   |
|  Warning: Director approval required for amounts > 100,000 TZS    |
|                                                                   |
|  [ Cancel ]              [ Confirm Void ]                         |
|                                                                   |
+------------------------------------------------------------------+
```
