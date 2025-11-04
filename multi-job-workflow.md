# Multi-Job Resume Tailoring Workflow

## Overview

Handles 3-5 similar jobs efficiently by consolidating experience discovery while maintaining per-job research depth.

**Architecture:** Shared Discovery + Per-Job Tailoring

**Target Use Case:**
- Small batches (3-5 jobs)
- Moderately similar roles (60%+ requirement overlap)
- Continuous workflow (add jobs incrementally)

## Phase 0: Job Intake & Batch Initialization

**Goal:** Collect all job descriptions and initialize batch structure

**User Interaction:**

```
SKILL: "Let's set up your multi-job batch. How would you like to provide
the job descriptions?

1. Paste them all now (recommended for efficiency)
2. Provide one at a time
3. Provide URLs to fetch

For each job, I need:
- Job description (text or URL)
- Company name (if not in JD)
- Role title (if not in JD)
- Optional: Priority (high/medium/low) or notes"
```

**Data Collection Loop:**

For each job (until user says "done"):
1. Collect JD text or URL
2. Collect company name (extract from JD if possible, else ask)
3. Collect role title (extract from JD if possible, else ask)
4. Ask: "Priority for this job? (high/medium/low, default: medium)"
5. Ask: "Any notes about this job? (optional, e.g., 'referral from X')"
6. Assign job_id: "job-1", "job-2", etc.
7. Set status: "pending"
8. Add to batch

**Quick JD Parsing:**

For each job, lightweight extraction (NOT full research yet):

```python
# Pseudo-code
def quick_parse_jd(jd_text):
    return {
        "requirements_must_have": extract_requirements(jd_text, required=True),
        "requirements_nice_to_have": extract_requirements(jd_text, required=False),
        "technical_skills": extract_technical_keywords(jd_text),
        "soft_skills": extract_soft_skills(jd_text),
        "domain_areas": identify_domains(jd_text)
    }
```

Purpose: Just enough to identify gaps for discovery phase. Full research happens per-job later.

**Batch Initialization:**

Create batch directory structure:

```
resumes/batches/batch-{YYYY-MM-DD}-{slug}/
├── _batch_state.json          # State tracking
├── _aggregate_gaps.md         # Gap analysis (created in Phase 1)
├── _discovered_experiences.md # Discovery output (created in Phase 2)
└── (job directories created during per-job processing)
```

Initialize _batch_state.json:

```json
{
  "batch_id": "batch-2025-11-04-job-search",
  "created": "2025-11-04T10:30:00Z",
  "current_phase": "intake",
  "processing_mode": "interactive",
  "jobs": [
    {
      "job_id": "job-1",
      "company": "Microsoft",
      "role": "Principal PM - 1ES",
      "jd_text": "...",
      "jd_url": "https://...",
      "priority": "high",
      "notes": "Internal referral from Alice",
      "status": "pending",
      "requirements": ["Kubernetes", "CI/CD", "Leadership"],
      "gaps": []
    }
  ],
  "discoveries": [],
  "aggregate_gaps": {}
}
```

**Library Initialization:**

Run standard Phase 0 from SKILL.md (library initialization) once for the entire batch.

**Output:**

```
"Batch initialized with {N} jobs:
- Job 1: {Company} - {Role} (priority: {priority})
- Job 2: {Company} - {Role}
...

Next: Aggregate gap analysis across all jobs.
Continue? (Y/N)"
```

**Checkpoint:** User confirms batch is complete before proceeding.
