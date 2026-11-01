# Implementation — Job Fit Evaluator

**Version:** 2.0 (30 October 2026). Supersedes v1. A change log is at the end.

## 1. Overview

| Layer | Tool | Role |
|---|---|---|
| Interface | Lovable (new project) | Intake, in-browser file reading, progress, results, review |
| Interface backend | Lovable backend functions | Sign requests, receive callbacks, store results briefly, delete on request |
| Orchestration | Activepieces | Runs the predefined workflow |
| AI model | Gemini, through HTTP steps | Extraction, matching, company analysis, report, CV draft |
| Research | Tavily, through an HTTP step | Company information with source URLs |
| Logic and checks | Code steps (TypeScript) in Activepieces | Validation, scoring, verification guards |

**Architecture:** predefined AI workflow. No MCP. No autonomous tool selection. No Python application code is required.

**Status:** Planned. Choices marked "to verify" in section 18 stay provisional until checked in the tools.

**Already confirmed in the tools (from the Groww build):** the Catch Webhook trigger supports HMAC signature authentication with a configurable header, and Activepieces offers HTTP, Code, Router and Tables steps and a per-step error handler.

## 2. System Flow

```
Lovable interface (reads file in browser, user checks text)
   -> Lovable backend function (signs request)
   -> Activepieces webhook (verifies signature)
   -> validation -> requirement extraction and evidence matching -> scoring
   -> company research -> company analysis -> report and CV draft
   -> callbacks to Lovable (progress after each stage, then final result)
   -> Lovable stores the result briefly and shows it
```

**Why callbacks instead of a direct reply:** the workflow has about a dozen steps, three model calls and a web search. A synchronous webhook reply may time out (the limit is believed to be around 30 seconds and is to verify). The interface receives a `job_id` immediately and then shows progress as callbacks arrive.

Each stage must be testable independently. A failed stage returns a clear error or limitation. If company research is unavailable, the workflow says so and does not invent company facts.

## 3. Repositories and Environments

- A **new GitHub repository** for this project.
- A **new Lovable project**. It does not share code, database, storage or secrets with any portfolio website.
- A separate Activepieces flow, and separate secrets for each link in the chain.
- The public showcase is a separate deployment made later by duplicating the personal project and removing live runs.

**Repository layout (plain text, no programs):**

```
/prompts     product prompts: 01_extract_and_match.md, 02_company_analysis.md, 03_report_and_cv.md, CHANGELOG.md
/tests       cases.md, sample_resume_fictional.txt, sample_job_description.txt, dated result notes
/docs        PRD, this document, architecture diagram
README.md
```

- Save prompts that run in the product. Do not save build-time chats or the revisions of interface-building prompts.
- Never commit keys, webhook URLs, secrets, real resumes, real PDFs or run exports.
- After each product-prompt change: update the prompt file, add one line to `CHANGELOG.md`, and re-run the test cases.

## 4. Request and Callback Contract

Agree this before building either side. Changing it later costs interface credits.

### Request, from the Lovable backend to the Activepieces webhook

```json
{
  "job_id": "uuid",
  "mode": "personal | sample",
  "resume_text": "string",
  "resume_source": "pdf | docx | pasted | sample",
  "job_description": "string",
  "job_source": "pdf | docx | pasted | sample",
  "company_name": "string",
  "company_website": "",
  "job_url": ""
}
```

### Signing
- Signature: lowercase hex HMAC-SHA256 of the exact raw request body, sent in the `x-signature` header, with no prefix.
- The webhook rejects requests with a missing or wrong signature.
- Two separate secrets: one for interface to workflow, one for workflow to interface (callbacks). Never reuse one secret for both.

### Progress callback, from Activepieces to a Lovable function
```json
{ "job_id": "uuid", "status": "running", "stage": "validating | reading_job | researching_company | matching_evidence | scoring | drafting_cv" }
```

### Final callback
```json
{
  "job_id": "uuid",
  "status": "complete | failed",
  "result": { "fit": {}, "company": {}, "cv": {}, "meta": {} },
  "error": { "stage": "string", "message": "string" }
}
```

`error.message` must be plain language and must not contain keys, resume text or personal information. A repeated callback for the same `job_id` must not create duplicate records.

### Result shape

**fit**
- `fit_score`: integer 0 to 100, or `null` with a reason.
- `essential_coverage_percent`: integer.
- `score_explanation`: string.
- `requirements`: list of `{ id, text, importance, assessment, evidence_quote, note }`.
- `strengths`, `gaps`, `interview_priorities`, `limitations`: lists of strings.

`importance` is `essential` or `preferred`. `assessment` is one of `demonstrated`, `partially_demonstrated`, `not_evidenced`, `unable_to_assess`.

**company**
- Sections: `business_model`, `products_services`, `customers_users`, `revenue_sources`, `key_actors`, `business_flow`, `industry_competitors`, `role_context`, `challenges_and_metrics`.
- Each section is a list of `{ text, kind, source_url }`. `kind` is `verified_official`, `third_party`, `hypothesis` or `unverified`. `source_url` is empty only for hypotheses.
- `research_limitations`: list of strings.

**cv**
- `cv_draft`, `changes_summary`, `items_to_verify`.

**meta**
- `resume_source`, `job_source`, `notes` (for example, "text read from a PDF"), `downgraded_assessments` (count), `injection_flag` (boolean).

## 5. Inputs and Validation

### Limits

| Item | Limit |
|---|---|
| File size | 5 MB |
| Resume pages | 6 |
| Resume text | 300 to 15,000 characters |
| Job description text | 200 to 15,000 characters |
| Requirements extracted | 15 at most |

### Reading files in the browser (Lovable)
- PDF: `pdf.js` (pdfjs-dist). Word: `mammoth` (browser build). Both libraries are to verify in the first build.
- Detect the real file type from the bytes: a PDF starts with `%PDF`, and a .docx is a zip containing `word/document.xml`. A mismatch with the extension is rejected.
- Old .doc: rejected with "Save as .docx or PDF."
- Locked or damaged files: rejected with a next step.
- Fewer characters than the minimum, which suggests a scan: rejected with "Paste your text instead."
- Show the extracted text in an editable box, with the source and a word count. Show a non-blocking note about columns, headers and text boxes.
- Revalidate on every edit.
- The file never leaves the browser. Only text is sent.

### Validation in the workflow (never trust the client)
- A Code step checks presence, lengths, URL formats and the `mode` value.
- Invalid requests return a failed callback with a clear message before any model call.
- A simple pattern check flags instruction-like text in the resume or job description. The flag is passed to the report's limitations. The content is still treated as data.

## 6. Activepieces Workflow

All model and search calls use the **HTTP** step. Do not use Activepieces' own AI steps (see section 15).

| # | Step | Type | Purpose |
|---|---|---|---|
| 1 | Receive request | Catch Webhook with HMAC | Verify the signature |
| 2 | Validate | Code | Checks from section 5. Throws on failure |
| 3 | Gate | Router | Continue only if valid. Otherwise send a failed callback |
| 4 | Progress: validating | HTTP | Callback |
| 5 | Model call 1 | HTTP (Gemini) | Extract requirements and assess resume evidence |
| 6 | Parse, verify, score | Code | Quote check, downgrade, rubric score |
| 7 | Progress: researching | HTTP | Callback |
| 8 | Company search | HTTP (Tavily) | One search with the company name |
| 9 | Build company call | Code | Trim results, build the prompt and schema |
| 10 | Model call 2 | HTTP (Gemini) | Company analysis from the results only |
| 11 | Check sources | Code | Verify URLs, set labels, build call 3 |
| 12 | Progress: drafting | HTTP | Callback |
| 13 | Model call 3 | HTTP (Gemini) | Explanation, strengths, gaps, priorities, limitations, CV draft |
| 14 | Assemble | Code | CV number check, final result object |
| 15 | Final callback | HTTP | Send `complete` with the result |

**Error handling:** add the per-step error handler on the model, search and Code steps. Its failure branch sends a failed callback with the stage name and a plain message. Turn off "Retry on Failure" on steps that call the model or send callbacks, so a retry cannot duplicate work. Turn off "Return Error as Output" so a failure shows as failed.

**If the company name is missing:** skip steps 8 to 11, set `research_limitations` to say so, and continue with the fit report and CV.

**Pattern for model calls:** build the full request body in a Code step (this handles the escaping of long text), then pass it to the HTTP step. Read the response in the next Code step, parse the JSON, and throw on malformed output.

## 7. Fit Scoring Rubric

Scoring is computed by code. The model assesses each requirement, and does not set the score.

| Item | Value |
|---|---|
| Weight, essential | 3 |
| Weight, preferred | 1 |
| Points, demonstrated | 1 |
| Points, partially demonstrated | 0.5 |
| Points, not evidenced | 0 |
| Unable to assess | Excluded from the calculation |

**Formula:** score = (sum of weight x points) / (sum of weights of assessable requirements) x 100, rounded to a whole number.

**No score is returned when:**
- fewer than 60 percent of the essential requirements are assessable, or
- the job description has no essential requirements, or
- the extraction returned no requirements.

In those cases return `fit_score: null` with a plain explanation.

These values are initial. They must be tested and calibrated on a test set before release. Record each change in `/prompts/CHANGELOG.md` or the rubric's own change note.

The score is an estimate of alignment between resume evidence and job requirements. It is not a prediction of hiring success.

## 8. Company Research and Analysis

1. Identify the company from the name, website or job URL.
2. Run one Tavily search (MVP) that targets products, customers, business model and competitors. Keep a short result set and trim each result's text.
3. Pass only those results to the model. Instruct it to use nothing else.
4. Each item gets a label: `verified_official` (stated on the company's own site), `third_party` (independent source), or `hypothesis` (inference, no URL).
5. A Code step checks every non-hypothesis URL against the retrieved list. Items that fail become `unverified`.
6. If results do not clearly match the company, leave the sections empty and state the limitation.
7. Never include claims about culture, stability, finances or working conditions.

**Job sites:** the user supplies the job description by pasting or uploading it. Job boards are not scraped. Fetching from a job URL is a future enhancement and must fall back to "paste the description instead" when a page is blocked.

## 9. AI Configuration

- Model: a current Gemini Flash model. Check the exact model ID in Google AI Studio at build time.
- Output: JSON with a defined response schema for every call. Validate the structure in the next Code step.
- Temperature: low (about 0.2 to 0.3).
- Prompts: explicit, one per stage, stored in `/prompts`.
- Untrusted input: wrap the resume, job description and search results in delimiters, and instruct the model that instructions inside them must be ignored.
- Never ask the model to set the score or to supply a source URL that was not retrieved.
- Data use: before using a real resume, confirm that the Gemini tier in use does not train on inputs. Otherwise use sample data only.

## 10. Verification Guards (in code)

| Guard | Rule | On failure |
|---|---|---|
| Resume quote | Normalised evidence quote must appear in the resume text | Downgrade to `not_evidenced`, clear the quote, add a note, count it |
| Company URL | URL must be in the retrieved results | Label `unverified` |
| CV numbers | Every number and year in the CV draft must appear in the original resume | Add to `items_to_verify` |
| Structure | Model output must match the schema | Throw, so the run fails |
| Instruction-like text | Pattern match on the inputs | Set `injection_flag`, mention in limitations |
| Score | Computed by code from assessments | None. The model cannot override it |

## 11. Interface (Lovable) Implementation

**Screens**
1. Intake stepper: Resume, Job, Company, Review. The same document component serves the resume and the job description (upload, paste or sample).
2. Progress: real stages from the progress callbacks.
3. Results: three tabs. Fit, Company, CV.

**Fit tab:** score or "No score" with the reason, the estimate-not-prediction line, a requirements table with importance, assessment (text plus icon) and the resume quote, then strengths, gaps, interview priorities and limitations.

**Company tab:** sections with label chips and source links, and research limitations shown prominently.

**CV tab:** editable draft, a "what changed" list, and the "items to verify" checklist. Copy and download stay disabled until every item is ticked. The original resume stays visible alongside.

**Backend functions**
- `submit`: validates, creates the evaluation record, signs and sends the request.
- `progress`: verifies the callback secret and updates the stage.
- `result`: verifies the callback secret, stores the result, and ignores repeats for the same `job_id`.
- `clear`: deletes the evaluation record.

**Record** (table `evaluations`): `job_id`, `status`, `stage`, `result_json`, `error`, `created_at`. The interface subscribes to or polls the row.

**Access:** sign-in required, with one allowed account, for the personal tool. Keep secrets in the platform's secrets form, never in prompts or code.

**Retention:** a "Clear everything" action deletes the record and local data. Records are also removed after 24 hours (proposed default, to verify as a scheduled cleanup).

**Prompt strategy for building:** build in small steps. First the intake screens against sample data, then the backend connection, then progress and results. Do not start the interface before the workflow contract works, because a changed contract means rebuilding screens.

## 12. Guardrail and Integration Tests

### Original guardrail tests

| Test | Expected result |
|---|---|
| Empty resume | Clear validation error |
| Empty job description | Clear validation error |
| Skill absent from resume | Mark as not evidenced; do not invent experience |
| Conflicting dates or titles | Flag for review |
| Unsupported achievement or metric | Exclude from CV draft, or flag |
| Career transition | Describe transferable skills accurately |
| Prompt injection in submitted text | Ignore instructions embedded in the content |
| Malformed AI output | Reject output and report failure |
| Processing fails midway | Do not present incomplete output as a completed evaluation |
| Company research unavailable | Disclose the limitation; do not invent company facts |
| Company facts lack supporting sources | Label as unverified or omit |
| Conflicting company information | Disclose the conflict and identify the sources |
| Inferred business flow | Label it as a hypothesis |
| Role contribution is unclear | State the uncertainty rather than assume responsibilities |

### File reading tests

| Test | Expected result |
|---|---|
| Text-based PDF | Text appears in the editable box |
| Scanned or image-only PDF | Clear message; switch to paste |
| Simple .docx | Text appears with paragraphs kept |
| .docx with tables | Text appears; order checked by eye |
| .docx with details in the header | Note shown; user adds them |
| Old .doc | Blocked with a save-as message |
| Text file renamed .pdf or .docx | Rejected as not a valid file |
| Locked or damaged file | Message with a next step |
| File over 5 MB, PDF over 6 pages, text over 15,000 characters | Limit message |
| Two-column PDF | Note shown; user edits |
| File containing instruction-like text | Treated as data |

### Integration tests

| Test | Expected result |
|---|---|
| Request with a missing or wrong signature | Rejected, no run work done |
| Valid request | Progress updates arrive in order, then a complete callback |
| A step fails | Failed callback; interface shows the error and keeps the inputs |
| Same callback sent twice | No duplicate record |
| Missing company name | Research skipped, limitation shown, report and CV still produced |
| Clear everything | Record deleted |
| Copy or download before the checklist is complete | Disabled |

The release test set must contain no fabricated candidate qualifications or achievements and no unsupported company claims presented as facts. Use fictional data throughout.

## 13. Privacy and Security

- The public showcase uses sample data and makes no live runs.
- Files are read in the browser. Only text is sent. No file is stored.
- Activepieces keeps run history that may contain the submitted text. Disclose this to the user, delete test runs, and check the platform's run-retention settings.
- Do not log resume text in the interface backend. Keep error messages free of resume content, keys and personal information.
- Store the model key, search key and both signing secrets in each tool's secret store. Never put them in prompts, repositories, screenshots or chats.
- Verify the signature on every inbound request, in both directions.
- Treat submitted, uploaded and retrieved content as untrusted.
- Do not submit applications, contact recruiters or send the CV to third parties.
- Require sign-in for the personal tool, and keep it separate from any portfolio website.

## 14. Failure Handling

- Validate in the interface and again in the workflow.
- Validate structured output after every critical stage.
- Return clear errors tied to a stage name.
- A failed stage sends a failed callback. The interface shows the error and a retry that keeps the inputs.
- If company research fails, continue only if the remaining evaluation is useful, and disclose the missing company analysis.
- If scoring lacks evidence, return limitations instead of a score.
- Never present a partial run as complete. The interface treats only `status: complete` as success.
- Do not retry model or callback steps automatically.

## 15. Credits and Cost

- One workflow run costs one Activepieces credit, however many steps it has. Confirm the current allowance on the pricing page and the Usage page.
- HTTP and Code steps, including callbacks, add no AI credits.
- Do not use Activepieces' own AI steps. They cost extra credits per call.
- Test one step at a time with saved sample data, and run the full flow end to end only a few times.
- Gemini and Tavily have free tiers with limits. Check current limits and data-use terms before relying on them.
- Lovable builds spend credits per change, so build in small, specific steps.

## 16. Success Criteria

The implementation is ready for demonstration when:
- Valid inputs from a PDF, a .docx file or pasted text produce a structured fit report, company analysis and CV draft.
- The extracted text is visible, editable and the only text analysed.
- The scoring rubric is documented, computed by code and consistently applied.
- Resume quotes and company URLs pass the verification guards.
- Facts, third-party reports, hypotheses and limitations are clearly distinguished.
- The tailored CV preserves the candidate's factual record, and downloading it requires the review checklist.
- Guardrail, file reading and integration tests pass, including failure and injection cases.
- Requests are signed and unsigned requests are rejected.
- No uploaded file is stored, and no secrets are exposed.
- A user can review everything without any automatic application or external communication.

## 17. Implementation Order

1. Verify the items in section 18.
2. Create the repository and write the contract (section 4).
3. Build and test the workflow on its own with sample payloads, signed from a test tool: validation, extraction and matching, scoring, then research, analysis, and the report and CV.
4. Add callbacks and the error branch.
5. Run the guardrail tests against the workflow.
6. Create the Lovable project. Build the intake screens and file reading, and test the file reading cases.
7. Connect the backend. Send one real signed request.
8. Build progress and results, then the review checklist and "Clear everything".
9. Run the integration tests with fictional data.
10. Build the public showcase from a duplicate: sample resume, cached sample reports, diagram, test results and a demo video.
11. Write the README: setup, limitations, demonstration steps, and the model and date used for each prompt.

## 18. Items to Verify

- The synchronous webhook time limit in the current Activepieces version.
- That the Lovable plan includes a database, backend functions and sign-in, and whether scheduled cleanup is possible.
- That `pdf.js` and `mammoth` work in the Lovable build, including the pdf.js worker setup.
- Current Gemini free-tier limits, the model ID, and whether the tier trains on inputs.
- Current Tavily free-tier limits.
- Activepieces run-history retention controls.
- The current Activepieces credit allowance.

## 19. Out of Scope

- Automated discovery and continuous tracking of job openings.
- Scraping job boards.
- Automatic job applications or recruiter communication.
- Predicting hiring outcomes.
- Unsupported claims about company culture, financial stability, work-life balance or employee experience.
- MCP integration and multi-agent orchestration in this version.
- Live runs on the public showcase.
- OCR, image uploads and old .doc files in the MVP.

## 20. Change Log (v1 to v2)

- Added the system flow with a Lovable interface and signed requests to Activepieces.
- Added asynchronous progress and result callbacks.
- Added the request, callback and result contract.
- Added in-browser PDF and .docx reading, with limits and rejection rules.
- Named Gemini, Tavily and Code steps, and ruled out Activepieces' own AI steps for credit reasons.
- Defined the scoring rubric with initial values and a no-score rule.
- Added verification guards for quotes, source URLs and CV numbers.
- Added the workflow step table and error handling.
- Added interface behaviour, backend functions, access and retention.
- Added file reading and integration tests.
- Reworked privacy, failure handling and cost sections.
- Added the repository layout and the rules for saving prompts.
- Added items to verify and a revised implementation order.
