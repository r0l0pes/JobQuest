---
name: ats-fixer
description: Run ATS consistency and keyword coverage checks on a tailored resume before PDF rendering. Compares the tailored .tex file against the job posting to catch missing keywords, title/seniority mismatches, and visibility issues. Produces a structured report and proposes minimal edits. Use after resume-tailor, before render_pdf.py.
---

# ATS Fixer Skill

## Purpose
Reduce avoidable ATS rejections by verifying keyword coverage, data consistency, and keyword visibility between the job posting and the tailored resume. This is a verification layer, not a rewriting layer. It catches what resume-tailor may have missed.

## How Modern ATS Actually Works (2025/2026)
Understanding ATS behavior prevents both under-optimization and over-optimization.

**What ATS platforms do:**
- 98.8% of Fortune 500 companies use ATS (Greenhouse, Lever, Workday, iCIMS, Taleo are the most common)
- 99.7% of recruiters use keyword filters to sort and prioritize applicants
- Modern ATS uses NLP and semantic matching, not just exact keyword counts. Greenhouse, for example, understands that "led a team of five" matches "managed five direct reports"
- The process is: Parsing (extract text into structured fields) -> Keyword Analysis (compare against job requirements) -> Scoring/Ranking (relevance score determines if resume reaches a human)

**What most people get wrong:**
- Only ~3% of resumes actually fail at the parsing stage. Most rejections attributed to ATS are human decisions after ATS ranking
- Keyword stuffing is actively counterproductive in 2025. Modern ATS flags unnatural repetition
- The goal is BOTH ATS readability AND human readability. A resume that passes ATS but reads badly to a recruiter still gets rejected
- Context matters more than count. ATS evaluates whether experience genuinely matches requirements, not just whether a keyword appears

**What actually causes ATS rejections:**
1. Missing keywords entirely (biggest factor)
2. Wrong terminology (e.g., "AngularJS" vs "Angular", abbreviation without full form)
3. Title mismatches (creative titles like "Chief Happiness Officer" don't match "HR Manager")
4. Missing standard sections (no Summary, no Education)
5. Non-parseable formatting (tables, graphics, text boxes, unusual fonts, scanned PDFs)

**What does NOT cause issues:**
- LaTeX-generated PDFs parse fine (text-based, not scanned)
- Standard bullet points, bold, italic all parse correctly
- Two-column layouts work IF using native document formatting (not text boxes)

## When to Use
Run this skill:
- After resume-tailor has produced a `.tex` file, before compiling PDF
- When user says "check ATS", "ATS check", or "is this ATS ready"
- Automatically as part of the application workflow (after tailoring, before rendering)

## What This Is NOT
- Not an ATS simulator or score predictor
- Not a resume rewriter
- Not a keyword stuffer (modern ATS penalizes this)
- Not a replacement for resume-tailor (it runs after it)

## Inputs
1. **Job posting text** - Already available in conversation from the resume-tailor step
2. **Tailored .tex file** - At `output/Resume_Rodrigo-Lopes.tex`
3. **Master resume data** - From Notion if needed for fact-checking proposed edits

## Process

### Step 1: Extract Job Requirements
From the job posting, build three lists:

**Must-have keywords** - Skills, tools, or qualifications that appear in required sections or are repeated 2+ times:
- Technical skills (e.g., "SQL", "Figma", "A/B testing")
- Methodologies (e.g., "Agile", "OKRs", "Design Thinking")
- Domain terms (e.g., "e-commerce", "SaaS", "B2B")
- Role-level signals (e.g., "senior", "lead", "cross-functional")

**Nice-to-have keywords** - Skills in preferred/bonus sections or mentioned once:
- Secondary tools
- Certifications
- Industry-specific jargon

**Job metadata** - Structured fields that ATS forms typically parse:
- Job title (exact wording used in the posting)
- Seniority level
- Location / remote policy
- Years of experience expected

### Step 2: Parse Tailored Resume
Read `output/Resume_Rodrigo-Lopes.tex` and extract:
- Summary text
- Each experience entry (role title, company, bullet points)
- Skills/Technical Proficiency section entries
- Certifications
- The resume's implied seniority and title

### Step 3: Keyword Coverage Check
For each must-have and nice-to-have keyword, check for presence using **semantic matching**, not just exact string matching:
- "Product roadmap" covers "roadmap ownership"
- "Cross-functional teams" covers "cross-functional leadership"
- "User research" covers "customer research" and "user testing"

But flag cases where the job posting uses a specific term that should appear verbatim:
- Tool names (Figma, Jira, Mixpanel) require exact matches
- Certifications require exact matches
- Industry-standard acronyms need both forms: "OKRs (Objectives and Key Results)" or "A/B testing"

Classify each keyword:
```
COVERED   - appears in Summary or Experience bullets (high visibility)
SEMANTIC  - a close synonym appears but not the exact term (may pass modern ATS but risky with older systems)
LOW_VIS   - appears only in Skills/Certifications section (parseable but not prominent)
MISSING   - not found anywhere in resume
N/A       - candidate does not have this skill (do not add)
```

### Step 4: Consistency Checks
Check for conflicts between resume and job posting:

**Title alignment**
- Does the resume's most recent job title reasonably match the target role?
- Modern ATS uses semantic matching here, but exact or close match is still strongest
- Flag significant mismatches (e.g., resume says "Product Manager" but job is "Engineering Manager")
- Note: "Senior Product Manager" applying for "Product Manager" is fine (overqualified is not a mismatch)

**Seniority alignment**
- Does the resume's experience level match what the job expects?
- Flag if resume undersells (e.g., "9+ years" but summary says "8+ years") or oversells

**Location alignment**
- Does the resume header location conflict with the job location?
- Flag if the job is location-specific and resume shows a different city

**Years of experience**
- Does the summary's stated years match actual career timeline?
- Flag inconsistencies

**Abbreviation check**
- For any acronym in the resume, verify the full form also appears (at least once)
- "OKRs" should have "Objectives and Key Results" somewhere, or vice versa
- Tool abbreviations that are universally known (SQL, API, CRM) don't need expansion
- Industry-specific abbreviations DO need expansion on first use

### Step 5: Visibility Check
High-priority keyword zones (where ATS and recruiters look first):
1. **Summary section** - First thing parsed and read
2. **Most recent job title** - Strong signal for role matching
3. **First bullet of each role** - Highest-visibility experience content
4. **Skills section** - Parsed as structured data by most ATS

For the top 5 must-have keywords, verify placement:
- At least 1 occurrence in Summary
- At least 1 occurrence in a recent Experience bullet (WFP or FORVIA)
- If a critical keyword only appears in Skills section, flag as LOW_VIS

Target: 60-80% of job posting keywords should appear in the resume. Below 60% is high risk. Above 80% may signal over-optimization.

### Step 6: Generate Report
Produce two outputs:

**JSON report** at `output/ats_report_<Company>.json`:
```json
{
  "company": "CompanyName",
  "job_title": "Target Role",
  "timestamp": "2026-01-29",
  "keyword_coverage": {
    "must_have": {
      "covered": ["keyword1", "keyword2"],
      "semantic_match": [{"keyword": "keyword3", "matched_by": "synonym used"}],
      "low_visibility": ["keyword4"],
      "missing": ["keyword5"],
      "not_applicable": ["keyword6"]
    },
    "nice_to_have": {
      "covered": [],
      "semantic_match": [],
      "low_visibility": [],
      "missing": [],
      "not_applicable": []
    }
  },
  "consistency": {
    "title_match": true,
    "seniority_match": true,
    "location_match": false,
    "location_note": "Job is Barcelona, resume says Berlin",
    "experience_years_match": true,
    "abbreviation_issues": []
  },
  "coverage_score": {
    "must_have_total": 10,
    "must_have_covered": 7,
    "must_have_visible": 5,
    "coverage_pct": 70,
    "verdict": "NEEDS REVIEW"
  },
  "suggested_edits": []
}
```

**Markdown report** at `output/ats_report_<Company>.md`:
```markdown
# ATS Check: [Company] - [Job Title]

## Coverage: X/Y must-have keywords (Z%)

### Must-Have Keywords
| Keyword | Status | Location in Resume |
|---------|--------|--------------------|
| SQL | COVERED | Skills, Accenture bullet 1 |
| Agile | SEMANTIC | "agile sprint cycles" in Accenture |
| SaaS | MISSING | - |
| OKRs | LOW_VIS | Skills only |

### Nice-to-Have Keywords
| Keyword | Status | Location in Resume |
|---------|--------|--------------------|
| ...     | ...    | ...                |

## Consistency
- Title: OK / MISMATCH (detail)
- Seniority: OK / MISMATCH (detail)
- Location: OK / FLAG (detail)
- Experience years: OK / MISMATCH
- Abbreviations: OK / [list issues]

## Suggested Edits
1. **[keyword]** - [edit type]
   - Before: "current text..."
   - After: "proposed text..."
   - Why: [rationale]

## Verdict: [READY / NEEDS REVIEW / HIGH RISK]
- READY: 80%+ coverage, no consistency issues
- NEEDS REVIEW: 60-79% coverage or minor consistency issues
- HIGH RISK: <60% coverage or major consistency issues
```

### Step 7: Propose Edits
For each MISSING or LOW_VIS keyword that the candidate actually has:

1. Find the best existing bullet to modify (prefer recent roles, first bullets)
2. Propose a minimal text change that inserts the keyword naturally
3. Show before/after
4. Mark each edit as the type:
   - `keyword_add` - inserting a missing keyword into an experience bullet
   - `keyword_promote` - moving a LOW_VIS keyword from Skills into a bullet
   - `semantic_to_exact` - replacing a synonym with the exact job posting term
   - `consistency_fix` - fixing a title/seniority/location issue
   - `abbreviation_fix` - adding full form or abbreviation

**Edit quality rules:**
- Each edit should change no more than 5-10 words in a bullet
- The edited bullet must still read naturally to a human
- Never insert more than 2 keywords into a single bullet
- If a keyword can't be inserted naturally, note it as "manual review needed"

### Step 8: Present to User
Show the Markdown report in the conversation. Ask user which edits to apply. Only modify the .tex file after explicit approval.

After approved edits are applied, compile PDF:
```bash
venv/bin/python scripts/render_pdf.py output/Resume_Rodrigo-Lopes.tex
```

## Integration with Workflow
The full application flow with ATS Fixer:
1. User pastes job URL
2. `resume-tailor` generates tailored .tex
3. **`ats-fixer` runs coverage + consistency checks**
4. **User reviews report, approves/rejects edits**
5. `render_pdf.py` compiles final PDF
6. Cover letter / Q&A generation (qa-generator)
7. `notion_tracker.py` tracks application

## DO
- Use semantic matching (modern ATS does this) but flag when exact terms are safer
- Only propose edits for keywords the candidate genuinely possesses
- Verify against master resume in Notion before suggesting any addition
- Keep edits minimal (change a few words, not entire bullets)
- Prefer modifying existing text over adding new bullets
- Show clear before/after for every proposed edit
- Flag when a keyword is genuinely not applicable (candidate lacks the skill)
- Include both abbreviation and full form for industry-specific terms
- Respect the existing LaTeX format exactly
- Optimize for both ATS parsing AND human readability

## DO NOT
- Invent skills, tools, certifications, or achievements
- Keyword stuff (modern ATS penalizes unnatural repetition)
- Rewrite entire bullets or sections
- Add keywords the candidate does not have, even if they're "must-have"
- Override the user's decision to skip an edit
- Attempt to predict or simulate ATS scoring algorithms
- Change verified metrics or numbers
- Inflate scope or seniority of past roles
- Add content not supported by Notion master resume data
- Optimize past 80% keyword coverage (diminishing returns, stuffing risk)
