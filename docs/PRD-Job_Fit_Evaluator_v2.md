# PRD — Job Fit Evaluator

**Version:** 2.0 (30 October 2026). Supersedes v1. A change log is at the end.

## 1. Overview

**Purpose:** Help candidates assess role fit, understand the hiring company's business, and create a tailored CV draft without overstating their qualifications.

**Modes:** Two, kept separate (see section 4).
- **Personal tool:** private, for real resumes and real jobs.
- **Public showcase:** sample data only, for recruiters and portfolio visitors.

**Platform:**
- **Orchestration:** Activepieces.
- **Interface:** Lovable, in a new project that is separate from any portfolio website.
- **AI model:** Gemini, called from the workflow.
- **Company research:** Tavily search, called from the workflow.

**Architecture:** AI-powered predefined workflow. No MCP, no autonomous tool selection.

## 2. Problem

Candidates often struggle to assess their fit against job requirements, to understand the company's business and product context, and to tailor their CVs without overstating their qualifications.

## 3. Target Users

Candidates evaluating their suitability for a role, including those transitioning careers. The first user is the product's author.

## 4. Product Modes

| | Personal tool | Public showcase |
|---|---|---|
| Audience | The author only | Recruiters, portfolio visitors |
| Data | Real resumes and job descriptions | Fictional sample resume and sample jobs |
| Live runs | Yes | No. Cached sample reports only |
| Sign-in | Required (single allowed account) | Not needed |
| Keys and credits | Used | Never exposed |

The public showcase is a separate deployment. It cannot reach private data, the private workflow, or the private Job Hunt Intelligence agent.

## 5. MVP Workflow

### Inputs

| Input | Required | Details |
|---|---|---|
| Resume | Yes | PDF, Word (.docx), pasted text, or a sample resume |
| Job description | Yes | Pasted text, PDF or .docx upload, or a sample |
| Company name | Yes in the interface | If the workflow receives none, it skips research and states the limitation |
| Company website | Optional | Helps identify the company |
| Job URL | Optional | A hint only. The page is not fetched in the MVP |

**File handling**
- Files are read in the user's browser. Only the extracted text is sent for analysis. The file itself is never uploaded or stored.
- The user sees the extracted text in an editable box and can correct it before anything runs.
- Limits: 5 MB per file, 6 pages for a resume, 300 to 15,000 characters for a resume, 200 to 15,000 characters for a job description.
- Not supported in the MVP: old Word (.doc), scanned or image-only PDFs, and images such as PNG. Each gets a clear message that tells the user what to do instead.

### Process

1. Read the resume and job description, and let the user check the extracted text.
2. Validate inputs.
3. Extract essential and preferred job requirements.
4. Research the company using reliable, cited sources.
5. Analyse its business model, products, customers, key actors, value flows and competitive context.
6. Match job requirements against resume evidence.
7. Score role fit using the documented rubric.
8. Identify strengths, gaps and interview preparation priorities.
9. Generate a company and role context report.
10. Generate a tailored CV draft with items to verify.
11. User reviews and edits the results.

### Outputs

**Role fit report**
- Fit score with explanation, or an explanation of why no score was given.
- Requirements, each with importance, assessment and the supporting resume quote.
- Strengths, gaps, interview preparation priorities and limitations.

**Company analysis**
- Business model and revenue sources, where verifiable.
- Products and services, target customers and users.
- Key actors, including relevant partners and stakeholders.
- Core business and product flows: how value is created, delivered and monetised.
- Industry, competitors and market position.
- Likely product challenges and relevant business metrics, labelled as hypotheses where inferred.
- Sources, evidence gaps and information requiring verification.

**CV draft**
- Editable, role-tailored CV.
- Relevant experience prioritised for the target role.
- Summary of material changes and items requiring user review.

## 6. User Experience Requirements

User experience is a first-class requirement, not decoration.

- **Guided intake:** a short stepper (Resume, Job, Company, Review) with sample data available at each step.
- **Visible extraction:** the user always sees, and can edit, the text that will be analysed.
- **Early, specific errors:** every failure explains the problem and the next action (for example, "This PDF looks like a scan. Paste your text instead.").
- **Honest progress:** real stage updates (Validating, Reading the job, Researching the company, Matching evidence, Scoring, Drafting the CV). No fake progress bar.
- **Clear results:** three tabs (Fit, Company, CV). Assessments use text plus an icon, never colour alone.
- **Visible evidence and labels:** each requirement shows the resume quote behind it. Each company item shows its label: Verified, Third-party, Hypothesis or Unverified.
- **Limits stated up front:** the score is described as an estimate of alignment, not a hiring prediction, and limitations are shown in the main view.
- **Review before use:** copying or downloading the CV draft unlocks only after the user ticks each item in the "items to verify" list.
- **Control over data:** a "Clear everything" action, and a plain statement of what is sent and kept.
- **Accessibility and devices:** keyboard accessible, readable on mobile, plain language.

## 7. Evaluation Rules

- Classify requirements as essential or preferred.
- Assess each requirement as demonstrated, partially demonstrated, not evidenced, or unable to assess.
- Weight requirements with the documented rubric (see the Implementation document, section 7).
- Link every assessment to a specific resume quote.
- Distinguish missing evidence from a confirmed skills gap.
- Treat the score as an estimate of role alignment, not hiring probability.
- Label company information as Verified (official source), Third-party, Hypothesis (AI inference) or Unverified (no supporting source).
- Cite a source URL for every non-hypothesis company claim, and disclose unavailable or uncertain information.
- Do not present inferred internal workflows, team responsibilities or product challenges as confirmed facts.

## 8. Guardrails

### Accuracy
- Use resume evidence for candidate assessments and reliable sources for company research.
- Never invent skills, qualifications, experience, achievements or metrics.
- Do not imply Product Management experience or ownership unless the evidence supports it.
- Flag ambiguous or conflicting information for review.
- Do not treat missing resume information as proof that a candidate lacks a skill.
- Do not assume public company information reveals internal team processes.

### Verification by the system, not only by the model
- An assessment of demonstrated or partially demonstrated is kept only if its quote appears in the resume. Otherwise it is downgraded to not evidenced and flagged.
- A company claim keeps its Verified or Third-party label only if its URL was in the retrieved search results. Otherwise it is labelled Unverified.
- Numbers and years in the CV draft must exist in the original resume. Others are flagged for review.
- Model outputs are checked against a defined structure. Malformed output fails the run.

### Fit Scoring
- Apply a documented, consistent scoring rubric.
- Do not present the score as a prediction of hiring success.
- Do not assign a score when required inputs are missing, unusable or too thin to assess.
- Disclose insufficient evidence and assessment limitations.

### Company Analysis
- Prefer official company sources for products, business model and stated strategy.
- Use credible independent sources for industry and competitor context.
- Distinguish verified facts, employee-review signals and AI inference.
- Do not make unsupported claims about company culture, stability, financial health or working conditions.
- Do not scrape job boards or other sites whose terms prohibit it.

### CV Generation
- Preserve the original resume.
- Generate a separate, editable draft.
- Include keywords only when supported by the candidate's experience.
- Do not alter dates, job titles, qualifications or achievements without evidence.
- Require user review before the CV is used.

### Privacy and Security
- Use sample data for the public showcase.
- Read uploaded files in the browser. Never store the file.
- Do not retain resume content by default beyond a short, disclosed period (see section 10).
- Avoid logging personal information.
- Never expose API keys, credentials, webhook secrets or submitted resume content.
- Keep the public showcase separate from the private tool and from the private Job Hunt Intelligence agent.

### User Control
- Do not submit applications or contact employers.
- Allow users to review and edit generated content.
- Clearly disclose incomplete evaluations and uncertain findings.
- Do not present AI assessments as definitive hiring decisions.

### Failure Handling
- Validate inputs before processing, in the interface and again in the workflow.
- Reject missing or unusable required inputs with a clear explanation.
- If company information cannot be verified, disclose the limitation rather than fabricate an analysis.
- Report processing failures and allow retries that keep the user's inputs.
- Never present an incomplete or failed evaluation as successful.

## 9. Architecture

**Flow:** Lovable interface → signed request → Activepieces workflow → progress and result callbacks → Lovable interface.

Workflow stages: Input Validation → Requirement Extraction and Evidence Matching → Fit Scoring → Company Research → Company Analysis → Report and CV Draft → User Review.

| Layer | Tool | Role |
|---|---|---|
| Interface | Lovable (new, separate project) | Intake, file reading in the browser, progress, results, review |
| Interface backend | Lovable backend functions | Sign requests, receive callbacks, store results briefly, delete on request |
| Orchestration | Activepieces | Runs the predefined workflow |
| AI model | Gemini (HTTP calls) | Requirement extraction, evidence comparison, company analysis, report and CV drafting |
| Research | Tavily (HTTP call) | Company information with source URLs |
| Logic and checks | Code steps in Activepieces | Validation, scoring, verification guards |

No Python application code is required. MCP is out of scope.

## 10. Privacy

- Public showcase: fictional sample profiles only, no live runs.
- Personal tool: sign-in required, a single allowed account.
- Files are read in the browser. Only text is sent.
- Resume and job text pass through the workflow and may appear in the workflow's run history. This is disclosed to the user, and test runs are deleted.
- Evaluation results are held in a private record only until the user clicks "Clear everything" or 24 hours pass, whichever is first. The 24-hour period is a proposed default and needs confirming.
- Do not log resume text or unnecessary personal information in the interface backend.
- Store credentials in each tool's supported secret store. Never put them in prompts, source files or repositories.
- Use a Gemini tier whose terms exclude training on inputs before using a real resume, or use sample data only.
- Use public company information for research. Do not seek private or unauthorised internal information.
- Keep this project's accounts, repository, database and secrets separate from any portfolio website.

## 11. Success Metrics

### Functional
- Workflow completion rate.
- Successful CV draft generation rate.
- Successful company report generation rate.
- Input validation and processing failure rate.
- File reading success rate by format (PDF, .docx).

### Quality
- Agreement between fit assessments and human reviewers.
- Accuracy of resume evidence references.
- Accuracy and traceability of company facts.
- Unsupported-claim rate in generated CVs.
- User-rated usefulness of the fit and company reports.
- Share of users who correct the extracted text (a signal of extraction quality).

### Guardrails
- Zero fabricated qualifications or achievements in the release test set.
- No unsupported company claims presented as verified facts.
- No unauthorised external sharing or application submission.
- No exposed secrets or private sample data.
- No uploaded file stored.

## 12. Acceptance Criteria

The MVP is ready for demonstration when:

1. Users can provide a resume as a PDF, a .docx file, pasted text or a sample, and a job description as pasted text or a file.
2. The extracted text is shown, editable, and the only content that is analysed.
3. Unreadable, scanned, legacy .doc, oversized and invalid files are rejected with a clear next step.
4. Company research is performed when sufficient company information and supported sources are available, and is skipped with a stated limitation when not.
5. The workflow produces a structured fit report and company analysis.
6. Candidate assessments reference resume quotes or identify insufficient evidence, and quotes are verified against the resume.
7. The fit score follows the documented rubric and is computed by code, not by the model.
8. Company claims include sources or are labelled as hypotheses, and source URLs are verified against retrieved results.
9. The CV draft contains no unsupported claims in the test set.
10. The original resume remains unchanged.
11. Users can review and edit the generated CV, and copy or download unlocks only after the review checklist is complete.
12. Requests to the workflow are signed, and unsigned requests are rejected.
13. Missing inputs, unavailable sources, malformed model output and processing failures are handled clearly, and a failed run is never shown as complete.
14. Progress shows real stages.
15. Tests cover incomplete resumes, conflicting information, unsupported skills, missing job requirements, unverified company claims, file-reading failures and prompt injection.
16. Sample data, personal information and credentials are protected, and the public showcase makes no live runs.
17. Known limitations are documented (section 13).

## 13. Known Limitations

- Scanned or image-only PDFs cannot be read. The user pastes text instead.
- Multi-column PDFs can scramble line order. Headers, footers and text boxes in .docx files can be missed. The editable text box exists for this reason.
- Old .doc files and images (including PNG) are not supported in the MVP.
- Quote verification checks that evidence appears in the resume. It does not prove the evidence supports the requirement. The model judges that, and a human reviews it.
- The fit score is an estimate that depends on the job description's wording and on rubric values that have not yet been calibrated on a large test set.
- Company analysis depends on what a single search returns. Thin or conflicting results produce limitations, not a fuller story.
- Evaluations take one to two minutes and depend on third-party services (workflow platform, model, search).

## 14. Out of Scope

- Automatic job applications or recruiter outreach.
- Automated discovery and ongoing tracking of job openings.
- Scraping job boards.
- Comprehensive company due diligence or investment analysis.
- Access to private company systems or internal information.
- MCP integration and multi-agent orchestration.
- Hiring outcome predictions.
- Live runs on the public showcase.
- Mixing this project with a portfolio website's code, database or storage.

## 15. Future Enhancements

- Downloadable CV as .docx.
- OCR for scanned PDFs and images, read in the browser.
- Job description retrieval from a URL, with a paste fallback when a page is blocked.
- Broader research sources, subject to each source's terms of use.
- Improved score calibration using a larger test set.
- Comparison of multiple job descriptions and companies.
- Deeper research into culture, work-life balance, compensation and company stability using reliable evidence.
- A scheduled cleanup of stored results, and a configurable retention period.

These will be prioritised after the MVP and its research capabilities are tested.

## 16. Change Log (v1 to v2)

- Added two separate modes: personal tool and public showcase.
- Added Lovable as the interface, in a new project separate from any portfolio.
- Moved PDF and .docx upload from future enhancements into the MVP, with files read in the browser.
- Added user experience requirements (section 6).
- Named the tools: Gemini for AI, Tavily for research, Code steps for checks.
- Added system-side verification guards (quotes, source URLs, CV numbers).
- Required signed requests to the workflow.
- Made the company name required in the interface.
- Reworked privacy: browser-side file reading, disclosed run history, short retention, a Clear everything action.
- Removed "automated company research from a company name" from future enhancements, since it is in the MVP.
- Added known limitations, extra acceptance criteria and extra success metrics.
- Added out-of-scope items: job board scraping, live public runs, mixing with a portfolio.
