# Product Requirements Document (PRD)
## Project: Job Fit Evaluator

### 1. Executive Summary & Purpose
The **Job Fit Evaluator** is an AI-powered web application designed to help job seekers evaluate how well their background aligns with a target Job Description (JD). Beyond basic keyword matching, the platform performs real-time market and company intelligence to break down the employer's business model, tech stack, and user flows, empowering candidates with actionable resume enhancements and high-context interview preparation.

---

### 2. Core Value Proposition
- **High-Context Fit Analysis:** Evaluates candidates against both stated job requirements and unstated company context.
- **Deep Company Intelligence:** Uncovers the target company's business model, customer journeys, tech stack, and key actors so candidates understand *why* their skills matter.
- **Actionable CV Enhancements:** Generates ready-to-use resume bullet rewrites tailored to the company's specific product domain.
- **Privacy First:** Raw resumes and documents are parsed entirely in the browser; only sanitized text leaves the user's device, and data is ephemeral with a 24-hour expiration or instant purge.

---

### 3. User Personas
- **Active Job Seekers:** Need fast, rigorous feedback on whether their resume will pass initial screening.
- **Career Switchers:** Need to translate existing experience into the target company's terminology and user domain.
- **Interview Preppers:** Need a quick briefing on the company's tech stack, business model, and operational actors to speak with authority during interviews.

---

### 4. Key Functional Features

#### 4.1. Privacy-First Intake Flow (4 Steps)
1. **Resume Input:**
   - Upload PDF, DOCX, or TXT file, or paste raw text.
   - In-browser file parsing via client-side libraries (`pdf.js` / `mammoth`). Raw files are never uploaded.
   - Live character/word count and text preview with editable fields.
2. **Job Description Input:**
   - Paste job description text or use pre-loaded sample job descriptions.
   - Validation against minimum and maximum character limits.
3. **Company Context:**
   - Target Company Name (required).
   - Company Website / Domain (optional, boosts research precision).
   - Direct Job Posting URL (optional).
4. **Review & Confirm:**
   - Summary of inputs before running evaluation.
   - Privacy guarantee reminder and instant "Clear Everything" button.

#### 4.2. Deep Company Intelligence (via Tavily Search API)
The application conducts automated web research on the target company to extract:
- **Tech Stack & Tooling:** Core frameworks, languages, infrastructure, and third-party tools used by the engineering or product teams.
- **Business Model:** Value proposition, monetization strategy (B2B SaaS, marketplace, subscription, enterprise), and target audience.
- **User Flows & Journeys:** Primary end-to-end user paths (e.g., onboarding, checkout, workflow automation).
- **Actors & Personas:** Key internal and external stakeholders (e.g., buyers, end-users, admins, partners).
- **Core Actions & Features:** Critical actions users take within the company's product or service.

#### 4.3. Multi-Dimensional AI Fit Evaluation (via Lovable AI Gateway)
- **Weighted Overall Fit Score (0–100%):**
  - Technical / Hard Skill Match (40%)
  - Relevant Experience & Scope Match (35%)
  - Company Domain & Product Alignment (25%)
- **Skill Gap Analysis:** Categorized as *Must-Have Matches*, *Nice-to-Have Matches*, and *Critical Missing Gaps* with specific evidence references from the resume and JD.
- **ATS Keyword Alignment:** Visual tags showing matched vs. missing domain terms and industry keywords.
- **Tailored Resume Enhancements:** 3–5 bullet-point rewrites showing how to frame past accomplishments around the target company’s actors, metrics, and business model.
- **Interview Talking Points:** High-impact conversational prompts demonstrating deep domain knowledge of the company's user flows and business challenges.

#### 4.4. Calm, Professional Results Dashboard
- **Fit Overview Tab:** High-level gauge visualization, fit score breakdown, executive summary, and top gaps.
- **Company Deep Dive Tab:** Structured cards detailing Tech Stack, Business Model, User Flows, and Key Actors.
- **CV Action Plan Tab:** Side-by-side comparison of original vs. improved bullet points with one-click copy.
- **Data Lifecycle Controls:** Automatic 24-hour expiration or immediate client-triggered purge.

---

### 5. Non-Functional Requirements
- **Privacy & Security:** Zero persistent storage of personal identifiers. Client-side document parsing.
- **Performance:** End-to-end evaluation completed within 15–25 seconds with real-time stage progress indicators (`Researching Company` → `Analyzing Job Fit` → `Generating Action Plan`).
- **Reliability:** Graceful fallbacks if company website information is sparse or search queries return partial results.
