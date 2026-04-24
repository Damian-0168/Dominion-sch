# Student Promotion Algorithm — Logic & Usage Guide

This document provides a non-technical breakdown of the `promotion.md` algorithm and how it functions within the School Management System (SMS).

---

### 1. The Configuration Phase (Setting the Rules)
Before the promotion process begins, administrators define the school's policy in the settings. This ensures the system adapts to the school's specific needs rather than using hardcoded rules.

*   **Promotion Mode:** 
    *   `AUTOMATIC`: Every student moves to the next grade regardless of performance.
    *   `RESULT_BASED`: Students must achieve a minimum average score to be promoted.
*   **Promotion Cutoff:** The passing mark (e.g., 50.0).
*   **Repeat Policy:** Defines how many times a student can stay in the same grade before the school takes specific action (like a manual review or forced promotion).

---

### 2. Phase 0: System Readiness (Pre-flight)
The system performs a "sanity check" before touching any student records. It ensures:
1.  There is an **active** academic year ending.
2.  The **next** academic year is already created and ready for students.
If these aren't set up, the system stops immediately to prevent students from being "promoted into a vacuum."

---

### 3. Phase 1: Handling Departures (Withdrawals)
Before promoting the "stayers," the system processes the "leavers." Any student with a pending withdrawal request is marked as `INACTIVE`. This prevents students who have already left the school from accidentally appearing on next year’s class lists.

---

### 4. Phase 2: The Decision Engine (Core Promotion)
The system processes every active student one by one. For each student, it follows these sub-steps:

*   **Duplicate Prevention:** It checks if the student is already enrolled in the next year. This allows the job to be restarted safely if it crashes halfway through.
*   **Graduation Check:** If a student is in the highest grade (e.g., Standard 7), they are marked as `GRADUATED` and their school journey is closed.
*   **The Decision:**
    *   **In Automatic Mode:** The student is simply assigned to the next grade level.
    *   **In Result-Based Mode:** The system looks for the student's **Finalized Yearly Result**. 
        *   **Pass:** They move to the next grade.
        *   **Fail:** They are `RETAINED` (repeat) or handled based on the "Max Repeats" policy.
        *   **Missing Marks:** If marks aren't ready, the student is marked as `PENDING` and skipped for now.
*   **The Transaction:** If a decision is made, the system closes the old year and opens the new year record simultaneously. If any error occurs, it rolls back the change so no data is left "half-finished."

---

### 5. Phase 3: The "Catch-up" Job (Retries)
Students who were marked as `PENDING` (usually because their marks weren't finalized yet) are not forgotten. A daily background task automatically re-checks these students. Once their results are published, the system finishes their promotion.

---

### 6. Phase 4: Accountability (Audit Logs & Reporting)
The system records every single movement in a **Promotion Audit Log**. This is a permanent record that answers:
*   "Why was Student X retained?" (e.g., Average score was 42.0, below cutoff of 50.0).
*   "Who promoted Student Y?" (e.g., The Automated System on Dec 15th).

After the process, the SMS generates a **Post-Run Report** for the Principal, highlighting students who need manual attention or those who successfully graduated.

---

### 7. The Safety Net (Database Constraints)
At the lowest level, the database has a "Unique Constraint." This is a hard rule that prevents a student from ever having two enrollments in the same academic year. This is the ultimate shield against double-billing or duplicate class lists.
