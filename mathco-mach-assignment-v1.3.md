# MACH Level Assignment Framework

## Document Purpose

This document explains the complete process for assigning MACH levels to employees based on their assessment performance. It is designed for L&D consultants, talent managers, and project team members who need to understand how final MACH levels are determined.

---

## 1. Overview

### What is MACH?

MACH is a five-level framework used to classify technical skills. Each technology area (such as SQL, Python, Cloud, or Spark) has skills distributed across MACH levels 1 through 5, where:

- **MACH 1** represents foundational skills
- **MACH 5** represents advanced expert-level skills

### The Goal

After employees complete their assessments, we assign **one final MACH level per employee per technology area**. This level indicates the highest level of cumulative proficiency the employee has demonstrated.

### Core Principle: MACH is Cumulative

MACH levels build on each other. If an employee is assigned MACH 4, it means they have demonstrated proficiency at MACH 1, 2, 3, and 4. An employee cannot skip levels. If they fail at a lower level, they cannot be assigned a higher level, even if they perform well on advanced skills.

---

## 2. The Complete Process

The MACH assignment process follows six sequential steps:

### Step 1: Participants Submit Assessments

Employees complete their skills assessments. Each assessment captures their performance on individual skills within a technology area.

### Step 2: Evaluation in Creator Account

Mentors evaluate their respective parts of the assessment and provide scores to the skills demonstrated by the employees. Each skill is scored between 1 and 10 based on the employee's performance.

### Step 3: Data Export from Database

The assessment scores are exported into sheets from the database using a deployment ID.

### Step 4: Data Preparation in Google Sheets

The exported data is organized in a Google Sheet. Below is an example of how the data appears:

| email | skill_name | avg_total_score | avg_score_obtained | MACH Level of Skill | Proficient |
|-------|------------|-----------------|-------------------|---------------------|------------|
| john.doe@mathco.com | indexing | 10 | 8 | MACH 5 | 1 |
| john.doe@mathco.com | conditional-logic | 10 | 10 | MACH 2 | 1 |
| john.doe@mathco.com | cte | 10 | 10 | MACH 3 | 1 |
| john.doe@mathco.com | date-processing | 10 | 7 | MACH 2 | 1 |
| john.doe@mathco.com | filter | 10 | 5 | MACH 1 | 0 |
| john.doe@mathco.com | group-by-aggregate | 10 | 10 | MACH 2 | 1 |
| john.doe@mathco.com | join | 10 | 9 | MACH 2 | 1 |
| john.doe@mathco.com | procedure | 10 | 10 | MACH 4 | 1 |
| john.doe@mathco.com | recursive-cte | 10 | 0 | MACH 4 | 0 |
| john.doe@mathco.com | sort | 10 | 3 | MACH 1 | 0 |
| john.doe@mathco.com | text-processing | 10 | 10 | MACH 1 | 1 |
| john.doe@mathco.com | window | 10 | 8 | MACH 3 | 1 |
| john.doe@mathco.com | materialized-views | 10 | 10 | MACH 5 | 1 |

**Column Descriptions:**

- **Email**: Employee email address (masked for confidentiality)
- **Skill Name**: The specific skill that was tested
- **Total Possible Score**: Always 10 (maximum score per skill)
- **Score Obtained**: The employee's actual score on that skill
- **MACH Level of Skill**: The MACH classification of the skill (1-5)
- **Proficient**: A yes/no indicator showing whether the employee scored above 6 (1 = Yes, 0 = No)

### Step 5: Pivot Summary by MACH Level

The individual skill data is summarized into a pivot table that shows performance at each MACH level. Below is an example of the pivot structure:

| email | MACH | Total No. of Skills | Total Proficient Skills | Skill Proficiency Required | MACH Readiness | Leniency Flag |
|-------|------|---------------------|------------------------|---------------------------|----------------|---------------|
| john.doe@mathco.com | MACH 1 | 3 | 1 | 2 | Not Ready | 1 |
| john.doe@mathco.com | MACH 2 | 4 | 4 | 3 | Ready | 0 |
| john.doe@mathco.com | MACH 3 | 2 | 2 | 1 | Ready | 0 |
| john.doe@mathco.com | MACH 4 | 2 | 1 | 1 | Ready | 0 |
| john.doe@mathco.com | MACH 5 | 2 | 2 | 1 | Ready | 0 |

**Column Descriptions:**

For each employee and each MACH level, we calculate:

- **Total Number of Skills**: How many skills were tested at this MACH level
- **Total Proficient Skills**: How many of those skills the employee passed
- **Skill Proficiency Required**: The minimum number of skills the employee needs to pass (calculated as 65% of total skills, rounded to the nearest whole number)
- **MACH Readiness**: Shows "Ready" if the employee met or exceeded the required number, otherwise "Not Ready"
- **Leniency Flag**: Indicates whether the employee qualifies for a one-time exception (explained in detail below)

### Step 6: Final MACH Level Assignment

A systematic evaluation determines the final MACH level by reviewing MACH levels 1 through 5 in order. The process stops at the first level the employee cannot clear, and the last successfully cleared level becomes their final assignment.

---

## 3. The Rules

### Rule 1: Determining Readiness at Each MACH Level

To be considered "Ready" at a MACH level, an employee must demonstrate proficiency in at least 65% of the skills at that level. The 65% threshold was defined by the client and applies uniformly across all MACH levels and technology areas.

**Calculation:**
- Multiply the total number of skills by 0.65
- Round to the nearest whole number
- This is the minimum number of skills the employee must pass

**Example:**
- If MACH 2 has 4 skills tested
- 4 x 0.65 = 2.6, which rounds to 3
- The employee must pass at least 3 out of 4 skills to be "Ready" at MACH 2

### Rule 2: The Near-Miss Exception (Leniency Flag)

Sometimes an employee comes very close to meeting the requirement but falls short by exactly one skill. In these cases, we allow a one-time exception, but only under specific conditions:

**When the exception applies:**
- The employee must be marked "Not Ready" at that MACH level
- The employee must have been tested on 3 or more skills at that MACH level
- The employee must have missed the requirement by exactly 1 skill
- The employee can only use this exception **once** per technology area

**When the exception does NOT apply:**
- If the employee is already "Ready" at that level (no exception needed)
- If there are only 1 or 2 skills at that level (missing by 1 on a small set is not considered a near-miss)
- If the employee missed by 2 or more skills
- If the employee has already used their one-time exception at a lower level

**Why we have this rule:**

Assessments are not perfect, and small differences in performance can sometimes reflect testing conditions rather than actual capability. When an employee demonstrates strong performance across multiple skills but falls just short on one, it is reasonable to give them the benefit of the doubt once.

However, we limit this to one use per technology area for a principled reason: the exception exists to account for assessment imperfection, not to compensate for lack of knowledge. Allowing multiple exceptions would mean an employee could progress through several levels without genuinely meeting the standard — which is unfair to employees who cleared those levels on their own merit.

### Rule 3: Sequential Evaluation with One Exception Allowed

The final MACH level is determined by evaluating levels 1 through 5 in strict order:

**The Process:**

1. Start at MACH 1
2. Check if the employee is "Ready" at this level
   - If yes, mark this level as cleared and move to the next level
   - If no, check if they qualify for the near-miss exception AND have not used it yet
     - If yes, mark this level as cleared using the exception, note that the exception has been used, and move to the next level
     - If no, stop the evaluation
3. Continue until you reach a level that cannot be cleared
4. The final MACH level is the last level that was successfully cleared
5. If MACH 1 cannot be cleared (even with the exception), the final result is "Not Ready"

**Important:** Once the evaluation stops at a level, all higher levels are ignored, even if the employee performed well on them. This enforces the cumulative nature of MACH.

### Possible Final MACH Levels

An employee can achieve one of the following final MACH levels:

- **MACH 1**: Cleared MACH 1 only (or used exception at MACH 1, failed at MACH 2)
- **MACH 2**: Cleared MACH 1 and 2 consecutively, failed at MACH 3
- **MACH 3**: Cleared MACH 1, 2, and 3 consecutively, failed at MACH 4
- **MACH 4**: Cleared MACH 1, 2, 3, and 4 consecutively, failed at MACH 5
- **MACH 5**: Cleared all five MACH levels consecutively
- **Not Ready**: Failed to clear MACH 1 (even with the near-miss exception)

---

## 4. Worked Examples

### Example 1: Clean Progression

**Employee Performance:**

| MACH Level | Skills Tested | Skills Passed | Required | Status |
|------------|---------------|---------------|----------|--------|
| 1 | 3 | 2 | 2 | Ready |
| 2 | 4 | 3 | 3 | Ready |
| 3 | 2 | 2 | 1 | Ready |
| 4 | 2 | 1 | 1 | Ready |
| 5 | 2 | 0 | 1 | Not Ready |

**Evaluation:**
- MACH 1: Ready - Cleared
- MACH 2: Ready - Cleared
- MACH 3: Ready - Cleared
- MACH 4: Ready - Cleared
- MACH 5: Not Ready - No exception available (only 2 skills tested, minimum of 3 required for near-miss exception) - Stop

**Final Assignment: MACH 4**

---

### Example 2: Using the Near-Miss Exception at Foundation

**Employee Performance:**

| MACH Level | Skills Tested | Skills Passed | Required | Status | Leniency Flag |
|------------|---------------|---------------|----------|--------|---------------|
| 1 | 3 | 1 | 2 | Not Ready | Yes |
| 2 | 4 | 4 | 3 | Ready | No |
| 3 | 2 | 2 | 1 | Ready | No |
| 4 | 2 | 1 | 1 | Ready | No |
| 5 | 2 | 2 | 1 | Ready | No |

**Evaluation:**
- MACH 1: Not Ready, but qualifies for exception (3 skills, missed by 1) - Cleared using exception
- MACH 2: Ready - Cleared
- MACH 3: Ready - Cleared
- MACH 4: Ready - Cleared
- MACH 5: Ready - Cleared

**Final Assignment: MACH 5**

**Note:** The employee used their one-time exception at MACH 1, but cleared all higher levels normally.

---

### Example 3: Foundation Failure with No Exception

**Employee Performance:**

| MACH Level | Skills Tested | Skills Passed | Required | Status | Leniency Flag |
|------------|---------------|---------------|----------|--------|---------------|
| 1 | 3 | 0 | 2 | Not Ready | No |
| 2 | 4 | 4 | 3 | Ready | No |
| 3 | 2 | 2 | 1 | Ready | No |
| 4 | 2 | 1 | 1 | Ready | No |
| 5 | 2 | 2 | 1 | Ready | No |

**Evaluation:**
- MACH 1: Not Ready, does not qualify for exception (0 out of 3, missed by 2) - Stop

**Final Assignment: Not Ready**

**Important:** Even though the employee performed perfectly at MACH 2 through 5, they cannot be assigned any MACH level because they failed the foundation. This enforces the cumulative principle.

---

### Example 4: Multiple Near-Misses (Only First Gets Exception)

**Employee Performance:**

| MACH Level | Skills Tested | Skills Passed | Required | Status | Leniency Flag |
|------------|---------------|---------------|----------|--------|---------------|
| 1 | 3 | 1 | 2 | Not Ready | Yes |
| 2 | 4 | 2 | 3 | Not Ready | Yes |
| 3 | 2 | 2 | 1 | Ready | No |
| 4 | 2 | 1 | 1 | Ready | No |
| 5 | 2 | 2 | 1 | Ready | No |

**Evaluation:**
- MACH 1: Not Ready, qualifies for exception - Cleared using exception (exception now used)
- MACH 2: Not Ready, qualifies for exception BUT exception already used - Stop

**Final Assignment: MACH 1**

**Note:** Both MACH 1 and MACH 2 qualified for the near-miss exception, but the employee can only use it once. The chain breaks at MACH 2.

---

### Example 5: Leniency Flag Available at Higher Level but Chain Already Broken

**Employee Performance:**

| MACH Level | Skills Tested | Skills Passed | Required | Status | Leniency Flag |
|------------|---------------|---------------|----------|--------|---------------|
| 1 | 3 | 3 | 2 | Ready | No |
| 2 | 4 | 4 | 3 | Ready | No |
| 3 | 3 | 0 | 2 | Not Ready | No |
| 4 | 4 | 2 | 3 | Not Ready | Yes |
| 5 | 2 | 2 | 1 | Ready | No |

**Evaluation:**
- MACH 1: Ready — Cleared
- MACH 2: Ready — Cleared
- MACH 3: Not Ready, does not qualify for exception (missed by 2, not 1) — Stop
- MACH 4 and MACH 5 are not evaluated

**Final Assignment: MACH 2**

**Important:** Even though MACH 4 has a leniency flag and MACH 5 is fully Ready, they are completely ignored. The chain broke at MACH 3 with a hard fail. Once the evaluation stops, all higher levels are irrelevant regardless of performance. The leniency flag at MACH 4 is never consumed.

---

### Example 6: Small Skill Count (No Exception Applied)

**Employee Performance:**

| MACH Level | Skills Tested | Skills Passed | Required | Status | Leniency Flag |
|------------|---------------|---------------|----------|--------|---------------|
| 1 | 2 | 0 | 1 | Not Ready | No |
| 2 | 5 | 4 | 3 | Ready | No |
| 3 | 3 | 2 | 2 | Ready | No |

**Evaluation:**
- MACH 1: Not Ready, does not qualify for exception (only 2 skills tested, need 3 or more) - Stop

**Final Assignment: Not Ready**

**Note:** When a MACH level has only 1 or 2 skills, the near-miss exception does not apply. The standard is strict for small skill sets.

---

## 5. Fair vs Unfair Outcomes

### What Makes the Old Approach Unfair

**Problem 1: Inconsistent Thresholds**

The previous system used different passing thresholds for different levels:
- MACH 1: 30% passing rate
- MACH 2-5: 50% passing rate

This created unfair outcomes:
- An employee could score 0 out of 3 on MACH 1 (0% proficiency) and still be assigned MACH 1 because the threshold was only 30%
- This gave a false signal that the employee had foundational skills when they had none

**Problem 2: Always Awarding at Least MACH 1**

The old system never assigned "Not Ready" as a final outcome. Everyone received at least MACH 1, even if they failed every foundational skill. This:
- Prevented honest conversations about skill gaps
- Made it impossible to identify employees who needed fundamental retraining
- Undermined the credibility of the assessment

**Problem 3: Ignoring the Cumulative Principle**

The old system would sometimes assign MACH 4 or 5 to employees who failed MACH 1 or 2, simply because they performed well on higher-level skills. This violated the core principle that MACH levels are cumulative.

### What Makes the New Approach Fair

**Consistent Standards**

All MACH levels use the same 65% threshold. This creates consistency and eliminates arbitrary differences between levels.

**Honest Outcomes**

"Not Ready" is a valid final assignment. If an employee cannot demonstrate foundational proficiency, the system reflects this honestly. This enables:
- Targeted development planning
- Clear communication about skill gaps
- Appropriate role alignment

**Cumulative Integrity**

The sequential evaluation ensures that employees cannot skip levels. If they fail at MACH 2, they cannot be assigned MACH 3, even if they aced all MACH 3 skills. This respects the building-block nature of skill development.

**Bounded Leniency**

The near-miss exception provides humanity and flexibility, but it is:
- Limited to one use per technology area
- Only available when there are 3 or more skills
- Only available when the employee missed by exactly 1 skill

This prevents abuse while acknowledging that assessments are not perfect.

---

## 6. Why the Leniency Flag Exists

### The Problem It Solves

Without the leniency flag, the system would be rigid and unforgiving. An employee who scored 2 out of 3 skills (67% when 65% is required) would fail, while an employee who scored 2 out of 3 would pass. This one-skill difference could determine someone's career trajectory.

However, if we simply lowered the threshold or made exceptions too freely, we would undermine the entire framework. Employees could progress through all five levels by consistently scoring just below the requirement.

### What the Leniency Flag Does

The leniency flag identifies when an employee has come very close to meeting the requirement under conditions where that near-miss is meaningful:

1. **Meaningful sample size**: At least 3 skills were tested (not just 1 or 2)
2. **Very close performance**: Missed by exactly 1 skill, not 2 or more
3. **One-time use**: Can only be applied once per technology area

### How It Works in Practice

The leniency flag is calculated during Step 5 (the pivot summary). For each MACH level, the system checks:

- Is the employee "Not Ready" at this level?
- Were 3 or more skills tested?
- Did the employee miss the requirement by exactly 1 skill?

If all three conditions are true, the leniency flag is set to "Yes" for that level.

During Step 6 (final assignment), the system reads these flags and applies the exception to the first level where it appears. All subsequent flags are ignored.

### Why This Approach is Defensible

When an L&D consultant or manager asks, "Why did this employee pass MACH 1 with only 1 out of 3 skills?", the answer is clear:

"The employee qualified for a one-time near-miss exception under Rule 2. They were tested on 3 or more skills and missed the requirement by exactly 1 skill. This exception can only be used once per technology area, and it was used at MACH 1. All higher levels were cleared normally."

This explanation is:
- Transparent: The rule is documented and applied consistently
- Defensible: The conditions are objective and measurable
- Fair: It gives the benefit of the doubt once, but not repeatedly
- Humane: It acknowledges that assessments are not perfect

---

## 7. Edge Cases and Special Situations

### MACH Levels Are Always Sequential

Within any technology area, MACH levels are always sequential starting from MACH 1. A tech stack may only go up to MACH 2, 3, or 4 — but it will never skip a level in the middle (for example, having MACH 1, 2, and 4 with no MACH 3). This means the evaluation always starts at MACH 1 and moves upward without any gaps.

### No Optimization of Exception Placement

The near-miss exception is always applied at the first eligible level encountered sequentially. No attempt is made to find the most impactful position for the exception. This is intentional — optimizing for the best outcome would introduce inconsistency and bias between employees with identical raw scores, since two employees with the same performance could receive different MACH levels based purely on where the algorithm placed the exception. First-eligible sequential application ensures every employee is treated identically.

### Exact Threshold Performance

If an employee's performance exactly meets the requirement (for example, 2 out of 2 when 2 is required), they are considered "Ready" and clear that level normally.

### Multiple Technology Areas

The one-time leniency exception is tracked separately for each technology area. An employee can use the exception once for SQL, once for Python, once for Cloud, and so on. The exception is not global across all technologies.

### Tied Scores at Threshold

When calculating 65% of the total skills, rounding is used to determine the requirement. For example:
- 2 skills x 0.65 = 1.3, rounds to 1 (employee needs 1 out of 2, or 50%)
- 3 skills x 0.65 = 1.95, rounds to 2 (employee needs 2 out of 3, or 67%)

This rounding is intentional and accepted as reasonable given the uneven distribution of skills across MACH levels.

---

## 8. Summary

### The Complete Logic in Simple Terms

1. **Calculate requirements**: For each MACH level, determine how many skills the employee needs to pass (65% of total, rounded)

2. **Check readiness**: Compare actual performance to the requirement for each level

3. **Identify near-misses**: Flag any level where the employee had 3+ skills and missed by exactly 1

4. **Evaluate sequentially**: Starting at MACH 1, check each level in order:
   - If ready, continue
   - If not ready but has a near-miss flag and hasn't used the exception yet, continue and mark exception as used
   - If not ready and no exception available, stop

5. **Assign final level**: The last level cleared becomes the final MACH assignment. If MACH 1 was not cleared, the result is "Not Ready"

### Key Principles

- **Cumulative**: Levels build on each other; you cannot skip
- **Consistent**: The same 65% standard applies to all levels
- **Transparent**: Every decision can be explained using the documented rules
- **Fair**: One near-miss exception provides flexibility without compromising standards
- **Honest**: "Not Ready" is a valid outcome when foundational skills are missing

---

## 9. Document History

| Version | Summary | Edited By |
|---------|---------|-----------|
| 1.0 | Initial framework documentation | Mandar Sawant |
| 1.1 | Fixed incorrect exception denial reason in Example 1 (was "missed by more than 1", corrected to "only 2 skills tested"); removed invalid "zero skills" edge case and replaced with sequential MACH guarantee; added Example 5 explicitly covering broken chain scenario where leniency flag exists at a higher level | Mandar Sawant |
| 1.2 | Added client-mandated note to Rule 1 (65% threshold); strengthened one-time leniency justification in Rule 2; added explicit anti-optimization note in Section 7 | Mandar Sawant |
| 1.3 | Corrected Example 5 data — MACH 3 Skills Passed updated to 0 (hard fail, missed by 2); MACH 4 Skills Passed updated to 2 (genuinely Not Ready, missed by 1, Leniency Flag correctly triggered) | Mandar Sawant |

- **Rules Status**: Rule 1, 2, and 3 finalized and locked
- **Next Review**: To be scheduled after initial implementation

---

**Created By:** Mandar Sawant