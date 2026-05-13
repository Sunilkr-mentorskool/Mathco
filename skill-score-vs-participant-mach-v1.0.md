# Skill Score % vs Participant MACH — Understanding the Gap

**Document Version:** 1.0

**Author:** Mandar Sawant

**Audience:** Enqurious Team, MathCo L&D Team

---

## 1. Background

As part of the MACH assessment initiative, the employee dashboard displays three key data points per technology area:

- **Role MACH**: The expected MACH level for the employee's job role in that technology area — essentially the benchmark the employee is being measured against
- **Skill Score %**: The employee's overall percentage score across all skills tested in a technology area
- **Participant MACH**: The final MACH level assigned to the employee based on their cumulative performance across MACH levels

These metrics exist side by side on the dashboard and together provide a picture of an employee's capability relative to their role requirements. However, Skill Score % and Participant MACH are fundamentally different in what they measure and how they are calculated — and conflating them has been a source of confusion when interpreting assessment results.

---

## 2. The Problem

A recurring question from the L&D team has been:

> *"How is this employee not at MACH 4 when they scored 83% in Python?"*

To understand why this happens, consider the following scenario. The Role MACH for Python is MACH 4, which means the assessment tests skills across all four MACH levels. Python had **14 skills** tested in total, distributed as follows:

| MACH Level | Number of Skills |
|------------|-----------------|
| MACH 1 | 4 |
| MACH 2 | 7 |
| MACH 3 | 2 |
| MACH 4 | 1 |

Here is how the employee performed across each MACH level:

| MACH | Total No. of Skills | Total Proficient Skills | Proficiency Required | MACH Readiness | Leniency Flag |
|------|---------------------|------------------------|---------------------|----------------|---------------|
| MACH 1 | 4 | 4 | 3 | Ready | 0 |
| MACH 2 | 7 | 6 | 5 | Ready | 0 |
| MACH 3 | 2 | 0 | 1 | Not Ready | 0 |
| MACH 4 | 1 | 1 | 1 | Ready | 0 |

The employee performed strongly at MACH 1 and MACH 2 — clearing both levels comfortably. However, they did not meet the readiness threshold at MACH 3, and the leniency exception does not apply here because there are only 2 skills at that level (minimum of 3 required for the exception).

Since MACH is evaluated sequentially, the evaluation stops at MACH 3. **MACH 4's Readiness is not considered** — not because the employee failed it, but because the hard stop at MACH 3 means higher levels are excluded from the assignment process entirely.

The result: **83% Skill Score, Participant MACH 2.**

This can appear contradictory when looking only at the Skill Score %. In reality, it is not — the two metrics are simply answering different questions.

---

## 3. Why These Two Metrics Are Different

| | Skill Score % | Participant MACH |
|---|---|---|
| **What it measures** | Overall performance across all skills in the assessment | Cumulative level of demonstrated proficiency |
| **How it is calculated** | Total Score Obtained / Total Score Available | Sequential evaluation across MACH levels using the 65% readiness rule |
| **What it answers** | "How did the employee perform in this assessment overall?" | "What is the highest MACH level the employee has cumulatively demonstrated?" |
| **Affected by skill distribution** | Yes — levels with more skills contribute more to the score | No — each level is evaluated independently |
| **Can be high while MACH is low** | Yes | — |

The core issue is that Skill Score % is a **volume-weighted average**. MACH levels with more skills naturally contribute more to it. MACH levels with fewer skills — even critical ones — can be drowned out. Participant MACH, on the other hand, treats each level as a gate that must be cleared regardless of how many skills it contains.

A high Skill Score % does not guarantee a higher Participant MACH. These are two separate measurements and should be interpreted independently.

---

## 4. Options Considered

### Option 1 — Replace Aggregate with Per MACH Level Score Breakdown

**Description:** Instead of a single Skill Score %, display a separate score for each MACH level. For example, Python MACH 1: 90%, MACH 2: 88%, MACH 3: 0%, MACH 4: 100%.

**Example:**

| Skill | Role MACH | Participant MACH | MACH 1 Score | MACH 2 Score | MACH 3 Score | MACH 4 Score |
|-------|-----------|-----------------|--------------|--------------|--------------|--------------|
| Python | MACH 4 | MACH 2 | 100% | 86% | 0% | 100% |

The L&D team can immediately see that the employee excelled at MACH 1, 2, and 4, but scored 0% at MACH 3 — which is precisely why their Participant MACH is capped at MACH 2.

**Pros:**
- Provides the most complete picture of employee performance at each level
- Removes reliance on a single aggregate number that may not tell the full story
- Makes the relationship between performance and MACH level immediately visible

**Cons:**
- The dashboard table becomes significantly wider and harder to read
- Employees assessed on 5 MACH levels would require 5 score columns per technology area
- May add complexity for stakeholders who prefer a simpler view
- Requires significant restructuring of the dashboard and underlying data model

---

### Option 2 — Keep Aggregate, Add Drill-Down Breakdown

**Description:** The main dashboard table retains the aggregate Skill Score %, but clicking on a row opens a detailed breakdown showing scores per MACH level.

**Example:**

Main view (what the L&D team sees by default):

| Skill | Role MACH | Participant MACH | Skill Score % |
|-------|-----------|-----------------|---------------|
| Python | MACH 4 | MACH 2 | 83% |

Drill-down view (accessed by clicking the row):

| MACH Level | Score |
|------------|-------|
| MACH 1 | 100% |
| MACH 2 | 86% |
| MACH 3 | 0% |
| MACH 4 | 100% |

The main view stays clean, while the detail is available for those who need it.

**Pros:**
- Keeps the main view clean and simple
- Provides depth for stakeholders who need it
- Does not require changes to the existing aggregate calculation

**Cons:**
- Metabase's native drill-down capability is limited and may not support this without custom development
- Stakeholders who do not navigate to the detail view may continue to rely solely on the aggregate score
- Adds complexity to dashboard maintenance

---

### Option 3 — Replace Aggregate with Score at Participant MACH Level Only

**Description:** Instead of averaging across all skills, the Skill Score % shows the employee's performance only on the skills at their achieved Participant MACH level.

**Example:**

The employee's Participant MACH is MACH 2. MACH 2 had 7 skills, and the employee passed 6 out of 7. The Skill Score % would therefore show:

| Skill | Role MACH | Participant MACH | Skill Score % |
|-------|-----------|-----------------|---------------|
| Python | MACH 4 | MACH 2 | 86% |

Instead of 83% across all 14 skills, the score now reflects only the 7 MACH 2 skills — directly tied to the level the employee achieved.

**Pros:**
- Single number, but directly relevant to the level the employee has been assessed at
- Directly answers "how did they perform at the level they achieved?"
- Removes the volume-weighting effect caused by uneven skill distribution

**Cons:**
- Loses visibility into performance at lower MACH levels
- An employee who performed strongly at MACH 1 and 2 but narrowly cleared MACH 3 would show a lower score, which may not fully represent their overall capability
- Changes the meaning of Skill Score % significantly and would require clear communication to all stakeholders

---

### Option 4 — Equal Weighted Score Per MACH Level

**Description:** Each MACH level contributes equally to the final Skill Score %, regardless of how many skills it contains. If there are 4 MACH levels, each contributes 25%.

**Example:**

| MACH Level | Skills | Score Obtained | Score Available | Level Score % | Equal Weight | Weighted Contribution |
|------------|--------|---------------|----------------|--------------|--------------|----------------------|
| MACH 1 | 4 | 40 | 40 | 100% | 25% | 25% |
| MACH 2 | 7 | 60 | 70 | 86% | 25% | 21.4% |
| MACH 3 | 2 | 0 | 20 | 0% | 25% | 0% |
| MACH 4 | 1 | 10 | 10 | 100% | 25% | 25% |

**Final Weighted Score: 71.4%** — compared to the raw aggregate of 83%.

While this reduces the dominance of MACH 2 (which has the most skills), it equally amplifies MACH 3 and MACH 4, which have far fewer skills. The single MACH 4 skill now carries the same weight as the 7 MACH 2 skills combined.

**Pros:**
- Reduces the dominance of MACH levels with a higher number of skills
- Each level gets equal representation in the final score

**Cons:**
- May overcorrect — a MACH level with only 2 skills would carry the same weight as one with 7, which may not reflect the actual volume of work assessed
- The resulting score becomes less intuitive to interpret and explain
- Introduces a different form of imbalance rather than resolving the underlying issue

---

### Option 5 — Keep Current Calculation, Add Context and Education *(Recommended)*

**Description:** The Skill Score % calculation remains unchanged as `Total Score Obtained / Total Score Available`. A contextual note or tooltip is added to the dashboard, and the L&D team is briefed on the distinction between Skill Score % and Participant MACH.

**Example:**

The dashboard continues to show:

| Skill | Role MACH | Participant MACH | Skill Score % |
|-------|-----------|-----------------|---------------|
| Python | MACH 4 | MACH 2 | 83% |

A tooltip on the Skill Score % column header clarifies:

> *"Skill Score % reflects overall performance across all skills tested. It is not a direct indicator of Participant MACH. Use Participant MACH for role alignment decisions."*

The L&D team now understands that 83% reflects strong performance at MACH 1 and 2 but does not indicate MACH level readiness. No data or dashboard changes are required.

**Pros:**
- No changes required to the data model, dashboard structure, or calculation logic
- The current Skill Score % accurately reflects overall performance in the assessment
- Addresses the root cause of the confusion, which is interpretation rather than calculation
- Sustainable — once the distinction is understood, the confusion does not recur
- Tooltip or note can be added in Metabase without significant development effort

**Cons:**
- Relies on stakeholders reading and internalising the contextual note
- Does not structurally prevent misinterpretation for stakeholders who are new to the dashboard
- Requires a one-time briefing effort

---

## 5. Recommended Solution

**Option 5 — Keep current calculation, add context and education.**

The Skill Score % accurately reflects the employee's overall performance in the assessment. The gap in understanding arises from using it to infer MACH level, which it was not designed to do. The recommended approach is to preserve the existing calculation and address the interpretation gap through contextual guidance on the dashboard and a clear briefing for the L&D team.

### Dashboard Tooltip / Contextual Note

The following note should be added to the Skill Score % column header on the dashboard:

> *"Skill Score % reflects the employee's overall performance across all skills tested in this technology area. It is not a direct indicator of Participant MACH level. Participant MACH is determined by sequential readiness at each MACH level and can differ from overall score due to uneven skill distribution across levels."*

### L&D Team Briefing Points

When briefing the L&D team, the following points should be communicated clearly:

1. **Skill Score % and Participant MACH answer different questions.** Skill Score % tells you how the employee performed overall. Participant MACH tells you what level they have cumulatively demonstrated.

2. **A high Skill Score % does not guarantee a high Participant MACH.** If a critical MACH level has very few skills, strong performance on other levels can inflate the aggregate score without changing the MACH outcome.

3. **Use Participant MACH for role alignment decisions.** When assessing whether an employee meets the requirements of their current role, Participant MACH is the relevant metric — not Skill Score %.

4. **Use Skill Score % for overall performance context.** Skill Score % is useful for understanding the employee's general engagement and effort across the assessment, not for determining level readiness.

---

## 6. What This Does Not Fix

It is important to set clear expectations. Option 5 addresses the confusion around interpretation but does not resolve the following:

- **Uneven skill distribution across MACH levels** remains a structural characteristic of the current assessment design. A technology area with 2 skills at MACH 3 and 7 skills at MACH 2 will naturally produce scenarios where Skill Score % and Participant MACH diverge. This is expected behaviour, not an error.

- **New stakeholders** who have not been briefed may encounter the same questions when they first review the dashboard. The tooltip provides helpful context, but an onboarding conversation remains valuable for anyone new to the assessment framework.

- **The distinction between overall performance and cumulative level readiness** is inherent to the MACH framework. These will always be two different things, and no single metric can fully capture both simultaneously.

The long-term opportunity to reduce this divergence is to work towards a more consistent number of skills per MACH level across technology areas during assessment design. However, that is a separate initiative and outside the scope of this document.

---

## 7. Document History

- **Version 1.0**: Initial draft covering problem statement, options considered, and recommended solution
