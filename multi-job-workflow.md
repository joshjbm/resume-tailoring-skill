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

## Phase 1: Aggregate Gap Analysis

**Goal:** Build unified gap list across all jobs to guide single efficient discovery session

**Process:**

**1.1 Extract Requirements from All JDs:**

For each job:
- Parse requirements (already done in Phase 0 quick parse)
- Categorize: must-have vs nice-to-have
- Extract keywords and skill areas

Example output:
```
Job 1 (Microsoft 1ES): Kubernetes, CI/CD, cross-functional leadership, Azure
Job 2 (Google Cloud): Kubernetes, GCP, distributed systems, team management
Job 3 (AWS): Container orchestration, AWS services, program management
```

**1.2 Match Against Resume Library:**

For each requirement across ALL jobs:
1. Search library for matching experiences (using matching-strategies.md)
2. Score confidence (0-100%)
3. Flag as gap if confidence < 60%

```python
# Pseudo-code
for job in batch.jobs:
    for requirement in job.requirements:
        matches = search_library(requirement)
        best_score = max(match.score for match in matches)
        if best_score < 60:
            flag_as_gap(requirement, best_score, job.job_id)
```

**1.3 Build Aggregate Gap Map:**

Deduplicate gaps across jobs and prioritize:

```python
# Pseudo-code
def build_aggregate_gaps(all_gaps):
    gap_map = {}

    for gap in all_gaps:
        if gap.name not in gap_map:
            gap_map[gap.name] = {
                "appears_in_jobs": [],
                "best_match": gap.confidence,
                "priority": 0
            }
        gap_map[gap.name]["appears_in_jobs"].append(gap.job_id)

    # Prioritize
    for gap_name, gap_data in gap_map.items():
        job_count = len(gap_data["appears_in_jobs"])
        if job_count >= 3:
            gap_data["priority"] = 3  # Critical
        elif job_count == 2:
            gap_data["priority"] = 2  # Important
        else:
            gap_data["priority"] = 1  # Job-specific

    return gap_map
```

**1.4 Create Gap Analysis Report:**

Generate `_aggregate_gaps.md`:

```markdown
# Aggregate Gap Analysis
**Batch:** batch-2025-11-04-job-search
**Generated:** 2025-11-04T11:00:00Z

## Coverage Summary

- Job 1 (Microsoft): 68% coverage, 5 gaps
- Job 2 (Google): 72% coverage, 4 gaps
- Job 3 (AWS): 65% coverage, 6 gaps

## Critical Gaps (appear in 3+ jobs)

### Kubernetes at scale
- **Appears in:** Jobs 1, 2, 3
- **Current best match:** 45% confidence
- **Match source:** "Deployed containerized app for nonprofit" (2023)
- **Gap:** No production Kubernetes management at scale

### CI/CD pipeline management
- **Appears in:** Jobs 1, 2, 3
- **Current best match:** 58% confidence
- **Match source:** "Set up GitHub Actions workflow" (2024)
- **Gap:** Limited enterprise CI/CD experience

## Important Gaps (appear in 2 jobs)

### Cloud-native architecture
- **Appears in:** Jobs 2, 3
- **Current best match:** 52% confidence

### Cross-functional team leadership
- **Appears in:** Jobs 1, 2
- **Current best match:** 67% confidence (not a gap, but could improve)

## Job-Specific Gaps

### Azure-specific experience
- **Appears in:** Job 1 only
- **Current best match:** 40% confidence

### GCP experience
- **Appears in:** Job 2 only
- **Current best match:** 35% confidence

## Aggregate Statistics

- **Total gaps:** 14
- **Unique gaps:** 8 (after deduplication)
- **Critical gaps:** 3
- **Important gaps:** 4
- **Job-specific gaps:** 1

## Recommended Discovery Time

- Critical gaps (3 gaps × 5-7 min): 15-20 minutes
- Important gaps (4 gaps × 3-5 min): 12-20 minutes
- Job-specific gaps (1 gap × 2-3 min): 2-3 minutes

**Total estimated discovery time:** 30-40 minutes

For 3 similar jobs, this replaces 3 × 15 min = 45 min of sequential discovery.
```

**1.5 Update Batch State:**

```json
{
  "current_phase": "gap_analysis",
  "aggregate_gaps": {
    "critical_gaps": [
      {
        "gap_name": "Kubernetes at scale",
        "appears_in_jobs": ["job-1", "job-2", "job-3"],
        "current_best_match": 45,
        "priority": 3
      }
    ],
    "important_gaps": [...],
    "job_specific_gaps": [...]
  }
}
```

**Output to User:**

```
"Gap analysis complete! Here's what I found:

COVERAGE SUMMARY:
- Job 1 (Microsoft): 68% coverage, 5 gaps
- Job 2 (Google): 72% coverage, 4 gaps
- Job 3 (AWS): 65% coverage, 6 gaps

AGGREGATE GAPS (14 total, 8 unique after deduplication):
- 3 critical gaps (appear in all jobs) 🔴
- 4 important gaps (appear in 2 jobs) 🟡
- 1 job-specific gap 🔵

I recommend a 30-40 minute experience discovery session to address
these gaps. This will benefit all 3 applications.

Would you like to:
1. START DISCOVERY - Address gaps through conversational discovery
2. SKIP DISCOVERY - Proceed with current library (not recommended)
3. REVIEW GAPS - See detailed gap analysis first

Recommendation: Option 1 or 3 (review then start)"
```

**Checkpoint:** User chooses next action before proceeding.

## Phase 2: Shared Experience Discovery

**Goal:** Surface undocumented experiences across all gaps through single conversational session

**Core Principle:** Same branching interview from branching-questions.md, but with multi-job context for each question.

**Session Flow:**

**2.1 Start with Highest-Leverage Gaps:**

Process gaps in priority order:
1. Critical gaps (appear in 3+ jobs) - 5-7 min each
2. Important gaps (appear in 2 jobs) - 3-5 min each
3. Job-specific gaps - 2-3 min each

**2.2 Multi-Job Contextualized Questions:**

For each gap, provide multi-job context before branching interview:

**Single-Job Version (from branching-questions.md):**
```
"I noticed the job requires Kubernetes experience. Have you worked with Kubernetes?"
```

**Multi-Job Version (new):**
```
"Kubernetes experience appears in 3 of your target jobs (Microsoft, Google, AWS).

This is a HIGH-LEVERAGE gap - addressing it helps multiple applications.

Current best match: 45% confidence ('Deployed containerized app for nonprofit')

Have you worked with Kubernetes or container orchestration?"
```

**2.3 Conduct Branching Interview:**

For each gap:
1. Initial probe with multi-job context (see above)
2. Branch based on answer using branching-questions.md patterns:
   - YES → Deep dive (scale, challenges, metrics)
   - INDIRECT → Explore role and transferability
   - ADJACENT → Explore related experience
   - PERSONAL → Assess recency and substance
   - NO → Try broader category or move on

3. Follow up systematically:
   - "What," "how," "why" questions
   - Quantify: "Any metrics?"
   - Contextualize: "Was this production?"
   - Validate: "This addresses {gap} for {jobs}"

4. Capture immediately with job tags:

**2.4 Capture Structure:**

As experiences are discovered, capture to `_discovered_experiences.md`:

```markdown
# Discovered Experiences
**Batch:** batch-2025-11-04-job-search
**Discovery Date:** 2025-11-04T11:30:00Z

## Experience 1: Kubernetes CI/CD for nonprofit project

**Context:** Side project, 2023-2024, production deployment

**Scope:**
- GitHub Actions pipeline with Kubernetes deployments
- 3 nonprofit organizations using it
- Integrated pytest for testing
- Managed scaling and monitoring

**Metrics:**
- 3 production deployments
- 99.9% uptime over 12 months
- Reduced deployment time from 2 hours to 15 minutes

**Addresses gaps in:**
- Jobs 1, 2, 3: Kubernetes at scale
- Jobs 1, 2: CI/CD pipeline management

**Confidence Improvement:**
- Kubernetes: 45% → 75% (+30%)
- CI/CD: 58% → 82% (+24%)

**Bullet Draft:**
"Designed and implemented Kubernetes-based CI/CD pipeline using GitHub Actions
and pytest, supporting production deployments for 3 nonprofit organizations with
99.9% uptime and 87% reduction in deployment time"

**Integration Decision:** [Pending user approval]

---

## Experience 2: Azure migration for university lab

**Context:** Graduate research, 2022-2023

**Scope:**
- Migrated on-premise compute to Azure VMs
- Set up Azure DevOps for lab
- Managed costs and resource allocation

**Metrics:**
- Migrated 15 TB of data
- Reduced compute costs by 40%
- Supported 25 researchers

**Addresses gaps in:**
- Job 1 only: Azure-specific experience

**Confidence Improvement:**
- Azure: 40% → 70% (+30%)

**Bullet Draft:**
"Led Azure cloud migration for university research lab, migrating 15 TB of data
and implementing Azure DevOps, reducing compute costs by 40% while supporting
25 researchers"

**Integration Decision:** [Pending user approval]
```

**2.5 Track Coverage Improvement in Real-Time:**

After each discovery, update user:

```
"Great! That addresses Kubernetes for all 3 jobs.

UPDATED COVERAGE:
- Job 1 (Microsoft): 68% → 78% (+10%)
- Job 2 (Google): 72% → 82% (+10%)
- Job 3 (AWS): 65% → 75% (+10%)

Remaining critical gaps: 2 (down from 3)

Continue with next gap? (Y/N)"
```

**2.6 Integration Decision Per Experience:**

After discovery session complete:

```
"Excellent! I captured 5 new experiences addressing gaps across your jobs.

For each experience, how should I integrate it?

---
EXPERIENCE 1: Kubernetes CI/CD for nonprofit project
├─ Addresses: Jobs 1, 2, 3
└─ Options:
   1. ADD TO LIBRARY FOR ALL JOBS - Integrate and use everywhere
   2. ADD TO LIBRARY, USE SELECTIVELY - User picks which jobs
   3. SKIP - Don't integrate

Your choice for Experience 1? (1/2/3)

---
EXPERIENCE 2: Azure migration for university lab
├─ Addresses: Job 1 only
└─ Options:
   1. ADD TO LIBRARY - Integrate for Job 1
   2. SKIP - Not needed

Your choice for Experience 2? (1/2)
```

**2.7 Enrich Library:**

For each approved experience:
1. Add to library database
2. Tag with metadata:
   - discovered_date
   - addressed_gaps
   - used_in_jobs
   - confidence_improvement

**2.8 Update Batch State:**

```json
{
  "current_phase": "discovery",
  "discoveries": [
    {
      "experience_id": "disc-1",
      "text": "Kubernetes CI/CD for nonprofit project",
      "context": "Side project, 2023-2024, production",
      "scope": "GitHub Actions, 3 nonprofits, pytest, monitoring",
      "addresses_jobs": ["job-1", "job-2", "job-3"],
      "addresses_gaps": ["Kubernetes", "CI/CD"],
      "confidence_improvement": {
        "Kubernetes": {"before": 45, "after": 75},
        "CI/CD": {"before": 58, "after": 82}
      },
      "integrated": true,
      "bullet_draft": "Designed and implemented..."
    }
  ]
}
```

**Output:**

```
"Discovery complete!

SUMMARY:
- New experiences captured: 5
- Experiences integrated: 5
- Average coverage improvement: +16%

FINAL COVERAGE:
- Job 1 (Microsoft): 68% → 85% (+17%)
- Job 2 (Google): 72% → 88% (+16%)
- Job 3 (AWS): 65% → 78% (+13%)

Remaining gaps: 5 (down from 14)
├─ 0 critical gaps ✓
├─ 2 important gaps
└─ 3 job-specific gaps

Ready to proceed with per-job processing? (Y/N)"
```

**Checkpoint:** User approves before moving to per-job processing.
