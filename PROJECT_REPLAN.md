# Dominion School System Replan

## 1. Current Position

Based on the repository contents, the project is currently at the planning stage, not at an implemented application stage. The existing documents describe:

- Node.js + Express backend
- React + TypeScript frontend
- Prisma + MySQL
- Nginx as the production web server
- A broad schema that tries to cover full school management early

That old direction is now misaligned with the current goal. The main gaps are:

- Backend stack should move from Node.js/Express to Laravel
- Web server should move from Nginx to Apache
- Authentication should follow Laravel's native approach
- The schema is too wide for the current product scope
- Some tables model future modules too early instead of the school's immediate accounting needs

## 2. New Product Direction

The system should become a focused school fee and student-accounting platform first, not a full school management system from day one.

The MVP should focus on:

- student registration and level placement
- parent/guardian relationships
- level-based school fees
- other payments tracking
- debts/balances per student
- combined billing view for guardians with multiple students
- discounts and sponsored students
- comments/notes for operational follow-up
- promotion workflow based on `promotion.md`

Modules such as inventory, messaging, parent portal, and broader school operations should be added later as updates, not built into the first release.

## 3. Recommended Architecture

### Core Stack

- **Backend:** Laravel 12
- **PHP Runtime:** PHP 8.3
- **Frontend:** React + TypeScript
- **Laravel Frontend Integration:** Laravel Breeze with React + TypeScript + Inertia.js
- **Database:** MySQL 8
- **Web Server:** Apache
- **Assets:** Vite
- **Containerization:** Docker + Docker Compose

### Why This Stack

Laravel Breeze with React + TypeScript gives the project:

- Laravel-native authentication and session handling
- React + TypeScript without maintaining a separate auth system
- simple role/middleware integration
- a cleaner path for future admin modules

This is a better fit than keeping a fully separate SPA with custom JWT auth, because the requirement is to stay compatible with Laravel's default UI and authentication flow.

### Docker Layout

Recommended Docker services:

- `app`: Laravel + Apache container
- `mysql`: MySQL 8 with persistent volume
- `node`: Vite/dev assets container for local development
- `phpmyadmin`: optional admin container for database inspection

Production serving model:

- Apache serves Laravel
- Laravel serves the built React assets
- no separate frontend Nginx container is needed

## 4. Domain Simplification Principles

The new schema should be intentionally smaller.

### Keep

- academic years and terms
- levels
- students
- guardians
- enrollments
- fee setup
- charges
- payments
- comments
- promotion audit history

### Remove From MVP

These should not be first-class modules in the first version:

- `families`
- `inventory_items`
- `student_inventory`
- `transport` as a standalone service table
- `debt_records` as a standalone ledger
- `receipts` as a separate table unless later required
- `notifications`
- broad audit/event subsystems outside what promotion and payments need

### Key Simplification Rule

Debt should be derived from charges minus allocations, not stored as a separate truth table.

That means:

- each student gets charge entries
- each payment is allocated against those charges
- balance/debt is computed from unpaid charge amounts

This avoids duplicated debt data and reduces inconsistencies.

## 5. Revised Database Model

## 5.1 Core Tables

### `users`

Laravel auth users with roles such as:

- admin
- accounts
- director
- receptionist

### `guardians`

Stores parent/guardian details.

Suggested fields:

- `id`
- `full_name`
- `phone`
- `email`
- `address`
- `notes`

### `students`

Stores student identity and per-student flags.

Suggested fields:

- `id`
- `admission_no`
- `first_name`
- `middle_name`
- `last_name`
- `gender`
- `date_of_birth`
- `status` (`active`, `inactive`, `graduated`, `withdrawn`)
- `id_pass_no`
- `id_pass_status` (`paid`, `not_paid`, `issued_pending_payment`)
- `guardian_id`
- `is_sponsored`
- `sponsor_name`
- `is_slow_learner`
- `remarks`

Relationship rule:

- one student belongs to one parent/guardian
- one parent/guardian can have many students
- no `families` table is needed

### `academic_years`

- `id`
- `name`
- `start_date`
- `end_date`
- `is_active`

### `terms`

- `id`
- `academic_year_id`
- `name`
- `start_date`
- `end_date`
- `is_active`

### `levels`

Levels replace free-form classes.

Suggested fixed mapping:

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

Suggested fields:

- `id`
- `code`
- `name`
- `sort_order`
- `is_terminal`

### `enrollments`

This table is essential and should remain central because it supports the promotion algorithm properly.

Suggested fields:

- `id`
- `student_id`
- `academic_year_id`
- `level_id`
- `status` (`active`, `completed`, `withdrawn`)
- `promotion_outcome`
- `promotion_status`
- `start_date`
- `end_date`

Critical constraint:

- unique `(student_id, academic_year_id)`

This matches the promotion rules in `promotion.md`.

## 5.2 Fee Structure Split

The old `fee_structures` table should be replaced with two fee-definition tables.

### `school_fees`

This stores the main tuition fee per level.

Suggested fields:

- `id`
- `academic_year_id`
- `term_id`
- `level_id`
- `amount`
- `slow_learner_additional_amount`
- `is_active`

Purpose:

- each level has its own school fee amount
- slow learners can have an additional tuition amount on top of the normal school fee when flagged

### `other_payments`

This stores all non-tuition fee definitions.

Suggested fields:

- `id`
- `code`
- `name`
- `category`
- `amount`
- `billing_mode` (`one_time`, `termly`, `monthly`, `manual`)
- `is_active`

Examples under `other_payments`:

- transport fee
- ream
- file
- report book
- uniform
- kofia
- tracksuit
- sweater
- shoes
- socks
- ID pass
- remedial

This satisfies the requirement that `fee_structure` should have two parts:

- `school_fees`
- `other_payments`

## 5.3 Charges, Discounts, and Payments

### `student_charges`

This is the operational billing table.

Each record is a charge applied to a specific student for a specific term or billing period.

Suggested fields:

- `id`
- `student_id`
- `academic_year_id`
- `term_id`
- `level_id`
- `charge_type` (`school_fee`, `other_payment`, `discount`, `adjustment`)
- `reference_id`
- `description`
- `amount`
- `due_date`
- `status` (`open`, `partial`, `paid`, `void`)

Examples:

- Baby Class school fee for Term 2
- slow learner additional tuition charge
- transport for April
- 1 ream
- uniform charge
- ID pass charge
- discount adjustment

### `discounts`

Keep discounts separate so they are traceable and can be applied at either student or guardian scope.

Suggested fields:

- `id`
- `scope_type` (`student`, `guardian`)
- `scope_id`
- `discount_type` (`fixed`, `percentage`)
- `amount`
- `reason`
- `academic_year_id`
- `term_id`
- `approved_by`
- `is_active`

This supports:

- annual discount arrangements
- sibling/combined guardian discounts without a family table
- ad hoc director-approved reductions

### `payments`

Payments should be recorded against the paying guardian when possible, while still supporting student-level views.

Suggested fields:

- `id`
- `guardian_id`
- `payment_date`
- `amount`
- `payment_method`
- `reference_no`
- `recorded_by`
- `notes`
- `status` (`active`, `voided`)
- `void_reason`

### `payment_allocations`

Each payment may settle several student charges across multiple students linked to the same guardian.

Suggested fields:

- `id`
- `payment_id`
- `student_charge_id`
- `allocated_amount`

This is what makes combined guardian accounting work cleanly.

## 5.4 Comments and Promotion Logs

### `comments`

Comments should be lightweight and easy to add or clear.

Recommended approach:

- use a polymorphic comment table

Suggested fields:

- `id`
- `commentable_type`
- `commentable_id`
- `comment_text`
- `created_by`
- `cleared_at`
- `cleared_by`

This allows comments on:

- students
- payments
- guardians
- promotion cases

### `promotion_audit_logs`

Keep this directly aligned with `promotion.md`.

Suggested fields:

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

## 6. Core Business Rules To Preserve

The redesign should keep these rules:

- each student belongs to a level through `enrollments`
- each level has its own school fee amount
- slow learners may receive an additional tuition charge if marked as one
- sponsored students must be explicitly flagged
- ID pass must track paid vs not paid status per student
- transport is a charge type, not a separate MVP subsystem
- students under one guardian should have a combined payment and debt view
- comments can be added and cleared at any time
- promotions must use the enrollment-based algorithm from `promotion.md`

## 7. How Debt Should Work

Debt should be a computed value, not manually maintained.

### Student Debt

For each student:

`debt = sum(open student_charges) - sum(payment_allocations)`

### Guardian Combined Debt

For each guardian:

- gather all linked students
- combine their unpaid student charges
- subtract allocations from payments made by that guardian

This directly supports the real school workflow where one parent/guardian settles fees for more than one child together.

## 8. Promotion Design

The promotion design in `promotion.md` is still valid and should be retained with minor naming alignment.

What must remain:

- enrollment-based promotion
- one active enrollment per academic year
- automatic and result-based promotion modes
- pending promotion state
- graduation at terminal level
- promotion audit logging
- retry job for pending cases

What changes:

- `classes` become `levels`
- Standard 7 is the terminal level
- level progression follows `levels.sort_order`

## 9. Laravel Module Plan

### Phase 1: Foundation

- Laravel app scaffold
- Breeze React + TypeScript auth
- role middleware
- Docker with Apache, MySQL, Node
- core migrations for users, guardians, students, levels, academic years, terms, enrollments

### Phase 2: Billing Core

- `school_fees`
- `other_payments`
- `student_charges`
- `payments`
- `payment_allocations`
- student debt view
- guardian combined debt view

### Phase 3: Operations

- discounts
- sponsored student handling
- ID pass status tracking
- comments and clearing flow
- student account statement

### Phase 4: Promotion

- promotion settings
- promotion command/job
- promotion audit logs
- retry process for pending students

### Phase 5: Reports

- collections report
- debtors report
- level-based fee summary
- guardian combined balance report
- student ledger

### Phase 6: Future Add-ons

These should be added only after the billing core is stable:

- SMS/WhatsApp notifications
- parent portal
- inventory/stock control
- online payments
- broader academic features

## 10. What To Drop From The Old Plan

The following old assumptions should be considered replaced:

- Node.js + Express backend
- Prisma ORM
- JWT-first authentication
- Nginx frontend container
- `families` table
- inventory-first schema
- transport-first standalone module
- standalone debt table

## 11. Recommended Implementation Decision

The best implementation path is:

- one Laravel project
- React + TypeScript inside Laravel using Breeze/Inertia
- Apache inside Docker
- MySQL as the main datastore
- balances derived from charges and allocations
- promotions implemented on top of `enrollments`

This keeps the system smaller, more maintainable, and easier to extend later without rebuilding the core again.

## 12. Immediate Next Build Order

If development starts from this revised plan, the first build sequence should be:

1. scaffold Laravel + Breeze React/TypeScript
2. set up Docker with Apache + MySQL + Node
3. create migrations for `levels`, `guardians`, `students`, `academic_years`, `terms`, `enrollments`
4. create migrations for `school_fees`, `other_payments`, `student_charges`, `payments`, `payment_allocations`, `comments`, `promotion_audit_logs`
5. build student and guardian CRUD
6. build fee assignment and payment allocation flows
7. build debt dashboards
8. implement promotion workflow

This document should now be treated as the new planning baseline. The old `PROJECT_STATUS.md` can remain as archive/reference unless later replaced entirely.
