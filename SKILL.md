---
name: resume-tailoring
description: Use when creating tailored resumes for job applications - researches company/role, creates optimized templates, conducts branching experience discovery to surface undocumented skills, and generates professional multi-format resumes from user's resume library while maintaining factual integrity
---

# Resume Tailoring Skill

## Overview

Generates high-quality, tailored resumes optimized for specific job descriptions while maintaining factual integrity. Builds resumes around the holistic person by surfacing undocumented experiences through conversational discovery.

**Core Principle:** Truth-preserving optimization - maximize fit while maintaining factual integrity. Never fabricate experience, but intelligently reframe and emphasize relevant aspects.

**Mission:** A person's ability to get a job should be based on their experiences and capabilities, not on their resume writing skills.

## When to Use

Use this skill when:
- User provides a job description and wants a tailored resume
- User has multiple existing resumes in markdown format
- User wants to optimize their application for a specific role/company
- User needs help surfacing and articulating undocumented experiences

**DO NOT use for:**
- Generic resume writing from scratch (user needs existing resume library)
- Cover letters (different skill)
- LinkedIn profile optimization (different skill)

## Quick Start

**Required from user:**
1. Job description (text or URL)
2. Resume library location (defaults to `resumes/` in current directory)

**Workflow:**
1. Build library from existing resumes
2. Research company/role
3. Create template (with user checkpoint)
4. Optional: Branching experience discovery
5. Match content with confidence scoring
6. Generate MD + DOCX + PDF + Report
7. User review → Optional library update

## Implementation

See supporting files:
- `research-prompts.md` - Structured prompts for company/role research
- `matching-strategies.md` - Content matching algorithms and scoring
- `branching-questions.md` - Experience discovery conversation patterns

## Workflow Details

### Phase 0: Library Initialization

**Always runs first - builds fresh resume database**

**Process:**

1. **Locate resume directory:**
   ```
   User provides path OR default to ./resumes/
   Validate directory exists
   ```

2. **Scan for markdown files:**
   ```
   Use Glob tool: pattern="*.md" path={resume_directory}
   Count files found
   Announce: "Building resume library... found {N} resumes"
   ```

3. **Parse each resume:**
   For each resume file:
   - Use Read tool to load content
   - Extract sections: roles, bullets, skills, education
   - Identify patterns: bullet structure, length, formatting

4. **Build experience database structure:**
   ```json
   {
     "roles": [
       {
         "role_id": "company_title_year",
         "company": "Company Name",
         "title": "Job Title",
         "dates": "YYYY-YYYY",
         "description": "Role summary",
         "bullets": [
           {
             "text": "Full bullet text",
             "themes": ["leadership", "technical"],
             "metrics": ["17x improvement", "$3M revenue"],
             "keywords": ["cross-functional", "program"],
             "source_resumes": ["resume1.md"]
           }
         ]
       }
     ],
     "skills": {
       "technical": ["Python", "Kusto", "AI/ML"],
       "product": ["Roadmap", "Strategy"],
       "leadership": ["Stakeholder mgmt"]
     },
     "education": [...],
     "user_preferences": {
       "typical_length": "1-page|2-page",
       "section_order": ["summary", "experience", "education"],
       "bullet_style": "pattern"
     }
   }
   ```

5. **Tag content automatically:**
   - Themes: Scan for keywords (leadership, technical, analytics, etc.)
   - Metrics: Extract numbers, percentages, dollar amounts
   - Keywords: Frequent technical terms, action verbs

**Output:** In-memory database ready for matching

**Code pattern:**
```python
# Pseudo-code for reference
library = {
    "roles": [],
    "skills": {},
    "education": []
}

for resume_file in glob("resumes/*.md"):
    content = read(resume_file)
    roles = extract_roles(content)
    for role in roles:
        role["bullets"] = tag_bullets(role["bullets"])
        library["roles"].append(role)

return library
```
