# Dominion School System Developer Plan

## 1. Project Goal

Build a focused school fee and student-accounting system for Dominion School using Laravel for the backend and React + TypeScript for the frontend, inside one Laravel application.

The first version is not a full school management system. It should cover the accounting and student-record features needed from the source documents and leave room for later expansion.

## 2. Required Stack

- Backend: Laravel 12
- PHP: 8.3
- Frontend: React + TypeScript
- Frontend integration: Laravel Breeze + Inertia.js
- Database: MySQL 8
- Web server: Apache
- Build tool: Vite
- Containers: Docker + Docker Compose

## 3. Architecture Decision

Use a single Laravel codebase.

- Laravel handles routing, authentication, authorization, validation, jobs, and database access
- React + TypeScript powers the UI through Inertia
- Apache serves the Laravel application
- MySQL stores all business data
- Docker remains mandatory for local development and deployment consistency

Do not build:

- a separate Node/Express backend
- a separate SPA deployment with JWT auth
- a separate Nginx frontend container

## 4. Product Scope for Version 1

The developer should implement these modules first:

- authentication and role-based access
- guardians
- students
- levels
- academic years and terms
- enrollments
- school fee setup
- other payment setup
- student charges
- payments and payment allocations
- balances and debts
- comments
- promotion workflow
- core reports

These are out of scope for version 1 unless explicitly re-added later:

- inventory management
- parent portal
- SMS/WhatsApp messaging
- online payments
- broad academic management

## 5. Core Business Rules

The implementation must follow these rules:

- one student belongs to one parent/guardian
- one parent/guardian can have many students
- no `families` table should be created
- each student must belong to a level through `enrollments`
- levels are fixed from `0` to `9`
- each level has its own school fee amount
- slow learners use the normal school fee plus an additional tuition amount if flagged
- sponsored students must be tracked explicitly
- ID pass must support paid and not paid states
- debts must be computed from charges and allocations, not stored as a separate source of truth
- comments can be added and cleared at any time
- promotion must follow `promotion.md`

## 6. Level Structure

Seed the following levels:

| code | name |
|------|------|
| 0 | Baby Class |
| 1 | Middle Class |
| 2 | Pre-Unit |
| 3 | Standard 1 |
| 4 | Standard 2 |
| 5 | Standard 3 |
| 6 | Standard 4 |
| 7 | Standard 5 |
| 8 | Standard 6 |
| 9 | Standard 7 |

`Standard 7` is the terminal level for promotion.

## 7. Data Model

## 7.1 Core Tables

### `users`

Use Laravel auth users plus role handling.

Minimum roles:

- admin
- accounts
- director
- receptionist

### `guardians`

Suggested columns:

- `id`
- `full_name`
- `phone`
- `email`
- `address`
- `notes`
- timestamps

### `students`

Suggested columns:

- `id`
- `admission_no`
- `guardian_id`
- `first_name`
- `middle_name`
- `last_name`
- `gender`
- `date_of_birth`
- `status`
- `id_pass_no`
- `id_pass_status`
- `is_sponsored`
- `sponsor_name`
- `is_slow_learner`
- `remarks`
- timestamps

Relationship:

- `guardians.id` -> `students.guardian_id`

### `academic_years`

Suggested columns:

- `id`
- `name`
- `start_date`
- `end_date`
- `is_active`
- timestamps

### `terms`

Suggested columns:

- `id`
- `academic_year_id`
- `name`
- `start_date`
- `end_date`
- `is_active`
- timestamps

### `levels`

Suggested columns:

- `id`
- `code`
- `name`
- `sort_order`
- `is_terminal`
- timestamps

### `enrollments`

Suggested columns:

- `id`
- `student_id`
- `academic_year_id`
- `level_id`
- `status`
- `promotion_outcome`
- `promotion_status`
- `start_date`
- `end_date`
- timestamps

Required constraint:

- unique `(student_id, academic_year_id)`

## 7.2 Billing Tables

### `school_fees`

Purpose:

- store the normal tuition amount per level and term
- store any extra amount to charge flagged slow learners

Suggested columns:

- `id`
- `academic_year_id`
- `term_id`
- `level_id`
- `amount`
- `slow_learner_additional_amount`
- `is_active`
- timestamps

### `other_payments`

Purpose:

- store non-tuition payment definitions

Suggested columns:

- `id`
- `code`
- `name`
- `category`
- `amount`
- `billing_mode`
- `is_active`
- timestamps

Expected records include:

- transport
- ream
- file
- uniform
- kofia
- tracksuit
- sweater
- shoes
- socks
- report book
- ID pass
- remedial

### `student_charges`

Purpose:

- hold the actual billable items applied to students

Suggested columns:

- `id`
- `student_id`
- `academic_year_id`
- `term_id`
- `level_id`
- `charge_type`
- `reference_id`
- `description`
- `amount`
- `due_date`
- `status`
- timestamps

Examples:

- school fee charge
- slow learner extra tuition charge
- transport charge
- ream charge
- uniform charge
- ID pass charge
- adjustment or discount charge

### `discounts`

Purpose:

- track approved discounts separately

Suggested columns:

- `id`
- `scope_type`
- `scope_id`
- `discount_type`
- `amount`
- `reason`
- `academic_year_id`
- `term_id`
- `approved_by`
- `is_active`
- timestamps

Use `scope_type` for:

- `student`
- `guardian`

### `payments`

Suggested columns:

- `id`
- `guardian_id`
- `payment_date`
- `amount`
- `payment_method`
- `reference_no`
- `recorded_by`
- `notes`
- `status`
- `void_reason`
- timestamps

### `payment_allocations`

Suggested columns:

- `id`
- `payment_id`
- `student_charge_id`
- `allocated_amount`
- timestamps

### `comments`

Suggested columns:

- `id`
- `commentable_type`
- `commentable_id`
- `comment_text`
- `created_by`
- `cleared_at`
- `cleared_by`
- timestamps

### `promotion_audit_logs`

Suggested columns:

- `id`
- `run_id`
- `student_id`
- `outcome`
- `promotion_mode`
- `level_from`
- `level_to`
- `year_from`
- `year_to`
- `average_score`
- `repeat_count_at_level`
- `notes`
- `processed_at`

## 8. Balance and Debt Logic

Do not create a standalone debt source table for the MVP.

Debt must be derived from billing data:

- student balance = total student charges - allocated payments
- guardian balance = sum of balances of all students linked to that guardian

The UI should support:

- per-student balance
- per-guardian combined balance
- level-based debt listing

## 9. Authentication and Authorization

Use Laravel Breeze with React + TypeScript.

Authentication requirements:

- session-based auth
- password hashing with Laravel defaults
- protected routes with middleware

Authorization requirements:

- admin: full access
- accounts: students, charges, payments, reports, comments
- director: approvals, discounts, promotions, reports, write-offs
- receptionist: read-heavy access with limited data entry

## 10. Required Screens

The developer should implement at least these pages:

- login
- dashboard
- guardians list and form
- students list and form
- student profile
- fee setup
- other payments setup
- charge entry
- payment entry
- guardian account summary
- debtors report
- promotion management
- comments/history view

## 11. Student Profile Requirements

Each student profile should show:

- student identity details
- parent/guardian details
- current level and enrollment
- school fee setup for the current term
- other charges
- payment history
- current debt
- ID pass status
- sponsored status
- slow learner flag
- comments

## 12. Charge Generation Rules

The system should support these charge scenarios:

- apply school fee for a student based on level and term
- if student is marked slow learner, add the extra tuition amount from `school_fees`
- apply transport as an `other_payments` charge
- apply one-time items such as uniform, ream, kofia, or ID pass
- apply discounts as negative adjustments or discount-linked charge reductions

## 13. Payment Rules

Payment flow should work like this:

- payment is received from a guardian
- one payment can be allocated across multiple students under the same guardian
- one payment can be split across multiple charges
- voiding is allowed only by approved roles and must keep audit information

## 14. Promotion Requirements

Promotion must follow `promotion.md`.

Implementation rules:

- use `enrollments` as the source of academic placement
- only one active enrollment per student per academic year
- support automatic and result-based promotion modes
- support pending promotion cases
- mark Standard 7 students as graduated
- keep promotion audit logs
- support retry processing for pending students

## 15. Docker Deliverables

The developer should provide:

- `Dockerfile` for Laravel + Apache
- `docker-compose.yml`
- MySQL service with persistent volume
- Node service for frontend asset development if needed
- environment examples for local setup

Expected services:

- `app`
- `mysql`
- `node`
- `phpmyadmin` optional

## 16. Implementation Phases

### Phase 1: Scaffold

- create Laravel project
- install Breeze with React + TypeScript
- configure Apache container
- configure MySQL container
- confirm app boots in Docker

### Phase 2: Core Schema

- create migrations for `users`, `guardians`, `students`, `academic_years`, `terms`, `levels`, `enrollments`
- add indexes and foreign keys
- seed default roles and levels

### Phase 3: Billing Schema

- create migrations for `school_fees`, `other_payments`, `student_charges`, `discounts`, `payments`, `payment_allocations`, `comments`, `promotion_audit_logs`

### Phase 4: Core CRUD

- guardians CRUD
- students CRUD
- academic years and terms CRUD
- levels read/seed management

### Phase 5: Accounting

- fee setup screens
- charge creation logic
- payment entry and allocation logic
- balance computation
- debt dashboards

### Phase 6: Operations

- comments add/clear flow
- sponsored student handling
- ID pass tracking
- slow learner extra tuition support

### Phase 7: Promotion

- promotion settings
- promotion command/job
- retry command/job
- promotion reports

### Phase 8: Reporting

- collections report
- debtors report
- guardian combined summary
- student statement
- level fee summary

## 17. Acceptance Criteria

The developer work should be considered acceptable when:

- the app runs in Docker using Apache and MySQL
- login works with Laravel auth
- students can be linked to exactly one guardian
- one guardian can have multiple students
- level-based school fees work
- slow learner extra tuition can be added automatically
- transport and uniform-like items can be added through `other_payments`
- balances are computed correctly from charges and allocations
- guardian combined balances display correctly
- comments can be added and cleared
- promotion logic follows `promotion.md`

## 18. Recommended First Delivery

The first delivery from the developer should contain:

1. Laravel app scaffold with Breeze React/TypeScript
2. Docker setup with Apache and MySQL
3. core migrations and seeders
4. authentication and roles
5. guardian and student CRUD
6. school fee and other payment setup
7. basic charge and payment flow
8. student and guardian balance views

This file should be used as the working implementation brief for the developer.
