# Mathco MAC Report System — Technical Specification
**Version**: 1.0 | **Date**: April 16, 2026 | **Prepared by**: Enqurious Internal Team

---

## 1. Overview

### 1.1 Problem Statement

The current Mathco dashboard system relies on Google Sheets, Apps Script, and manual CSV uploads to Metabase. This creates duplicate records, inconsistent MAC readiness logic, no audit trail, and unreliable reports for a paying enterprise client.

### 1.2 Proposed Solution

Replace the Sheets-based pipeline with a **Supabase Postgres data mart** that:

- Stores master mapping data (skill → MAC level, skill → category) with access control
- Syncs raw learner scores from the Enqurious DB via a Python ETL script
- Runs all MAC readiness calculations as Postgres functions and materialized views
- Exposes clean reporting tables directly to Metabase via Postgres connection

No manual CSVs. No Apps Script. No sheet logic.

### 1.3 System Architecture

```
Enqurious DB (PostgreSQL)
    └── learner scores, skill tags
          │
          ▼ (Python ETL — scheduled or on-demand)
Supabase PostgreSQL (Data Mart)
    ├── master.skill_mac_mapping       ← who can edit: admin only
    ├── master.skill_category_mapping  ← who can edit: admin only
    ├── raw.learner_scores             ← synced from Enqurious, deduped
    ├── functions & procedures         ← all MAC logic lives here
    ├── views.mac_readiness            ← computed per learner per MAC
    └── reporting.final_verdicts       ← clean table for Metabase
          │
          ▼ (Direct Postgres connection)
Metabase
    └── Dashboards & Reports → Mathco Portal
```

---

## 2. Supabase Database Schema

### 2.1 Schemas

Three logical schemas separate concerns cleanly:

| Schema | Purpose |
|--------|---------|
| `master` | Reference/mapping tables. Admin-editable only. |
| `raw` | Scores synced from Enqurious. Written by ETL only. |
| `reporting` | Aggregated, calculated tables consumed by Metabase. |

---

### 2.2 Master Tables

#### `master.skill_mac_mapping`

Maps each internal skill tag to a MAC level, scoped by role and client.

```sql
CREATE TABLE master.skill_mac_mapping (
  id              SERIAL PRIMARY KEY,
  client          TEXT NOT NULL DEFAULT 'mathco',
  role            TEXT NOT NULL,           -- e.g. DSE, CP, ESS, DAC
  skill_tag       TEXT NOT NULL,           -- e.g. CTE, Join, Window
  mac_level       INTEGER NOT NULL CHECK (mac_level BETWEEN 1 AND 5),
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_by      TEXT,                    -- email of admin who last edited
  UNIQUE (client, role, skill_tag)
);
```

**Sample Data**:

| client | role | skill_tag | mac_level |
|--------|------|-----------|-----------|
| mathco | DSE | CTE | 1 |
| mathco | DSE | Filter | 1 |
| mathco | DSE | Group By Aggregate | 1 |
| mathco | DSE | Join | 2 |
| mathco | DSE | Window | 3 |
| mathco | CP | GCP | 2 |
| mathco | CP | Azure | 2 |
| mathco | ESS | Pandas | 2 |

---

#### `master.skill_category_mapping`

Maps each internal skill tag to a customer-facing skill category.

```sql
CREATE TABLE master.skill_category_mapping (
  id              SERIAL PRIMARY KEY,
  client          TEXT NOT NULL DEFAULT 'mathco',
  skill_tag       TEXT NOT NULL,
  category        TEXT NOT NULL,           -- SQL, Python, Power BI, ML, EDA+Stat, Cloud
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_by      TEXT,
  UNIQUE (client, skill_tag)
);
```

**Sample Data**:

| client | skill_tag | category |
|--------|-----------|----------|
| mathco | CTE | SQL |
| mathco | Join | SQL |
| mathco | Window | SQL |
| mathco | Pandas | Python |
| mathco | Scikit-learn | ML |
| mathco | GCP | Cloud |
| mathco | Azure | Cloud |

---

### 2.3 Raw Tables

#### `raw.learner_scores`

Synced from Enqurious DB by the ETL script. One row per learner per skill per deployment. Deduplication happens at query time (latest score wins).

```sql
CREATE TABLE raw.learner_scores (
  id              SERIAL PRIMARY KEY,
  enqurious_id    TEXT NOT NULL,           -- original ID from Enqurious
  learner_id      TEXT NOT NULL,
  learner_name    TEXT,
  role            TEXT NOT NULL,
  service_line    TEXT,
  client          TEXT NOT NULL DEFAULT 'mathco',
  skill_tag       TEXT NOT NULL,
  score           NUMERIC(4,1) NOT NULL CHECK (score BETWEEN 0 AND 10),
  assessed_by     TEXT,
  assessed_at     TIMESTAMPTZ NOT NULL,
  deployment_id   TEXT,                   -- to track redeployments
  synced_at       TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE (enqurious_id)                   -- prevent duplicate syncs
);

CREATE INDEX idx_learner_scores_learner ON raw.learner_scores (learner_id, skill_tag, client);
CREATE INDEX idx_learner_scores_assessed ON raw.learner_scores (assessed_at DESC);
```

> **Deduplication Rule**: When the same learner has multiple scores for the same skill (across redeployments), always use the **most recent** `assessed_at` record. This is enforced in the view layer, not by deleting rows — preserving full history.

---

### 2.4 Postgres Functions

#### `fn_proficiency_flag(score NUMERIC) → INTEGER`

Converts a raw score to a binary proficiency flag.

```sql
CREATE OR REPLACE FUNCTION fn_proficiency_flag(score NUMERIC)
RETURNS INTEGER AS $$
BEGIN
  IF score >= 7 THEN RETURN 1;
  ELSE RETURN 0;
  END IF;
END;
$$ LANGUAGE plpgsql IMMUTABLE;
```

---

#### `fn_mac_readiness_pct(flags_passed INTEGER, total_skills INTEGER) → NUMERIC`

Calculates MAC readiness percentage for a given MAC level.

```sql
CREATE OR REPLACE FUNCTION fn_mac_readiness_pct(flags_passed INTEGER, total_skills INTEGER)
RETURNS NUMERIC AS $$
BEGIN
  IF total_skills = 0 THEN RETURN 0;
  END IF;
  RETURN ROUND((flags_passed::NUMERIC / total_skills) * 100, 2);
END;
$$ LANGUAGE plpgsql IMMUTABLE;
```

---

#### `fn_mac_status(readiness_pct NUMERIC) → TEXT`

Applies the 65% benchmark to determine per-MAC status.

```sql
CREATE OR REPLACE FUNCTION fn_mac_status(readiness_pct NUMERIC)
RETURNS TEXT AS $$
BEGIN
  IF readiness_pct >= 65 THEN RETURN 'Ready';
  ELSE RETURN 'Not Ready';
  END IF;
END;
$$ LANGUAGE plpgsql IMMUTABLE;
```

---

#### `fn_final_mac_level(learner_id TEXT, role TEXT, client TEXT) → INTEGER`

Determines the highest consecutive MAC level a learner qualifies for.

**Logic**: A learner's final MAC = the highest MAC N such that they are "Ready" at all MAC levels 1 through N. Gaps are penalized (agreed logic — see Section 5).

```sql
CREATE OR REPLACE FUNCTION fn_final_mac_level(
  p_learner_id TEXT,
  p_role TEXT,
  p_client TEXT
)
RETURNS INTEGER AS $$
DECLARE
  max_consecutive INTEGER := 0;
  current_mac INTEGER := 1;
  mac_status TEXT;
BEGIN
  LOOP
    SELECT fn_mac_status(
      fn_mac_readiness_pct(
        SUM(fn_proficiency_flag(ls.score))::INTEGER,
        COUNT(*)::INTEGER
      )
    )
    INTO mac_status
    FROM (
      -- deduped: latest score per skill
      SELECT DISTINCT ON (skill_tag) skill_tag, score
      FROM raw.learner_scores
      WHERE learner_id = p_learner_id
        AND role = p_role
        AND client = p_client
      ORDER BY skill_tag, assessed_at DESC
    ) ls
    JOIN master.skill_mac_mapping m
      ON m.skill_tag = ls.skill_tag
      AND m.role = p_role
      AND m.client = p_client
      AND m.mac_level = current_mac;

    IF mac_status IS NULL OR mac_status = 'Not Ready' THEN
      EXIT;
    END IF;

    max_consecutive := current_mac;
    current_mac := current_mac + 1;

    EXIT WHEN current_mac > 5;
  END LOOP;

  RETURN max_consecutive;
END;
$$ LANGUAGE plpgsql;
```

---

### 2.5 Views

#### `reporting.v_deduped_scores`

Base view — always latest score per learner per skill.

```sql
CREATE OR REPLACE VIEW reporting.v_deduped_scores AS
SELECT DISTINCT ON (learner_id, skill_tag, client)
  learner_id,
  learner_name,
  role,
  service_line,
  client,
  skill_tag,
  score,
  assessed_by,
  assessed_at
FROM raw.learner_scores
ORDER BY learner_id, skill_tag, client, assessed_at DESC;
```

---

#### `reporting.v_mac_readiness`

Per learner, per skill category, per MAC level readiness.

```sql
CREATE OR REPLACE VIEW reporting.v_mac_readiness AS
SELECT
  ds.learner_id,
  ds.learner_name,
  ds.role,
  ds.service_line,
  ds.client,
  sc.category,
  sm.mac_level,
  COUNT(*) AS total_skills,
  SUM(fn_proficiency_flag(ds.score)) AS skills_passed,
  fn_mac_readiness_pct(
    SUM(fn_proficiency_flag(ds.score))::INTEGER,
    COUNT(*)::INTEGER
  ) AS readiness_pct,
  fn_mac_status(
    fn_mac_readiness_pct(
      SUM(fn_proficiency_flag(ds.score))::INTEGER,
      COUNT(*)::INTEGER
    )
  ) AS mac_status
FROM reporting.v_deduped_scores ds
JOIN master.skill_mac_mapping sm
  ON sm.skill_tag = ds.skill_tag
  AND sm.role = ds.role
  AND sm.client = ds.client
JOIN master.skill_category_mapping sc
  ON sc.skill_tag = ds.skill_tag
  AND sc.client = ds.client
GROUP BY
  ds.learner_id, ds.learner_name, ds.role,
  ds.service_line, ds.client, sc.category, sm.mac_level;
```

---

#### `reporting.v_final_verdicts`

One row per learner. The table Metabase queries for the main dashboard.

```sql
CREATE OR REPLACE VIEW reporting.v_final_verdicts AS
SELECT
  ds.learner_id,
  ds.learner_name,
  ds.role,
  ds.service_line,
  ds.client,
  fn_final_mac_level(ds.learner_id, ds.role, ds.client) AS final_mac_level,
  CASE
    WHEN fn_final_mac_level(ds.learner_id, ds.role, ds.client) >= 1
    THEN 'Ready'
    ELSE 'Not Ready'
  END AS verdict,
  ROUND(
    SUM(ds.score) / (COUNT(*) * 10.0) * 100, 2
  ) AS overall_skill_score_pct,
  MAX(ds.assessed_at) AS last_assessed_at
FROM reporting.v_deduped_scores ds
GROUP BY ds.learner_id, ds.learner_name, ds.role, ds.service_line, ds.client;
```

---

## 3. Python ETL Script

### 3.1 Purpose

Syncs learner scores from the Enqurious DB into `raw.learner_scores` in Supabase. Runs on demand (or scheduled). Idempotent — safe to re-run.

### 3.2 File Structure

```
mathco-etl/
├── etl.py              ← main entry point
├── db_enqurious.py     ← Enqurious DB connection + fetch query
├── db_supabase.py      ← Supabase connection + upsert logic
├── config.py           ← env vars (never commit secrets)
├── requirements.txt
└── .env                ← ENQURIOUS_DB_URL, SUPABASE_DB_URL
```

### 3.3 Core ETL Logic (`etl.py`)

```python
import logging
from db_enqurious import fetch_learner_scores
from db_supabase import upsert_scores

logging.basicConfig(level=logging.INFO)

def run_etl(client: str = "mathco"):
    logging.info(f"Starting ETL for client: {client}")

    scores = fetch_learner_scores(client=client)
    logging.info(f"Fetched {len(scores)} records from Enqurious")

    inserted, skipped = upsert_scores(scores)
    logging.info(f"Upserted: {inserted} | Skipped (duplicates): {skipped}")

    logging.info("ETL complete.")

if __name__ == "__main__":
    run_etl()
```

### 3.4 Enqurious Fetch Query (`db_enqurious.py`)

```python
import psycopg2
import os

ENQURIOUS_QUERY = """
  SELECT
    a.id::TEXT            AS enqurious_id,
    u.id::TEXT            AS learner_id,
    u.name                AS learner_name,
    r.name                AS role,
    sl.name               AS service_line,
    s.tag                 AS skill_tag,
    a.score,
    a.assessed_by,
    a.assessed_at,
    a.deployment_id
  FROM assessments a
  JOIN users u         ON u.id = a.learner_id
  JOIN roles r         ON r.id = u.role_id
  JOIN service_lines sl ON sl.id = u.service_line_id
  JOIN skills s        ON s.id = a.skill_id
  WHERE u.client_slug = %(client)s
    AND a.score IS NOT NULL
  ORDER BY a.assessed_at DESC;
"""

def fetch_learner_scores(client: str) -> list[dict]:
    conn = psycopg2.connect(os.environ["ENQURIOUS_DB_URL"])
    with conn.cursor() as cur:
        cur.execute(ENQURIOUS_QUERY, {"client": client})
        cols = [desc[0] for desc in cur.description]
        rows = [dict(zip(cols, row)) for row in cur.fetchall()]
    conn.close()
    return rows
```

### 3.5 Supabase Upsert (`db_supabase.py`)

```python
import psycopg2
import os

UPSERT_QUERY = """
  INSERT INTO raw.learner_scores (
    enqurious_id, learner_id, learner_name, role, service_line,
    client, skill_tag, score, assessed_by, assessed_at, deployment_id
  ) VALUES (
    %(enqurious_id)s, %(learner_id)s, %(learner_name)s, %(role)s,
    %(service_line)s, %(client)s, %(skill_tag)s, %(score)s,
    %(assessed_by)s, %(assessed_at)s, %(deployment_id)s
  )
  ON CONFLICT (enqurious_id) DO NOTHING;
"""

def upsert_scores(scores: list[dict], client: str = "mathco") -> tuple[int, int]:
    conn = psycopg2.connect(os.environ["SUPABASE_DB_URL"])
    inserted = 0
    skipped = 0
    with conn.cursor() as cur:
        for row in scores:
            row["client"] = client
            cur.execute(UPSERT_QUERY, row)
            if cur.rowcount == 1:
                inserted += 1
            else:
                skipped += 1
    conn.commit()
    conn.close()
    return inserted, skipped
```

---

## 4. Metabase Integration

### 4.1 Connection

Connect Metabase to Supabase via direct **Postgres connection** using the Supabase connection pooler credentials. Use a **read-only service role** with SELECT access only on the `reporting` schema.

```
Host:     db.<project-ref>.supabase.co
Port:     5432
Database: postgres
User:     metabase_readonly
Password: <set in Supabase dashboard>
Schema:   reporting
```

### 4.2 Key Metabase Queries

Metabase queries should be simple — all logic is already in the views.

**Learner Summary Dashboard**:
```sql
SELECT * FROM reporting.v_final_verdicts
WHERE client = 'mathco'
ORDER BY learner_name;
```

**MAC Readiness Breakdown by Category**:
```sql
SELECT * FROM reporting.v_mac_readiness
WHERE client = 'mathco'
  AND learner_id = {{learner_id}}
ORDER BY category, mac_level;
```

**Not Ready Learners**:
```sql
SELECT learner_name, role, service_line, final_mac_level, overall_skill_score_pct
FROM reporting.v_final_verdicts
WHERE client = 'mathco'
  AND verdict = 'Not Ready'
ORDER BY overall_skill_score_pct DESC;
```

---

## 5. Agreed Business Logic (Standardized)

This section documents the decisions that eliminate the current ambiguity. These are the canonical rules encoded in the database functions.

| Decision | Rule | Rationale |
|----------|------|-----------|
| **Deduplication** | Latest `assessed_at` wins per learner+skill | Most recent = most valid attempt |
| **Proficiency threshold** | Score ≥ 7 → Qualified | Existing rubric, no change |
| **MAC readiness benchmark** | ≥ 65% skills qualified → MAC Ready | Existing benchmark, no change |
| **Gap handling** | Gaps penalized — highest consecutive MAC only | Fairness: foundational levels must be solid |
| **Overall score** | Sum of all scores / (total skills × 10) × 100% | Simple percentage across all skills |
| **Verdict** | Final MAC ≥ 1 → Ready; Final MAC = 0 → Not Ready | Clear binary for hiring decisions |
| **Over-qualification** | Report actual final MAC; customer decides on cap | System reports truth; business decision stays with HR |

---

## 6. Access Control

### 6.1 Supabase Roles

| Role | Access | Used By |
|------|--------|---------|
| `admin` | Full access to `master` schema (INSERT, UPDATE, DELETE) | Amit, Sunil |
| `etl_writer` | INSERT on `raw.learner_scores` only | Python ETL script |
| `metabase_readonly` | SELECT on `reporting` schema only | Metabase |
| `anon` | No access | Blocked entirely |

### 6.2 Row-Level Security

Enable RLS on master tables to ensure only `admin` role can modify mappings:

```sql
ALTER TABLE master.skill_mac_mapping ENABLE ROW LEVEL SECURITY;

CREATE POLICY admin_only ON master.skill_mac_mapping
  USING (auth.role() = 'admin');
```

---

## 7. Build Plan

### Phase 1 — Foundation (Week 1)

| Task | Owner | Output |
|------|-------|--------|
| Create Supabase project | Amit | Project URL, credentials |
| Run schema SQL (master + raw tables) | Amit | Tables live |
| Populate `skill_mac_mapping` from current Sheets | Sunil | Master data seeded |
| Populate `skill_category_mapping` from current Sheets | Sunil | Master data seeded |
| Create Postgres functions (`fn_proficiency_flag`, etc.) | Amit | Functions deployed |
| Create views (`v_deduped_scores`, `v_mac_readiness`, `v_final_verdicts`) | Amit | Views live |

### Phase 2 — ETL (Week 2)

| Task | Owner | Output |
|------|-------|--------|
| Write `etl.py` + `db_enqurious.py` + `db_supabase.py` | Amit | ETL script |
| Test ETL with sample Enqurious data | Amit + Sunil | Verified sync |
| Validate deduplication logic with Girish Kumar test case | Sunil | Dedup confirmed |
| Validate MAC readiness logic with Shivam test case | Sunil | Logic confirmed |

### Phase 3 — Metabase (Week 3)

| Task | Owner | Output |
|------|-------|--------|
| Connect Metabase to Supabase (readonly) | Amit | Live connection |
| Rebuild Mathco dashboard queries on new views | Sunil | Dashboard working |
| Parallel run: old Sheets vs new system | Both | Sign-off checklist |
| Customer demo of new reports | Amit | Mathco approval |

### Phase 4 — Cutover (Week 4)

| Task | Owner | Output |
|------|-------|--------|
| Deprecate manual CSV upload process | Chetan | No more manual uploads |
| Schedule ETL (cron or manual trigger) | Amit | Agreed refresh cadence |
| Document mapping edit process for Sunil | Amit | Runbook written |
| Archive old Sheets (read-only, not deleted) | Sunil | Clean handoff |

---

## 8. Issues Resolved by This System

| Original Issue | Resolution |
|----------------|------------|
| Duplicate records / averaging errors | `DISTINCT ON` dedup in `v_deduped_scores`; latest score always wins |
| Mismatched skill scores vs. MAC verdicts | Both calculated in same system from same data; explained side-by-side in Metabase |
| Inconsistent readiness logic | Single canonical function `fn_final_mac_level`; one version, always |
| Under-representation bias (MAC with 2 skills) | Documented as known limitation; surfaced in readiness view for transparency |
| Manual CSV uploads / data staleness | ETL script replaces uploads; runs on demand |
| No audit trail | `updated_by` + `updated_at` on master tables; full score history in `raw.learner_scores` |
| Scalability | New client/role = new rows in master tables; no sheet changes needed |

---

## 9. Out of Scope (This Version)

- UI for editing master mappings (Supabase Table Editor is sufficient for now)
- Automated ETL scheduling (manual trigger acceptable for Phase 1)
- Multi-client isolation beyond `client` column filtering
- Historical MAC verdict tracking (current system computes live; no snapshot table yet)

---

## Appendix A — Environment Variables

```env
# .env (never commit to git)
ENQURIOUS_DB_URL=postgresql://user:password@host:5432/enqurious
SUPABASE_DB_URL=postgresql://etl_writer:password@db.<ref>.supabase.co:5432/postgres
```

## Appendix B — Test Cases for Validation

| Test Case | Learner | Expected Result |
|-----------|---------|-----------------|
| Duplicate handling | Girish Kumar | Latest score used; verdict = Ready |
| MAC gap logic | Avinash Yadav | Final MAC = 1 (gap at MAC2 penalized) |
| Score vs verdict mismatch | Shivam | 80% score shown alongside Not Ready with explanation |
| All MACs passed | Any | Final MAC = highest qualified level |
| MAC1 failed | Any | Verdict = Not Ready; final_mac_level = 0 |

---

*This document is the single source of truth for the Mathco MAC Report System rebuild. Any changes to business logic must be reflected here before being deployed to the database.*
