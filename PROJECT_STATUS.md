# Project Status: Dominion School Management System

## Overview
A comprehensive management and accounting system for Dominion Pre & Primary School to replace manual document-based tracking.

## Objectives
- Digitalize student records and fee tracking.
- Automate balance calculations and debt management.
- Manage transport fees and stationery inventory.
- Generate financial reports and receipts.
- Streamline parent communication and payment reminders.
- Support multi-term, multi-year academic tracking.

## Architecture
- **Frontend:** React + TypeScript (Vite)
- **UI Library:** Ant Design
- **Backend:** Node.js + Express
- **Database:** MySQL 8.0 (Dockerized)
- **ORM:** Prisma (Type-safe)
- **Containerization:** Docker + Docker Compose
- **Web Server:** Nginx (serving Frontend as SPA)
- **Authentication:** JWT-based
- **Deployment:** Containerized (Multi-stage builds)

---

## Step 0: DevOps & Environment Setup (COMPLETED)

### Dockerization Status
- [ ] **Database Container**: MySQL 8.0 with persistent volume (`mysql_data`).
- [ ] **Backend Container**: Node 20-alpine, multi-stage build, auto-migration on start.
- [ ] **Frontend Container**: Node 20-alpine build stage + Nginx serving stage.
- [ ] **Orchestration**: Docker Compose managing all 4 services (DB, Backend, Frontend, phpMyAdmin).
- [ ] **Networking**: Internal Docker network for secure DB-to-Backend communication.
- [ ] **SPA Routing**: Nginx configured to handle React Router deep-links (`try_files`).

### Local Development Commands
- `docker-compose up --build`: Build and start the entire stack.
- `docker-compose down`: Stop all services.
- `docker-compose logs -f [service]`: View real-time logs for a specific service.

---

## Step 1: Database Schema Design (MySQL)

### Core Entities
| Model | Description | Key Fields (MySQL) |
|-------|-------------|-------------------|
| `academic_years` | School year definition | id (BIGINT AUTO_INCREMENT), year (e.g., 2026), start_date, end_date, is_active (TINYINT) |
| `academic_terms` | Term within a year | id (BIGINT AUTO_INCREMENT), academic_year_id (BIGINT), name (1st/2nd/3rd), start_date, end_date, is_active (TINYINT) |
| `classes` | Student class/grade | id (BIGINT AUTO_INCREMENT), name (Baby, Middle, Pre-Unit, etc.), term_fee (INT), annual_fee (INT) |
| `students` | Pupil record | id (BIGINT AUTO_INCREMENT), admission_no (VARCHAR 50 UNIQUE), id_pass_no (VARCHAR 50), name (VARCHAR 100), gender (ENUM("M","F")), date_of_birth (DATE), class_id (BIGINT), guardian_id (BIGINT), family_id (BIGINT), status (ENUM("active","promoted","withdrawn")), photo_path (VARCHAR 255), created_at (TIMESTAMP) |
| `guardians` | Parent/guardian info | id (BIGINT AUTO_INCREMENT), name (VARCHAR 100), phone (VARCHAR 20), email (VARCHAR 100), address (VARCHAR 255), alternate_contact (VARCHAR 20), family_id (BIGINT) |
| `families` | Sibling grouping | id (BIGINT AUTO_INCREMENT), family_name (VARCHAR 100), discount_amount (INT), discount_type (ENUM("fixed","percentage")) |
| `transport` | Transport service | id (BIGINT AUTO_INCREMENT), student_id (BIGINT), monthly_rate (INT), start_term_id (BIGINT), end_term_id (BIGINT), status (ENUM("active","inactive")), vehicle_details (VARCHAR 255) |
| `fee_structures` | Fee definitions | id (BIGINT AUTO_INCREMENT), class_id (BIGINT), academic_term_id (BIGINT), fee_type (ENUM("tuition","transport","stationery","id")), amount (INT) |
| `student_fees` | Student-specific fees | id (BIGINT AUTO_INCREMENT), student_id (BIGINT), academic_term_id (BIGINT), total_amount (INT), discount_amount (INT), net_amount (INT), is_sponsored (TINYINT), sponsor_name (VARCHAR 100) |
| `payments` | Payment records | id (BIGINT AUTO_INCREMENT), student_id (BIGINT), amount (INT), payment_date (DATE), payment_method (ENUM("cash","mobile","bank")), receipt_no (VARCHAR 50), term_id (BIGINT), recorded_by (BIGINT), notes (TEXT), status (ENUM("active","voided")), void_reason (TEXT), created_at (TIMESTAMP) |
| `payment_allocations` | Payment distribution | id (BIGINT AUTO_INCREMENT), payment_id (BIGINT), fee_type (VARCHAR 50), amount_allocated (INT), term_id (BIGINT) |
| `inventory_items` | School supplies | id (BIGINT AUTO_INCREMENT), name (Ream, File, Uniform, Kofia), unit_cost (INT), stock_quantity (INT), reorder_level (INT) |
| `student_inventory` | Items assigned to students | id (BIGINT AUTO_INCREMENT), student_id (BIGINT), item_id (BIGINT), quantity (INT), term_id (BIGINT), is_paid (TINYINT), status (ENUM("pending","fulfilled")) |
| `receipts` | Official receipt | id (BIGINT AUTO_INCREMENT), receipt_no (VARCHAR 50 UNIQUE), student_id (BIGINT), payment_id (BIGINT), issued_date (DATE), issued_by (BIGINT), pdf_path (VARCHAR 255) |
| `debt_records` | Arrears tracking | id (BIGINT AUTO_INCREMENT), student_id (BIGINT), term_id (BIGINT), amount_owed (INT), due_date (DATE), status (ENUM("pending","partial","paid")), notes (TEXT) |
| `promotions` | Class promotion history | id (BIGINT AUTO_INCREMENT), student_id (BIGINT), from_class_id (BIGINT), to_class_id (BIGINT), promotion_date (DATE), academic_year_id (BIGINT) |
| `users` | System users | id (BIGINT AUTO_INCREMENT), name (VARCHAR 100), email (VARCHAR 100 UNIQUE), role (ENUM("admin","accounts","director","reception")), password_hash (VARCHAR 255), created_at (TIMESTAMP), last_login (TIMESTAMP) |
| `comments` | Notes on student accounts | id (BIGINT AUTO_INCREMENT), student_id (BIGINT), comment_text (TEXT), created_by (BIGINT), created_at (TIMESTAMP) |
| `system_logs` | Audit trail | id (BIGINT AUTO_INCREMENT), user_id (BIGINT), action (VARCHAR 100), entity_type (VARCHAR 50), entity_id (BIGINT), timestamp (TIMESTAMP), details (JSON) |
| `notifications` | SMS/Email logs | id (BIGINT AUTO_INCREMENT), student_id (BIGINT), guardian_id (BIGINT), message (TEXT), channel (ENUM("sms","email")), status (ENUM("pending","sent","failed")), sent_at (TIMESTAMP) |

### Entity Relationships
- guardians → students (one-to-many via guardian_id)
- families → guardians (one-to-many via family_id)
- classes → students (one-to-many via class_id)
- students → transport (one-to-one active per term via student_id)
- students → payments (one-to-many via student_id)
- students → debt_records (one-to-many via student_id)
- students → student_inventory (one-to-many via student_id)
- students → comments (one-to-many via student_id)
- academic_years → academic_terms (one-to-many via academic_year_id)
- users → payments (one-to-many via recorded_by)
- users → system_logs (one-to-many via user_id)
- payments → payment_allocations (one-to-many via payment_id)

### MySQL-Specific Notes
- Use `BIGINT AUTO_INCREMENT` for all primary keys (simpler than UUID)
- Use `JSON` type for details field in system_logs (not JSONB)
- Use `ENUM` for status fields (MySQL native support)
- Use `TINYINT(1)` for boolean fields (is_active, is_sponsored, is_paid)
- Use `TIMESTAMP DEFAULT CURRENT_TIMESTAMP` for created_at fields
- Add indexes on: admission_no, id_pass_no, student_id, term_id, payment_date
- Foreign keys with `ON DELETE RESTRICT` to prevent accidental deletions

---

## Step 2: Features by Development Phase

### Phase 1: Core Foundation (MVP) - Weeks 1-4 (COMPLETED)
**Goal: Launch early, test with real users**
- [ ] Student registration with unique admission number generation
- [ ] Class management (Baby, Middle, Pre-Unit, etc.)
- [ ] Guardian/Parent information capture
- [ ] Fee structure setup per class/term
- [ ] Payment entry (FAST, minimal clicks) - Backend infrastructure ready
- [ ] Balance calculation (auto: total - paid = balance) - Backend logic ready
- [ ] Student search by name, ID, or class - Frontend implemented
- [ ] Basic dashboard (total students, collections today, outstanding) - Frontend/Backend integrated

### Phase 2: Financial Management - Weeks 5-8 (IN PROGRESS)
- [ ] Transport fee management (variable monthly rates)
- [ ] Family/sibling discount automation
- [ ] Debt/Arrears ("DENI") tracking with due dates
- [ ] Installment payment support
- [ ] Sponsored student flagging
- [ ] Stationery/Inventory tracking (Reams, Files, Uniforms, Kofia) - Other fees
- [ ] Receipt generation and reprint (PDF)
- [ ] Payment history per student (ledger view)
- [ ] Payment allocation engine (split payments across fee types)

### Phase 3: Advanced Operations - Weeks 9-11
- [ ] Student promotion workflow (end-of-term class changes)
- [ ] Multi-term and multi-year support
- [ ] New student onboarding workflow
- [ ] ID Pass management (issue, re-issue, track)
- [ ] Comment/Notes system for special arrangements ("MOM CALLED")
- [ ] Bulk payment import (for batch entries)
- [ ] Data export (Excel, PDF)
- [ ] Void/cancel payment with audit trail (requires reason)

### Phase 4: Reporting & Communication - Weeks 12-14
- [ ] Fee collection report (daily/weekly/monthly)
- [ ] Outstanding balances report (by class/student)
- [ ] Transport revenue report
- [ ] Debtors list with deadlines
- [ ] Family-wise billing summary
- [ ] Term comparison reports
- [ ] Student statement/ledger (printable PDF)
- [ ] SMS/WhatsApp payment reminders (integration)
- [ ] Email fee statements to parents

### Phase 5: Admin & Security - Weeks 15-16
- [ ] User role management (Admin, Accounts Officer, Director)
- [ ] Permission-based access control
- [ ] Audit trail (who recorded/modified payments)
- [ ] Data backup and restore (MySQL dump)
- [ ] System settings (fee structures, transport rates)
- [ ] Scheduled jobs (Node.js cron for transport billing)

### Phase 6: Parent Portal (Optional) - Weeks 17-20
- [ ] Parent login to view child fee status
- [ ] Online payment integration (mobile money, bank)
- [ ] Download fee statements
- [ ] Receive notifications (SMS/Email)

---

## Step 3: User Roles & Permissions

| Role | Permissions | Use Case |
|------|-------------|----------|
| **Admin** | Full access: all features, user management, system settings, data export/delete, audit log access | System administrator, IT staff |
| **Accounts Officer** | Record payments, view reports, search students, print receipts, add comments, void payments (with reason) | Daily fee collection, receipt issuance |
| **Director/Madam** | Approve discounts, view all reports, manage promotions, override payments, approve write-offs | Oversight, approvals, strategic decisions |
| **Reception/Secretary** | Search students, view payment status only (read-only), add basic comments | Front desk inquiries |
| **Parent** | View own children fee status, payment history, download statements (read-only) | Remote monitoring (Phase 6) |

---

## Step 4: UI/UX Design Guidelines

### Design Principles
- **Fast**: Data-heavy system must load quickly (paginated tables, lazy loading)
- **Clean**: Professional look with Ant Design components
- **Easy to use**: Minimal clicks for daily tasks (especially payment entry)

### Layout Structure
```
+-----------------------------------------+
|  Sidebar    |  Topbar (User + Stats)    |
|  (Nav)      +---------------------------+
|             |  Main Content Area        |
|  - Dashboard|  (Tables, Forms, Cards)   |
|  - Students |                           |
|  - Payments |                           |
|  - Reports  |                           |
|  - Settings |                           |
+-------------+---------------------------+
```

### Key Screens Design

#### 1. Student List Page
- **Table columns**: Name, Class, Balance, Status, Actions
- **Filters**: Class dropdown, Term dropdown, Search bar (name/ID)
- **Features**: Paginated (20-50 per page), sortable columns, quick view

#### 2. Student Profile Page (CORE SCREEN)
- **Tab structure**:
  - Overview (basic info, photo, guardian)
  - Payments (history table, add payment button)
  - Debt (outstanding balance, due dates)
  - Transport (status, rate, history)
  - Inventory (reams, files, uniforms - fulfilled/pending)
  - Comments (notes timeline)

#### 3. Payment Entry (CRITICAL - Used Daily)
- **Must be FAST** (target: < 30 seconds per transaction)
- **Minimal clicks** (auto-fill where possible)
- **Components**:
  - Student search (autocomplete)
  - Auto-display current balance
  - Amount input
  - Payment method dropdown
  - Allocation preview (how payment will be split)
  - Submit button
  - Print receipt option (immediate)

#### 4. Dashboard Widgets
- Total students (by class)
- Today collections (amount + count)
- Outstanding balance (total)
- Alerts (debts overdue, low inventory)
- Quick actions (Add Student, Record Payment)

### Ant Design Components to Use
| Component | Use Case |
|-----------|----------|
| Table | Student lists, payment history, reports |
| Form + Input | Data entry forms |
| Modal | Quick views, confirmations |
| Tabs | Student profile sections |
| Card | Dashboard widgets |
| Select | Dropdowns (class, term, method) |
| DatePicker | Date inputs |
| Badge | Status indicators (Paid/Unpaid) |
| Tag | Labels (NEW, PROMOTED, SPONSORED) |
| Menu | Sidebar navigation |
| Layout | Page structure |

---

## Step 5: Reports Specification

| Report Name | Filters | Output Format | Frequency | Primary User |
|-------------|---------|---------------|----------|-------------|
| Daily Collections | Date range, payment method, recorded_by | PDF, Excel | Daily | Accounts |
| Outstanding Balances | Class, term, student, balance range | PDF, Excel | Weekly | Director |
| Transport Revenue | Term, vehicle route, status | PDF, Excel | Monthly | Director |
| Debtors List | Due date, class, amount owed | PDF, Excel | Weekly | Accounts |
| Student Ledger | Student ID, term | PDF (printable) | On-demand | Accounts/Parent |
| Family Summary | Family name, term | PDF, Excel | On-demand | Accounts |
| Term Comparison | Term 1 vs Term 2 vs Term 3 | PDF, Excel | End-of-term | Director |
| Fee Collection by Class | Class, term, fee type | PDF, Excel | Monthly | Director |
| Inventory Usage | Item type, term, status | PDF, Excel | Monthly | Admin |
| Audit Log | User, date range, action type | PDF, Excel | On-demand | Admin/Director |
| New Admissions | Term, class, date range | PDF, Excel | Monthly | Director |
| Promotions Report | Academic year, from/to class | PDF, Excel | End-of-term | Director |
| Voided Transactions | User, date range, reason | PDF, Excel | On-demand | Director |

---

## Step 6: Validation Rules & Business Logic

### Input Validation
| Field | Validation Rule | Error Message |
|-------|----------------|---------------|
| Admission No | Unique, auto-generated format: ADM-YYYY-### | "Admission number must be unique" |
| ID Pass No | Unique per term, format: ### or ###/N/YY for re-issues | "ID Pass number already exists" |
| Phone Numbers | Valid Tanzanian format (+255 or 0, 10-15 digits) | "Invalid phone number format" |
| Email | Valid email format (optional field) | "Invalid email format" |
| Amounts | Positive integers, max 10,000,000 TZS | "Amount must be between 0 and 10,000,000 TZS" |
| Payment Date | Cannot be future date, cannot be before term start | "Payment date must be within term dates" |
| Receipt No | Auto-incremented, unique per academic year | "Receipt number generation failed" |
| Student Name | 2-100 characters, alphabetic with spaces | "Name must be 2-100 characters" |
| Gender | Must be M or F | "Gender must be Male or Female" |

### Business Rules
1. **Payment Deadlines**: Each term has a fee deadline; overdue = debt record created automatically
2. **ID Re-issue**: Format `###/1/26` indicates re-issue count and year; tracked in system_logs
3. **Family Discounts**: Applied at family level, not per student; cannot exceed 50% of total fees
4. **Transport**: Billed monthly on the 1st via Node.js cron job; can start/stop independently of tuition; minimum 1 month commitment
5. **Promotions**: Student history must be preserved when changing classes; promotion requires minimum fee clearance (70%)
6. **Receipt Numbers**: Sequential, unique per academic year; cannot be reused even if voided
7. **Currency**: All amounts in Tanzanian Shillings (TZS); integers only (no decimals)
8. **Duplicate Prevention**: No duplicate admission numbers or ID pass numbers; checked before save
9. **Minimum Payment**: Term fee must be at least 50% paid to sit for exams (configurable by Director)
10. **Promotion Eligibility**: Student must have cleared minimum 70% balance to be promoted (configurable)
11. **Withdrawal**: Outstanding balance must be cleared before withdrawal approval; requires Director approval
12. **Voiding Policy**: Payments CANNOT be deleted; must be marked as `voided` with mandatory reason; requires Director approval for amounts > 100,000 TZS
13. **Allocation Priority**: Payments allocated in order: Oldest Debt → Current Tuition → Transport → Stationery
14. **Material Tracking**: "Reams" and "Files" tracked as fulfilled/pending separately from cash balance

Glossary of Terms

| Term | Definition | Example |
|------|------------|--------|
| **DENI** | Swahili for "debt" - outstanding fee balance carried forward | "DENI 236,500" = owes 236,500 TZS |
| **REAM** | Stationery item (500 sheets of paper) charged as part of fees | "1 REAM" = one ream of paper |
| **ID PASS** | Student identification card/pass | "ID PAID 411" = ID number 411 issued |
| **KOFIA** | School cap/uniform item | "KOFIA 6,000" = cap costs 6,000 TZS |
| **MADAM** | School Director/Headmistress | "IN TALKS WITH MADAM" = discussing with Director |
| **TERM** | Academic period (2nd Term: April 8 - June 5, 2026) | "2ND TERM 2026" |
| **ANNUAL** | Full academic year fee payment | "ANNUAL 500,000" = full year fees |
| **SPONSORED** | Student whose fees are paid by external sponsor | "SPONSORED 1 REAM" |
| **NEW** | Student newly admitted in current term | "NEW 2026" |
| **PROMOTED** | Student moved to higher class mid-year | "[PROMOTED]" |
| **DATELINE** | Payment deadline | "DATELINE 31/05/2026" |
| **BAL** | Balance remaining | "BAL 5,000" = owes 5,000 TZS |
| **NIL** | Zero amount paid or owed | "NIL" = nothing |

---

## Step 9: Non-Functional Requirements

| Requirement | Description | Acceptance Criteria |
|-------------|-------------|--------------------|
| **Data Backup** | Daily automated backups; manual export on-demand | Backup completes by 2 AM daily; restore tested monthly |
| **Audit Trail** | Log all payment creations/modifications with user and timestamp | 100% of payment changes logged; searchable by user/date |
| **Multi-Term Support** | System handles multiple concurrent terms and historical data | Can view 3+ years of historical data without performance degradation |
| **Offline Capability** | PWA support for offline payment entry (sync when online) | Can record 50+ payments offline; sync on reconnection |
| **Data Migration** | Script to import existing student records from document/Excel | 95%+ auto-migration rate; manual review for exceptions |
| **Performance** | Load student list < 2 seconds; generate reports < 5 seconds | Tested with 500+ students, 10,000+ payments |
| **Security** | Password hashing, role-based access, session timeout | OWASP Top 10 compliance; session timeout after 30 min |
| **Scalability** | Support 500+ students, 5+ concurrent users | No degradation with 10 concurrent users |
| **Accessibility** | WCAG 2.1 AA compliance for basic navigation | Keyboard navigation; screen reader compatible |
| **Localization** | English (Swahili option for future) | All UI text externalized for translation |
| **Uptime** | 99% availability during school hours (7 AM - 6 PM) | Monitoring with alerts for downtime > 5 minutes |

---