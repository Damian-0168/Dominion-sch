## Final Student Promotion Algorithm — v2

### Definitions

```
ENROLLMENT_STATUS  : ACTIVE | COMPLETED | WITHDRAWN
PROMOTION_OUTCOME  : PROMOTED | RETAINED | DEMOTED | GRADUATED |
                     PENDING | SKIPPED | INTEGRITY_ERROR | FAILED
PROMOTION_MODES    : AUTOMATIC | RESULT_BASED
STUDENT_STATUS     : ACTIVE | INACTIVE | GRADUATED
```

---

### School-Level Configuration

```
Settings (configured per school, not hardcoded):
    promotion_mode            : AUTOMATIC | RESULT_BASED
    promotion_cutoff_score    : DECIMAL          ← e.g. 50.0
    attendance_required       : BOOLEAN
    max_repeats_before_action : INTEGER          ← e.g. 2
    action_after_max_repeats  : DEMOTE | FORCE_PROMOTE | MANUAL_REVIEW
```

---

### Phase 0: Pre-flight Validation

```
current_year ← AcademicYear WHERE starting_date <= TODAY <= end_date
IF current_year is null:
    ABORT "No active academic year found"

next_year ← AcademicYear
            WHERE starting_date > current_year.end_date
            ORDER BY starting_date ASC
            LIMIT 1
IF next_year is null:
    ABORT "Next academic year not configured"

settings ← load school settings
run_id   ← generate UUID
```

---

### Phase 1: Process Withdrawals

*Separate pass, before promotion. Each withdrawal is its own transaction.*

```
FOR EACH enrollment WHERE status        = ACTIVE
                      AND academic_year = current_year:

    IF student has withdrawal request:

        BEGIN TRANSACTION
            enrollment.status   ← WITHDRAWN
            enrollment.end_date ← TODAY
            student.status      ← INACTIVE

            WRITE AuditLog(
                run_id       = run_id,
                student_id   = student.id,
                outcome      = WITHDRAWAL_PROCESSED,
                level_from   = enrollment.level,
                processed_at = NOW()
            )
        COMMIT
        ON FAILURE → ROLLBACK, log error, continue
```

---

### Phase 2: Promotion

*Only students where `student.status = ACTIVE`. Each student is its own isolated transaction.*

```
FOR EACH student WHERE student.status = ACTIVE:


    ── Step 1: Resolve active enrollment ────────────────────────────────────

    active_enrollments ← SELECT ... FOR UPDATE
                         FROM Enrollments
                         WHERE student      = student
                           AND academic_year = current_year
                           AND status        = ACTIVE

    IF count = 0:
        SKIP  ← not enrolled this year

    IF count > 1:
        WRITE AuditLog(outcome = INTEGRITY_ERROR,
                       notes   = "multiple active enrollments detected")
        FLAG for manual review
        SKIP

    enrollment    ← active_enrollments[0]
    current_level ← enrollment.level


    ── Step 2: Duplicate guard ───────────────────────────────────────────────

    ← DB enforces UNIQUE(student_id, academic_year_id)
    ← Check here first to skip cleanly before opening a transaction

    existing ← Enrollment WHERE student      = student
                            AND academic_year = next_year

    IF existing found:
        WRITE AuditLog(outcome = SKIPPED,
                       notes   = "enrollment in next year already exists")
        SKIP  ← safe to re-run job


    ── Step 3: Graduation check ──────────────────────────────────────────────

    next_level ← Level WHERE order_key = current_level.order_key + 1

    IF next_level is null:
        ← student is in Std 7, the terminal level

        BEGIN TRANSACTION
            enrollment.status   ← COMPLETED
            enrollment.end_date ← current_year.end_date
            student.status      ← GRADUATED

            WRITE AuditLog(
                outcome    = GRADUATED,
                level_from = current_level,
                level_to   = null
            )
        COMMIT
        ON FAILURE → ROLLBACK, log error
        CONTINUE


    ── Step 4: Determine target level ───────────────────────────────────────

    IF settings.promotion_mode = AUTOMATIC:

        IF settings.attendance_required = TRUE:
            IF student has no attendance records this year:
                WRITE AuditLog(outcome = PENDING,
                               notes   = "attendance required but not recorded")
                FLAG for manual review
                SKIP

        target_level ← next_level
        outcome      ← PROMOTED


    IF settings.promotion_mode = RESULT_BASED:

        result ← YearlyResult WHERE student      = student
                                AND academic_year = current_year
                                AND status        = FINALIZED_AND_PUBLISHED

        IF result not found:
            enrollment.promotion_status ← PENDING
            WRITE AuditLog(outcome = PENDING,
                           notes   = "result not finalized or published")
            SKIP  ← Phase 3 retry job picks this up daily

        average_score ← result.average_score  ← all subjects, equal weight

        repeat_count ← COUNT of past enrollments
                        WHERE student = student
                          AND level   = current_level
                          AND promotion_outcome IN (RETAINED, DEMOTED)

        IF average_score >= settings.promotion_cutoff_score:
            target_level ← next_level
            outcome      ← PROMOTED

        ELSE IF repeat_count >= settings.max_repeats_before_action:

            SWITCH settings.action_after_max_repeats:

                CASE DEMOTE:
                    prev_level ← Level WHERE order_key = current_level.order_key - 1
                    IF prev_level is null:
                        ← already at Baby Class, cannot demote further
                        target_level ← current_level
                        outcome      ← RETAINED
                        FLAG for manual review, notes = "demotion impossible at lowest level"
                    ELSE:
                        target_level ← prev_level
                        outcome      ← DEMOTED

                CASE FORCE_PROMOTE:
                    target_level ← next_level
                    outcome      ← PROMOTED

                CASE MANUAL_REVIEW:
                    WRITE AuditLog(outcome = PENDING,
                                   notes   = "max repeats reached, awaiting admin decision")
                    FLAG for manual review
                    SKIP

        ELSE:
            target_level ← current_level
            outcome      ← RETAINED


    ── Step 5: Commit promotion ──────────────────────────────────────────────

    BEGIN TRANSACTION

        enrollment.status           ← COMPLETED
        enrollment.end_date         ← current_year.end_date
        enrollment.promotion_outcome← outcome      ← stored on closing enrollment

        CREATE Enrollment(
            student           = student,
            level             = target_level,
            academic_year     = next_year,
            status            = ACTIVE,
            start_date        = next_year.starting_date,
            promotion_status  = SETTLED
        )
        ← DB unique constraint on (student_id, academic_year_id)
          will reject duplicates as a final safety net

        WRITE AuditLog(
            run_id                = run_id,
            student_id            = student.id,
            outcome               = outcome,
            promotion_mode        = settings.promotion_mode,
            level_from            = current_level,
            level_to              = target_level,
            year_from             = current_year,
            year_to               = next_year,
            average_score         = average_score OR null,
            repeat_count_at_level = repeat_count OR null,
            processed_at          = NOW(),
            notes                 = null OR flag message
        )

    COMMIT
    ON FAILURE → ROLLBACK
                 WRITE AuditLog(outcome = FAILED, notes = error_message)
                 continue to next student
```

---

### Phase 3: Retry Pending Students

*Lightweight scheduled job. Runs daily. Only touches PENDING students.*

```
FOR EACH enrollment WHERE promotion_status = PENDING
                      AND academic_year    = current_year
                      AND student.status   = ACTIVE:

    Re-enter Phase 2, Step 2 onward for this student only
    ← Phase 2 duplicate guard ensures no double processing
```

---

### Phase 4: New Enrollments

*Completely separate from the promotion job. No promotion logic involved.*

```
Case A — New student joining any level mid-year or at year start:
    Admin manually creates Enrollment(student, level, academic_year)

Case B — Baby Class intake for a new academic year:
    Bulk intake process creates Enrollment(student, Baby Class, next_year)
    Triggered independently after next_year is configured
```

---

### Phase 5: Post-Run Report

```
REPORT for run_id:
    processed_at         ← timestamp of run
    academic_year        ← current_year
    total_processed      ← count of students evaluated

    outcome_summary:
        PROMOTED         ← count
        RETAINED         ← count
        DEMOTED          ← count
        GRADUATED        ← count
        PENDING          ← count
        SKIPPED          ← count
        INTEGRITY_ERROR  ← count
        FAILED           ← count

    flagged_students     ← list with student_id and reason
    pending_students     ← list awaiting result publication
```

---

### Database Constraints Supporting the Algorithm

```sql
── Prevents any duplicate enrollment regardless of level
ALTER TABLE enrollments
    ADD CONSTRAINT uq_student_academic_year
    UNIQUE (student_id, academic_year_id);

── Enrollment stores its closing outcome
ALTER TABLE enrollments
    ADD COLUMN promotion_outcome VARCHAR   -- PROMOTED | RETAINED | DEMOTED | GRADUATED | null
    ADD COLUMN promotion_status  VARCHAR   -- SETTLED | PENDING | null
    ADD COLUMN end_date          DATE;
```

---

### Audit Log Schema

```
PromotionAuditLog:
    id                     PK
    run_id                 UUID
    student_id             FK
    outcome                ENUM(PROMOTION_OUTCOME)
    promotion_mode         ENUM(PROMOTION_MODES)  | null
    level_from             FK → levels             | null
    level_to               FK → levels             | null
    year_from              FK → academic_years     | null
    year_to                FK → academic_years     | null
    average_score          DECIMAL                 | null
    repeat_count_at_level  INTEGER                 | null
    processed_at           TIMESTAMP
    notes                  TEXT                    | null
```

---

### Result Correction Policy

```
Once a promotion is committed, it is FINAL.

If a result is corrected after promotion:
    → Admin uses the Manual Override system
    → Override creates a new enrollment, closes the incorrect one
    → Both the original and override are recorded in AuditLog
    → Automated promotion job is never reversed automatically
```