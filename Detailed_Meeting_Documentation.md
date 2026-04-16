# Detailed Meeting Documentation: Dashboard Issues and Process Analysis

## Executive Summary
This meeting, held on April 15, 2026, involved a deep dive into issues with the learner dashboard system for skill assessments at Mathco. The discussion revealed multiple systemic problems including data inconsistencies, logic ambiguities, manual processes, and scalability challenges. Key issues include mismatched skill scores vs. MAC (Mastery Assessment Criteria) levels, duplicate records, and inconsistent readiness calculations. The team emphasized the need for comprehensive documentation, process automation, and customer alignment to prevent system failure. Recommendations include transitioning to a database-driven approach, leveraging AI for automation, and treating this as a consulting project to redefine the problem and build a reliable solution.

## Meeting Overview
- **Date and Time**: April 15, 2026; approximately 2 hours and 41 minutes (from 00:00:07 to 02:41:12).
- **Location**: Virtual meeting with screen sharing.
- **Attendees**:
  - Amit Choudhary (Lead/Moderator)
  - Sunil Kumar (Presenter, testing lead)
  - Mansi/Enqurious 4 (Team member, involved in mappings)
  - Mandar (Logic expert, shared sheets)
  - Chethan (Data uploader, Metabase maintainer)
  - Divyanshi Sharan (Mentor, involved in calculations)
  - Soumyaranjan (Team member, logic contributor)
  - Rohit (Team member, sheet maintainer)
  - Others mentioned indirectly (e.g., Sameer, Jen for investigations; Pratik for collaboration).
- **Purpose**: Debug dashboard discrepancies, understand end-to-end process, identify root causes, and propose fixes to ensure customer trust.
- **Key Outcomes**: Identified 5+ major issues; mapped process flow; recommended automation and documentation; assigned ownership to Sunil for follow-up.

## Agenda and Process Overview
The meeting focused on understanding the dashboard's data flow and logic:
1. **Data Sources**: Google Sheets (skill scores, MAC mappings), Metabase for queries/dashboards, Analytics DB for raw data.
2. **Data Flow**: Sheets → Manual updates → Metabase uploads → Queries → Dashboard reports.
3. **Artifacts**: Skill score sheets, MAC qualification sheets, consolidated reports, readiness verdicts.
4. **Steps**: Manual tagging, pivot calculations, benchmark applications (e.g., 65% for MAC readiness), final MAC aggregation.
5. **Challenges**: Manual processes, multiple interpretations, data leakage, lack of standardization.

## Detailed Issues and Discussions
The team discussed several examples and root causes, highlighting inconsistencies in logic, data, and processes.

### 1. **Duplicate Records and Averaging Issues**
   - **Example**: Girish Kumar (ESS, Data Engineer) showed 81% skill score but "not ready" MAC status due to duplicate deployments (one zero-score, one valid).
   - **Root Cause**: Redeployments create duplicates; aggregation averages scores (e.g., 0 + 10 = 5, below 65% benchmark).
   - **Discussion**: Script takes first instance; pivots avoid duplicates in some sheets but not others. Need unique counts.
   - **Impact**: Misleading readiness; customer confusion.
   - **Proposed Fix**: Deduplication rules; investigate with Sameer/Jen.

### 2. **Score Mismatches and MAC Logic Ambiguities**
   - **Example**: Avinash Yadav (CP) at MAC3 with 45% skill score; Shivam (ESS) at MAC1 with 80% score but "not ready".
   - **Root Cause**: Skill scores (sum of individual scores) vs. MAC readiness (aggregated 0/1 flags at 65% threshold). Different interpretations: some penalize lower MACs, others weight higher ones.
   - **Discussion**: MAC levels are hierarchical (1→2→3→4→5); benchmarks (7/10 for qualification, 65% for readiness). Under-representation (e.g., few skills in MAC1) causes uneven weighting. Subjective fairness debates (e.g., penalize for missing basics vs. reward advanced skills).
   - **Impact**: Conflicting signals (high score but low MAC); customer questions validity.
   - **Proposed Fix**: Standardize logic (e.g., weighted averages, hierarchical penalties); align with customer expectations.

### 3. **Inconsistent Readiness Calculations**
   - **Example**: Multiple cases where "not ready" despite high scores; NA vs. "not ready" distinctions.
   - **Root Cause**: Varying interpretations of MAC aggregation (e.g., Mandar's logic vs. Divyanshi's). No single source of truth; manual overrides.
   - **Discussion**: Readiness = final MAC level + skill score correlation. Issues with under-represented MACs (e.g., 2 skills in MAC1). Need common denominator for roles (e.g., MAC5 max).
   - **Impact**: Unreliable verdicts; affects hiring decisions.
   - **Proposed Fix**: Document all logics; choose one (e.g., highest MAC if qualified, penalize gaps).

### 4. **Data Leakage and Upload Issues**
   - **Example**: CP data uploads caused missing fields; ESS data showed after fixes.
   - **Root Cause**: Manual CSV uploads; changes in mappings (e.g., Cloud as single skill vs. GCP/Azure/AWS separately) without version control.
   - **Discussion**: Metabase consumes CSVs; no direct DB integration. Apps Script fetches from Analytics DB but manual sheets override.
   - **Impact**: Stale/incomplete data; testing failures.
   - **Proposed Fix**: Automate uploads; use DB procedures for mappings.

### 5. **Process and Tooling Challenges**
   - **Example**: Multiple sheets (20+ tabs), copy-paste errors, no primary keys.
   - **Root Cause**: Sheet-based approach; manual mappings (skill tags → MAC → groupings like SQL/Python).
   - **Discussion**: Mappings from customer sheets (e.g., Mathco-provided MAC levels); role-specific variations (ESS vs. CP). No audit trails; cognitive overload.
   - **Impact**: Errors in scaling; hard to maintain.
   - **Proposed Fix**: Database migration; AI automation; internal app for forms.

### 6. **Customer Alignment and Logic Validation**
   - **Discussion**: Logic not fully discussed/agreed with customer; assumptions on hierarchies and benchmarks.
   - **Impact**: Potential misalignment; customer may question fairness.
   - **Proposed Fix**: Re-discuss with customer; document agreements; use AI for evidence collection.

## Process Flow Mapping
Based on discussions, the end-to-end process:
1. **Input**: Customer-provided MAC mappings (sheets with skills/descriptions).
2. **Mapping**: Team maps descriptions to internal skill tags; assigns MAC levels (1-5).
3. **Assessment**: Mentors evaluate learners; scores (0-10) per skill.
4. **Calculation**: Aggregate to MAC readiness (0/1 flags); apply 65% threshold; compute skill scores (weighted sums).
5. **Aggregation**: Determine final MAC (e.g., highest qualified); apply readiness logic.
6. **Output**: Dashboard reports (expected MAC, actual MAC, skill score, readiness).
7. **Tools**: Google Sheets (manual), Apps Script (automation), Metabase (queries), Analytics DB (storage).

**Pain Points**: Manual steps (4-6); multiple logics; data movement causes leaks.

## Action Items
1. **Investigate Duplicates**: Sunil/Sameer/Jen to check deployments; implement deduplication.
2. **Standardize Logic**: Document all MAC/readiness logics; choose/align one; test with examples.
3. **Fix Data Flow**: Chethan to automate uploads; integrate DB directly.
4. **Document Everything**: Sunil to create detailed process docs (flowcharts, logics, mappings); version on GitHub.
5. **Customer Discussion**: Prepare evidence (examples, videos); discuss hierarchies/benchmarks.
6. **Automate**: Use AI/Claude to build DB schema, procedures, and internal app.
7. **Testing**: Iterate quickly; monitor for new issues.

**Timeline**: Complete documentation today; fixes in 1 week; customer alignment ASAP.

## Recommendations
- **Short-Term**: Fix duplicates/logics manually; improve docs.
- **Medium-Term**: Migrate to database (e.g., Supabase); add forms for data entry.
- **Long-Term**: Build productized solution with AI; ensure auditability.
- **Principles**: Own the problem; involve customer; minimize data movement; leverage AI for speed.
- **Risks**: Delays could erode trust; inconsistent logic affects decisions.

## Appendices
### Key Examples
- **Girish Kumar**: Duplicate issue; 81% score but not ready.
- **Avinash Yadav**: MAC3 vs. 45% score; under-represented MACs.
- **Shivam**: MAC1 with 80% but not ready; logic debate.
- **Aishi Ghosh**: MAC2 ready, MAC1 not; readiness calculation.

### References
- Full transcript: `GMT20260415-123647_Recording.transcript.vtt`.
- Sheets discussed: Skill score sheets, MAC qualification, consolidated reports.
- Tools: Google Sheets, Metabase, Analytics DB.

This documentation is comprehensive, derived from the transcript. For updates, collaborate on GitHub.</content>
<parameter name="filePath">c:\Users\lenovo\OneDrive\Desktop\Mathco\Detailed_Meeting_Documentation.md