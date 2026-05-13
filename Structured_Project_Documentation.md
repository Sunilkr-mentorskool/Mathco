# Structured Project Documentation: Mathco Dashboard System

## Document Purpose
This document provides a comprehensive overview of the learner dashboard project, including project goals, end-to-end process flow, calculation methodologies, data mappings, and current issues impacting customer experience.

---

## 1. PROJECT GOALS AND OBJECTIVES

### Primary Objectives
1. **Create a Reliable Skill Assessment Platform**: Build a dashboard that accurately captures, evaluates, and reports learner skill proficiency across multiple MAC levels.
2. **Standardize Skill-to-Role Mapping**: Map internal skill tags to customer-facing categories (e.g., SQL, Python, Machine Learning) for clear reporting.
3. **Generate Actionable Readiness Reports**: Provide customers with definitive readiness verdicts (Ready/Not Ready) to support hiring and talent allocation decisions.
4. **Support Multiple Roles and Service Lines**: Accommodate varying skill requirements across different organizational units (L1, L2, L3, CP, ESS, DAC, etc.).

### Secondary Objectives
- Automate data capture and processing to reduce manual effort.
- Ensure data integrity and consistency across all stages.
- Enable easy customization for new roles and skill sets.
- Provide audit trails and transparency for all decisions.

### Success Metrics
- 100% data accuracy in duplicate detection and removal.
- Consistent logic across all MAC readiness calculations.
- Zero customer complaints about conflicting scores/verdicts.
- 95%+ test coverage for all assessment scenarios.

---

## 2. END-TO-END PROCESS FLOW

### High-Level Overview
```
INPUT → MAPPING → ASSESSMENT → CALCULATION → AGGREGATION → OUTPUT
```

### Detailed Process Steps

#### **Stage 1: INPUT - Customer Requirements**
- **Source**: Customer (Mathco) provides skill mapping sheets with MAC levels (1-5) and skill descriptions.
- **Deliverable**: Master skill definitions per role/service line (e.g., "SQL for DSE at MAC2", "Join operations at MAC2").
- **Tools**: Google Sheets (customer-provided).
- **Issues**:
  - Customer provides messy, unstructured sheets.
  - No version control; manual updates without audit trails.

#### **Stage 2: MAPPING - Internal Tag Assignment**
- **Process**: Team maps customer-provided skill descriptions to internal skill tags (CTE, Filter, Group By Aggregate, Join, Window, etc.).
- **Output**: Mapping table linking:
  - Customer Skill Category (SQL, Python, Power BI, ML, EDA+Stat) → Internal Skill Tags → MAC Levels.
- **Tools**: Google Sheets, manual curation.
- **Current State**: Mapping exists in sheets; no formalized database.
- **Issues**:
  - Manual mapping prone to errors.
  - Different roles (CP vs. ESS) have different mappings for same skills (e.g., Cloud vs. GCP/Azure/AWS).
  - No centralized source of truth.

#### **Stage 3: ASSESSMENT - Learner Evaluation**
- **Process**: Mentors evaluate learners against skill tags; assign scores (0-10 per skill).
- **Scoring Rubric**:
  - 0-6: Failed/Not Qualified.
  - 7-10: Qualified.
- **Output**: Skill Score Sheet with rows: Learner ID | Skill Tag | MAC Level | Score (0-10).
- **Tools**: Google Sheets (mentor-populated), Apps Script (data aggregation).
- **Current State**: Mentors manually enter scores per learner per skill.
- **Issues**:
  - Manual entry delays and errors.
  - Duplicate entries (when learner redeployed for same assessment).
  - Inconsistent evaluation criteria.

#### **Stage 4: CALCULATION - Proficiency & MAC Readiness**
- **Process A - Proficiency Flag**:
  - For each skill: IF Score > 6 THEN Flag = 1 ELSE Flag = 0.
  - Aggregate by MAC level and learner.
  - Calculate proficiency % per MAC: (Count of Flags = 1) / (Total Skills in MAC) * 100%.

- **Process B - MAC Readiness (65% Benchmark)**:
  - For each MAC level: IF Proficiency % ≥ 65% THEN Ready ELSE Not Ready.
  - Example: MAC1 has 3 skills; learner qualified in 2 → 67% → Ready.

- **Process C - Skill Score (Overall %)**:
  - Sum all scores across all skills, all MACs.
  - Divide by (Total Skills × 10).
  - Result: Overall skill score percentage (e.g., 73%).

- **Tools**: Google Sheets (pivot tables, formulas), Apps Script (automation).
- **Issues**:
  - Under-representation of skills (e.g., MAC1 with only 2 skills → only 0%, 50%, 100% possible).
  - Skill scores and MAC readiness don't correlate (e.g., 80% score but "Not Ready").
  - Multiple logic interpretations (penalize gaps vs. reward advanced skills).

#### **Stage 5: AGGREGATION - Final MAC & Readiness Verdict**
- **Process**:
  - Aggregate individual MAC readiness to final MAC level using complex logic:
    - Hard Rule: Person is HIGHEST MAC if they qualify all lower MACs.
    - Exception Rule: If MAC gaps exist, assign lowest qualified consecutive MAC (e.g., MAC1 + MAC2 but not MAC3 = MAC2).
    - Fallback: If not qualified for MAC1, mark "Not Ready".
  - Special Cases: Over-qualification (e.g., qualified for MAC5 but only expected MAC3) → Cap at expected or allow override.

- **Tools**: Apps Script, Metabase queries.
- **Issues**:
  - Logic not standardized; multiple versions in use.
  - Subjective fairness criteria; no consensus.
  - Inconsistent application across learners.

#### **Stage 6: OUTPUT - Dashboard Reports**
- **Report Sections**:
  1. **Learner Profile**: Name, Role, Service Line.
  2. **Skill Mapping**: Expected MAC vs. Actual MAC per skill category (SQL, Python, etc.).
  3. **Readiness Verdict**: Ready/Not Ready status.
  4. **Skill Score**: Overall percentage.
  5. **Feedback**: Qualitative mentor observations.

- **Tools**: Metabase (queries), dashboard UI.
- **Audience**: HR, managers, learners.
- **Issues**:
  - Conflicting signals (high skill score + "Not Ready").
  - Unclear verdicts (NA vs. "Not Ready").
  - No audit trail for how verdict was calculated.

### Process Flow Diagram
```
Customer Sheets
     ↓
Mapping Table (Internal Tags ↔ MAC Levels)
     ↓
Mentor Assessment (Scores 0-10)
     ↓
Proficiency Flags (0/1 per Skill)
     ↓
MAC Readiness (% vs. 65% Benchmark)
     ↓
Skill Score (Overall %)
     ↓
Final MAC + Verdict Logic
     ↓
Dashboard Report → Customer
```

---

## 3. CALCULATION METHODOLOGIES

### Calculation 1: Proficiency Flag (per Skill)
```
IF Score >= 7 THEN Flag = 1 ELSE Flag = 0
```
**Example**:
- CTE scored 8 → Flag = 1 (Qualified)
- Filter scored 5 → Flag = 0 (Not Qualified)

### Calculation 2: MAC Readiness Percentage (per MAC Level)
```
MAC Readiness % = (Sum of Flags in MAC) / (Total Skills in MAC) * 100%
```
**Example (MAC1 with 3 skills)**:
- Skills: CTE (Flag=1), Filter (Flag=1), Group By (Flag=0)
- Readiness = (1+1+0) / 3 * 100% = 66.7%

### Calculation 3: MAC Status (per MAC Level)
```
IF MAC Readiness % >= 65% THEN Status = "Ready" ELSE Status = "Not Ready"
```
**Example**: 66.7% ≥ 65% → Status = "Ready"

### Calculation 4: Overall Skill Score
```
Skill Score % = (Sum of All Scores Across All Skills) / (Total Skills * 10) * 100%
```
**Example (10 skills total)**:
- Scores: [8, 7, 9, 6, 5, 8, 7, 0, 9, 6]
- Sum = 65, Total Possible = 100
- Skill Score = 65 / 100 * 100% = 65%

### Calculation 5: Final MAC Level (Complex Aggregation)
```
LOGIC A (Current - Contested):
- IF learner qualified for ALL MACs (1 through N) THEN Final MAC = N
- ELSE IF learner qualified for some MACs with gaps THEN:
  - Assign highest consecutive MAC (penalize gaps)
  - OR assign highest qualified MAC (reward advanced skills)
- ELSE IF learner not qualified for MAC1 THEN Final MAC = "Not Ready"

LOGIC B (Proposed - Under Discussion):
- Compare actual MAC readiness profile against expected MAC
- IF actual < expected THEN mark "Not Ready"
- IF actual >= expected THEN mark "Ready"
- Consider weighted scores by MAC representation
```

**Issues with Current Calculation**:
- Under-representation bias: MAC with 2 skills can only have 0%, 50%, 100% readiness.
- Conflicting signals: High overall skill score (80%) but low MAC readiness (50%).
- No consensus: Different team members apply different logic.

---

## 4. DATA SOURCE TO DESTINATION MAPPING

### Complete Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│ SOURCE: Customer Sheets (Google Sheets)                             │
│ - MAC Definitions (Skills 1-5, MAC Levels)                          │
│ - Role Requirements (ESS, DAC, CP, L1, L2, L3)                      │
└────────────────────┬────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────────────┐
│ PROCESSING 1: Manual Mapping                                        │
│ - Team creates internal mapping table                               │
│ - Skill Description → Internal Tag → MAC Level                      │
│ - LOCATION: Google Sheets (multiple tabs)                           │
└────────────────────┬────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────────────┐
│ SOURCE: Mentor Assessments (Google Sheets)                          │
│ - Learner ID | Skill Tag | Score (0-10)                            │
│ - LOCATION: Google Sheets (Skills Course sheet)                     │
│ - FREQUENCY: Updated weekly per evaluation                          │
└────────────────────┬────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────────────┐
│ STORAGE: Analytics Database                                         │
│ - Raw scores persisted from Apps Script                             │
│ - LOCATION: Analytics DB (main data repository)                     │
└────────────────────┬────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────────────┐
│ PROCESSING 2: Apps Script Automation                                │
│ - Calculate proficiency flags (0/1)                                 │
│ - Aggregate to MAC readiness percentages                            │
│ - Apply 65% benchmark                                               │
│ - OUTPUT: MAC Readiness Sheet (Google Sheets)                       │
└────────────────────┬────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────────────┐
│ PROCESSING 3: Manual Aggregation & Logic                            │
│ - Determine final MAC level (complex logic)                         │
│ - Create readiness verdict (Ready/Not Ready)                        │
│ - LOCATION: Google Sheets (MAC Qualification sheet)                 │
│ - NOTE: Manual updates by team; prone to inconsistencies            │
└────────────────────┬────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────────────┐
│ DATA UPLOAD: CSV to Metabase                                        │
│ - Chetan manually downloads consolidated sheet                      │
│ - Uploads as CSV to Metabase database                               │
│ - FREQUENCY: Weekly uploads                                         │
│ - RISK: Data stale; duplicates; format issues                       │
└────────────────────┬────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────────────┐
│ QUERIES: Metabase SQL Queries                                       │
│ - Fetch filtered data per learner, skill, role                      │
│ - Join mappings for customer-facing categories                      │
│ - Apply additional formatting/filters                               │
└────────────────────┬────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────────────┐
│ OUTPUT: Dashboard Reports                                           │
│ - Learner profiles with skill scores, MAC levels, readiness         │
│ - Exported as reports for HR/Managers                               │
│ - LOCATION: Mathco Portal (customer-facing)                         │
└─────────────────────────────────────────────────────────────────────┘
```

### Data Mapping Details

| Source | Data Type | Transformation | Destination | Issues |
|--------|-----------|-----------------|-------------|--------|
| Customer Sheets | Skill Definitions (Text) | Manual mapping to internal tags | Google Sheets (Mapping Table) | Manual; no versioning |
| Mentor Assessments | Scores (0-10) | Proficiency flags (0/1) | Analytics DB | Duplicates possible |
| Analytics DB | Raw Scores | Apps Script aggregation | MAC Readiness Sheet | Incomplete; logic gaps |
| MAC Readiness | Percentages | Manual final MAC logic | MAC Qualification Sheet | Inconsistent; subjective |
| Google Sheets | Consolidated Data | CSV export | Metabase Upload | Data leakage; staleness |
| Metabase | Stored Data | SQL queries + filtering | Dashboard | Report accuracy depends on upstream logic |
| Dashboard | Report Data | Display + export | Customer Portal | Conflicting signals (score vs. verdict) |

### Current Bottlenecks
1. **Google Sheets**: No primary keys, duplicates not prevented.
2. **Manual Mapping**: Prone to errors; changes not tracked.
3. **CSV Uploads**: Weekly cadence causes staleness; format mismatches.
4. **Logic in Sheets**: Calculations scattered across tabs; no single source of truth.
5. **No Automation**: Copy-paste, manual overrides, no version control.

---

## 5. CURRENT ISSUES AND IMPACT ON CUSTOMER EXPERIENCE

### Issue 1: Duplicate Records and Averaging Errors
**Description**: Redeployments create duplicate entries; aggregation averages scores, yielding incorrect results.

**Example**:
- Girish Kumar assessed twice: First attempt (scores 0), Second attempt (scores 8).
- Averaging: (0 + 8) / 2 = 4 → Below 65% benchmark → "Not Ready".
- Reality: Second (valid) assessment shows "Ready".

**Customer Impact**: Learner marked "not ready" despite strong performance; decision-maker confused; hiring frozen.

**Root Cause**: No deduplication logic; first instance always picked; duplicates from system redeployments.

---

### Issue 2: Mismatched Skill Scores and MAC Readiness
**Description**: Overall skill score (e.g., 80%) doesn't correlate with MAC readiness (e.g., "Not Ready").

**Example**:
- Shivam: SQL assessment
  - MAC1 (2 skills): 50% → Not Ready
  - MAC2 (multiple skills): 100% → Ready
  - Overall Skill Score: 80%
  - Final Verdict: "Not Ready" (because lower MAC missed)
  - Customer sees: 80% score but "not ready" → Contradiction.

**Customer Impact**: Hiring managers question system reliability; Shivam may be rejected despite strong overall performance.

**Root Cause**: Skill score = average of all skills; MAC readiness = hierarchical logic. Different calculations; no explicit correlation.

---

### Issue 3: Inconsistent Readiness Logic
**Description**: Multiple interpretations of final MAC assignment; no standardized rule.

**Example**:
- Avinash Yadav (Python, CP role):
  - Expected MAC: 3
  - Actual: MAC1 ✓, MAC2 ✗, MAC3 ✓
  - Logic A (Penalize Gaps): "Only MAC1 ready" → Not Ready.
  - Logic B (Reward Advanced): "MAC3 ready" → Ready.
  - Current: Mixed application; inconsistent verdicts.

**Customer Impact**: Same learner gets different verdicts depending on who evaluates; fairness questioned.

**Root Cause**: No agreed-upon logic; subjective interpretation; multiple versions in use.

---

### Issue 4: Under-Representation Bias
**Description**: MAC levels with few skills can only achieve discrete percentages (0%, 50%, 100%), causing unfair thresholds.

**Example**:
- MAC1 with 2 skills: Passing 0 → 0%, Passing 1 → 50%, Passing 2 → 100%.
- 65% benchmark can only be met at 100%.
- MAC3 with 10 skills: 70% easily achievable (7 of 10 passed).

**Customer Impact**: Learners penalized for MAC1 gaps despite strong MAC2/MAC3; unfair weighting.

**Root Cause**: Uneven skill distribution by MAC; no normalization.

---

### Issue 5: Data Leakage and Upload Issues
**Description**: Manual CSV uploads introduce missing fields, stale data, and version mismatches.

**Example**:
- CP data added; Cloud skill split into GCP/Azure/AWS.
- ESS data missing MAC mappings after upload.
- Fix took hours; tested report broken; customer notified late.

**Customer Impact**: Reports unreliable; delays in reporting; customer loses confidence.

**Root Cause**: Manual process; no automated validation; no version control; weekly batches.

---

### Issue 6: No Audit Trail or Transparency
**Description**: How verdicts calculated is opaque; changes not tracked.

**Example**:
- Learner queries "Why not ready?" → No documented logic; team uncertain.
- Update made to mapping; previous assessments affected retroactively; no rollback.

**Customer Impact**: Learners and managers unable to trust verdicts; no explanation.

**Root Cause**: Logic in sheets; no versioning; calculations scattered.

---

### Issue 7: Scalability and Maintenance Challenges
**Description**: Sheet-based system; copy-paste; 20+ tabs open; manual overrides.

**Example**:
- Adding new role requires updating multiple sheets manually.
- Testing new logic: Open 10 sheets, copy data, verify, copy back.
- One error cascades across all reports.

**Customer Impact**: New roles/skills delayed; error risk high; fixes slow.

**Root Cause**: No database; no automation; cognitive overload.

---

### Cumulative Customer Experience Impact

| Dimension | Issue | Impact |
|-----------|-------|--------|
| **Trust** | Conflicting scores; unclear logic | Low confidence in verdicts; hesitation to act on reports |
| **Fairness** | Inconsistent logic; biases | Learners feel unfairly assessed; complaints to HR |
| **Reliability** | Duplicates; data leakage | Reports unpredictable; decisions frozen pending fix |
| **Transparency** | No audit trail | Unable to explain verdicts; frustration |
| **Scalability** | Manual processes | New roles/skills delayed; system strained |
| **Efficiency** | Weekly uploads; manual steps | Slow feedback loops; delayed hiring cycles |

---

## 6. SUMMARY AND RECOMMENDATIONS

### What's Working Well
- Assessment methodology (scoring rubric, proficiency flags) is sound.
- Mapping approach (skill tags → MAC levels) is reasonable.
- Dashboard UI is user-friendly.

### What's Broken
- **Duplicates**: No deduplication; averaging errors.
- **Logic**: Multiple interpretations; no consensus; subjective fairness.
- **Data Flow**: Manual, error-prone, no validation.
- **Scalability**: Sheet-based, not sustainable.

### Recommended Actions
1. **Short-Term (Week 1)**:
   - Fix duplicates: Deduplication rules; remove old entries.
   - Standardize logic: Document and agree on final MAC calculation.
   - Align skill score: Clarify correlation with MAC readiness.

2. **Medium-Term (Week 2-4)**:
   - Automate uploads: DB integration; remove manual CSV steps.
   - Build internal app: Forms for data entry; validation rules.
   - Document everything: Version on GitHub; audit trails.

3. **Long-Term (Ongoing)**:
   - Migrate to database: Replace Sheets with Postgres/Supabase.
   - AI automation: Use Claude to refine logic; auto-detect anomalies.
   - Product roadmap: Buildtoward productized solution.

### Next Steps
- Share this document with stakeholders for alignment.
- Schedule customer discussion on logic assumptions.
- Assign Sunil as lead; begin documentation and fixes.

---

## Appendix: Examples from Transcript

### Example 1: Girish Kumar (Duplicate Record Issue)
- **Role**: Data Engineer, ESS
- **Assessment**: Spark module
- **Issue**: Duplicate entries (one with 0 score, one with valid score)
- **Result**: Average = 5 → "Not Ready" despite second attempt success
- **Customer Complaint**: Why not ready when scores are good?

### Example 2: Avinash Yadav (Score vs. Verdict Mismatch)
- **Role**: Custom Product, Python
- **Assessment**: Python
- **Expected MAC**: 3
- **Actual MAC**: 3 (on paper)
- **Skill Score**: 45% (despite MAC3)
- **Issue**: High MAC level but low skill percentage → Confusing
- **Customer Complaint**: How is skill score so low if MAC3 qualified?

### Example 3: Shivam (Hierarchical Logic Debate)
- **Role**: DSE (Data Science Engineering), SQL
- **Assessment**: SQL
- **Expected MAC**: 2
- **Actual**: MAC1 (only 50% in MAC1, 100% in MAC2)
- **Skill Score**: 80%
- **Verdict**: "Not Ready" (penalize MAC1 gap)
- **Issue**: 80% score but "not ready" → Contradiction
- **Customer Complaint**: Why not ready with 80% score?

---

## Document Version
- **Version**: 1.0
- **Date**: April 16, 2026
- **Prepared By**: Amit Choudhary, Sunil Kumar, Team
- **Source**: Meeting Transcript (April 15, 2026)

---

*This document is intended for internal and management review. It provides the foundational context for aligning on project goals, process improvements, and customer communication strategy.*
