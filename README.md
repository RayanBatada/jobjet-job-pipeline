# JobJet — Job Board Scraper + Match Scoring + Resume Tailoring (n8n)

JobJet is an n8n automation that:
1) pulls fresh job postings via Apify (API-based ingestion),
2) scores/ranks roles with a Match % + missing-skills extraction (OpenAI),
3) generates a job-tailored resume by copying a Google Docs template + applying targeted edits,
4) logs everything into a Google Sheets tracker with upsert/dedupe logic.

---

## Google Sheets Template (copy this first)

Make your own copy of the tracker template here:

`https://docs.google.com/spreadsheets/d/1n8dlsvArRmVUWHYKXUAxDei9IapQ3C61ik1EjreMHUw/edit?usp=sharing`

After you copy it, you’ll use *your* sheet ID inside n8n.

---

## Tech Stack

- **n8n** (orchestration + scheduling)
- **Apify** (job ingestion actor → dataset output)
- **OpenAI API** (job fit scoring + missing-skills extraction + skills-section tailoring)
- **Google Docs** (resume template copy + replaceAll edits)
- **Google Drive** (export tailored resume as PDF)
- **Google Sheets** (application tracker with upserts/dedupes)

---

## What it Produces

### A Google Sheets tracker row per job
Typical fields include:
- Job URL (dedupe key or part of it)
- Company Name
- Role / Title
- Location
- Date Posted
- Salary (if present)
- Job Description (raw text)
- Apply URL
- Platform (e.g., “Job Board” / “LinkedIn”)
- Match % (score out of 100)
- Missing Skills (list)
- Resume link (Google Drive PDF view link)
- (Optional/computed) JD Preview formula column for quick scanning

---

## Workflow Overview (high-level)

1. **Schedule Trigger**  
   Runs daily at a set hour (e.g., 8 AM).

2. **Call API (Apify Actor)**  
   Requests job listings using your query filters (title, location, published window, etc.).

3. **Parse Data (Apify Dataset)**  
   Reads the dataset items into n8n.

4. **Limit**  
   Caps number of jobs processed per run (e.g., 20).

5. **Get Match % by comparing JD & Resume (OpenAI)**  
   - Computes a score (0–100)
   - Extracts missing skills (tools/tech only)
   - Outputs verdict (“keep”/“drop”)

6. **Filter Irrelevant Jobs**  
   Drops low-fit roles before spending time generating resumes.

7. **Tailor Resume to JD (OpenAI)**  
   Returns:
   - updated SKILLS section (4 lines)
   - updated graduation year
   - overflow_skills if needed

8. **Make a Resume Copy (Google Drive) → Update Copied Resume (Google Docs)**  
   Copies template, then runs replaceAll edits to inject tailored skills / grad year.

9. **Export Resume as PDF (Google Drive)**  
   Exports the edited doc to a PDF file.

10. **Append or Update Row (Google Sheets)**  
   Upserts the job row (dedupe by Job URL or Job Key) and writes match %, missing skills, and resume link.

---

## Setup

### 1) Copy the Google Sheets template
Use the template link above and create your own copy.

### 2) Import the workflow JSON into n8n
- In n8n: **Workflows → Import from File**
- Import your “public” JSON workflow.

### 3) Create credentials in n8n
You’ll need:
- **Apify OAuth2**
- **Google Docs OAuth2**
- **Google Drive OAuth2**
- **Google Sheets OAuth2**
- **OpenAI API** (or another AI of your choice)

> Note: When you share the workflow publicly, do **not** include credentials in the export.

### 5) Tune your Apify query after selecting an API
Adjust fields like:
- `title` (keyword query)
- `location`
- `publishedAt` (time window)
- `rows` (how many to fetch)

### 6) Run once manually, then enable schedule
- Click **Execute Workflow** to validate
- Check: sheet gets rows, PDF links open, dedupe works
- Turn workflow “Active” to schedule daily runs

---

## Notes on “Fair play” / compliance

Even if you’re using an API-based tool (Apify actor), some job sites have strict Terms of Service around automated access. Keep that in mind if using the wrong APIs.

---

## Troubleshooting

**Only 6–8 jobs returned when you asked for 20**
- Apify actor may return fewer results given the filters (published window, location, title query).
- Make sure `rows` in the Apify input matches your expectation.
- Confirm your n8n **Limit** node isn’t set lower than you think.

**Sheet upsert doesn’t dedupe**
- Dedupe is controlled by `matchingColumns`.
- Use **Job Key** if the source provides a stable ID; Job URL can change.

**Resume export link doesn’t open**
- Confirm the Drive file permissions (you may need “Anyone with link can view” if you want shareable links).
---

## Contact / Credits
Built by: Rayan Batada 
Stack: n8n + Apify + Google Workspace + OpenAI
