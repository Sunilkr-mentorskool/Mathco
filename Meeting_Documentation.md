# Meeting Documentation: Dashboard Issues Discussion

## Meeting Details
- **Date**: April 15, 2026 (based on transcript filename: GMT20260415-123647_Recording.transcript.vtt)
- **Time**: Approximately 20 minutes (transcript covers 00:00:07 to 00:20:37, with gaps)
- **Location**: Virtual meeting (screen sharing mentioned)
- **Purpose**: Review and debug issues in the learner dashboard system, focusing on skill scores, MAC levels, data discrepancies, and process flow to ensure customer confidence in the platform.

## Attendees
- Amit Choudhary (Lead/Moderator)
- Sunil Kumar (Presenter, sharing screens and examples)
- Mansi (referred to as "Enqurious 4" in transcript, likely a transcription error)
- Chethan
- Sayli
- Mandar
- Others mentioned indirectly (e.g., Sameer, Jen for follow-up)

## Agenda and Process Overview
The discussion aimed to clarify the end-to-end process of the dashboard system and identify glitches. Key components of the process:
1. **Data Sources**: Google Sheets for skills (separate for L1/L2/L3/CP roles), MAC sheets, and ETL refreshes.
2. **Data Flow**: Data moves to analytics DB, queried via Metabase to generate charts/tables, parameterized and exposed via the platform.
3. **Artifacts**: Reports refreshed periodically, consumed by customers.
4. **Steps**: Manual and automated, involving mentors, DB updates, and SQL queries.

The goal is to implement this in the product while maintaining reliability.

## Key Discussion Points
- **Communication Guidelines**: Speak in English for better transcription; structure thoughts clearly; no blame game—focus on fixes.
- **Dashboard Testing**: Team has been populating MAC and skill codes into the dashboard and identifying ambiguities, especially with skill codes.
- **Data Sources Confirmation**: Using skills course sheets and MAC sheets; skills sheets are role-specific (L1/L2/L3/CP), while MAC is not directly the candidate's score but a skill categorization (e.g., "Managed Table" as MAC2 level).

## Issues Identified
1. **Duplicate Records and Averaging Problems**:
   - Example: Girish Kumar (Data Engineer from ESS/Cloud Inc.) shows 81% skill score but marked as "not ready" in MAC.
   - Root Cause: Duplicate entries from multiple deployments (e.g., one incomplete with zeros, one complete). Aggregation averages scores, leading to disqualification (e.g., 0 + 10 = 5, below benchmark).
   - Impact: Script takes the first instance; duplicates cause conflicts. Pivot tables avoid this in some sheets but not others.
   - Investigation Needed: Check with Sameer/Jen if Girish was in multiple deployments.

2. **Score Mismatches and Logic Conflicts**:
   - Example: Avinash Yadav (from Custom Product) has MAC3 level in Python but only 45% skill score.
   - Root Cause: Skill scores calculated from proficiency (e.g., 8/10 in some skills, 0 in others), aggregated to low percentages despite MAC achievement. Benchmark is 65%+ for qualification, but MAC is a separate categorization.
   - Impact: Confuses management—high MAC but low score signals inconsistency. Scores are in increments of 10 (e.g., 40%, 50%).

3. **Correlation Between Skill Scores and MAC Readiness**:
   - High skill scores should correlate with qualification, but current logic shows conflicts (e.g., 81% but not ready).
   - MAC levels are benchmarks (e.g., 7/10 qualifies), but aggregation may not handle duplicates properly.

4. **General Data Integrity**:
   - Duplicates from redeployments (incomplete attempts recorded).
   - Need for unique skill counting (e.g., 41 skills per participant).

## Action Items
- **Investigate Duplicates**: Sunil to check with Sameer/Jen on multiple deployments for affected learners (e.g., Girish Kumar).
- **Validate Aggregation Logic**: Ensure scripts handle duplicates correctly; use unique counts for skills.
- **Fix Score Calculations**: Align skill scores with MAC levels; review benchmarks and pivots.
- **Test and Iterate**: Continue populating dashboard with test data; monitor for more issues.
- **Documentation**: Create clear process docs to prevent future leaks.

## Conclusions
- Core issue is data duplication leading to incorrect aggregations and conflicting signals.
- Process is sound but needs refinement in handling edge cases (e.g., redeployments).
- Team to follow up on investigations; aim for a robust system to build customer confidence.
- Emphasize structured communication and no-blame approach for effective debugging.

## Notes
- Transcript includes informal speech, interruptions, and potential transcription errors (e.g., names like "Grish" for "Girish", "Enqurious 4" for "Mansi").
- Full transcript available in `GMT20260415-123647_Recording.transcript.vtt` for reference.
- This documentation is derived from the transcript; verify with participants for accuracy.</content>
<parameter name="filePath">c:\Users\lenovo\OneDrive\Desktop\Mathco\Meeting_Documentation.md