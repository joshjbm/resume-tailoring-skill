# Resume Tailoring Skill

AI-powered resume generation that researches roles, surfaces undocumented experiences, and creates tailored resumes from user's library.

## Quick Start

**User invocation:**
```
"I want to apply for [Role] at [Company]. Here's the JD: [paste/URL]"
```

**Skill automatically:**
1. Builds library from existing resumes
2. Researches company and role
3. Creates optimized template
4. Offers branching experience discovery
5. Matches content with confidence scores
6. Generates MD + DOCX + PDF + Report
7. Optionally updates library

## Files

- `SKILL.md` - Main skill implementation
- `research-prompts.md` - Company/role research templates
- `matching-strategies.md` - Content scoring algorithms
- `branching-questions.md` - Experience discovery patterns
- `README.md` - This file

## Key Features

**🔍 Deep Research**
- Company culture and values
- Role benchmarking via LinkedIn
- Success profile synthesis

**💬 Branching Discovery**
- Conversational experience surfacing
- Dynamic follow-up questions
- Surfaces undocumented work

**🎯 Smart Matching**
- Confidence-scored content selection
- Transparent gap identification
- Truth-preserving reframing

**📄 Multi-Format Output**
- Professional markdown
- ATS-friendly DOCX
- Print-ready PDF
- Interview prep report

**🔄 Self-Improving**
- Library grows with each resume
- Successful patterns reused
- New experiences captured

## Architecture

```
Phase 0: Library Build (always first)
   ↓
Phase 1: Research (JD + Company + Role)
   ↓
Phase 2: Template (Structure + Titles)
   ↓  [CHECKPOINT]
Phase 2.5: Experience Discovery (Optional, Branching)
   ↓
Phase 3: Assembly (Matching + Scoring)
   ↓  [CHECKPOINT]
Phase 4: Generation (MD + DOCX + PDF + Report)
   ↓  [USER REVIEW]
Phase 5: Library Update (Conditional)
```

## Design Philosophy

**Truth-Preserving Optimization:**
- NEVER fabricate experience
- Intelligently reframe and emphasize
- Transparent about gaps

**Holistic Person Focus:**
- Surface undocumented experiences
- Value volunteer work, side projects
- Build around complete background

**User Control:**
- Checkpoints at key decisions
- Options, not mandates
- Can adjust or go back

## Usage Patterns

**Internal role (same company):**
- Features most relevant internal experience
- Uses internal terminology
- Leverages organizational knowledge

**External role (new company):**
- Deep company research
- Cultural fit emphasis
- Risk mitigation

**Career transition:**
- Title reframing
- Transferable skill emphasis
- Bridge domain gaps

**With career gaps:**
- Gaps as valuable experience
- Alternative activities highlighted
- Truthful, positive framing

## Testing

See Testing Guidelines section in SKILL.md

**Key tests:**
- Happy path (full workflow)
- Minimal library (2 resumes)
- Research failures (obscure company)
- Experience discovery value
- Title reframing accuracy
- Multi-format generation

## Contributing

When modifying:
1. Update SKILL.md
2. Run regression tests
3. Update this README if architecture changes
4. Commit with descriptive message

## Design Reference

See `docs/plans/2025-10-31-resume-tailoring-skill-design.md` for:
- Complete design rationale
- Detailed phase specifications
- Success criteria
- Future enhancements
