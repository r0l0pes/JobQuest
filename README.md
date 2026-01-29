# JobSearch 🚀

An automated system to find, tailor resumes for, and track job applications, specifically designed for Product Manager roles in the European startup ecosystem.

## 🛠 Features

- **Automated Job Search**: Scans startup lists and uses DuckDuckGo to identify relevant Product Manager job postings.
- **Resume Tailoring**: Generates tailored resumes by compiling LaTeX templates (using `pdflatex`).
- **Form Automation**: Automatically fills out job application forms (ATS) using Playwright while preserving a manual review step.
- **Notion Integration**: Automatically tracks applications in a Notion database, including status updates and document links.
- **Skill-Based Automation**: Includes specialized "agentic skills" for AI-assisted QA generation and resume customization.

## 📁 Project Structure

- `batch_job_search.py`: Core script for bulk job discovery.
- `scripts/`:
  - `form_filler.py`: Generic ATS form filler using Playwright.
  - `notion_tracker.py`: Syncs application progress to Notion.
  - `render_pdf.py`: Compiles LaTeX resumes into PDFs.
  - `notion_db_setup.py`: Initializes the Notion workspace schema.
- `templates/`: Contains LaTeX resume templates.
- `.agent/skills/`: Custom AI workflows for tailoring content.

## 🚀 Getting Started

1. **Install Dependencies**:
   ```bash
   pip install -r requirements.txt
   playwright install chromium
   ```

2. **Configuration**:
   Update `.env` with your Notion API tokens and personal details.

3. **Search for Jobs**:
   ```bash
   python batch_job_search.py
   ```

4. **Fill Applications**:
   ```bash
   python scripts/form_filler.py --url <JOB_URL> --resume-pdf <PATH_TO_PDF>
   ```

## ⚠️ Notes
- The form filler intentionally pauses before submission to allow for manual review and CAPTCHA handling.
- Requires `pdflatex` (e.g., from TeX Live or MiKTeX) for resume generation.
