# Job Fit Evaluator 🎯

An AI-powered evaluation platform that analyzes candidate resumes against specific Job Descriptions (JDs), deep-dives into employer context via real-time web intelligence, and produces actionable fit scores and tailored CV rewrites.

![Job Fit Evaluator Preview](https://raw.githubusercontent.com/pmradha/job-fit-evaluator/main/public/favicon.ico) <!-- Replace with preview screenshot when available -->

---

## 🌟 Key Features

- **Privacy-First In-Browser Parsing:** Resumes (PDF, DOCX, TXT) are read directly inside the user's browser. Raw files are never uploaded to servers or stored.
- **Deep Company Intelligence (Tavily Search API):** Automatically extracts the target company's engineering tech stack, business model, primary user flows, and core customer actors.
- **Multi-Dimensional Fit Scoring:** Evaluates technical competencies, relevant scope of experience, and company domain alignment using AI.
- **Actionable CV Enhancements:** Generates ready-to-use resume bullet point rewrites aligned with the employer's specific product workflows and metrics.
- **Ephemeral Data Lifecycle:** All evaluation data expires automatically after 24 hours or can be purged instantly by the user.

---

## 🏗️ Architecture & Tech Stack

```text
Browser (React 19 / TanStack Start)
   │  • 4-step intake (Resume, JD, Company, Review)
   │  • Client-side document parsing (pdf.js / mammoth)
   ▼
Lovable Cloud Server Functions
   ├── 1. Validation & sanitization
   ├── 2. Tavily Search API (Tech stack, business model, actors, user flows)
   ├── 3. Lovable AI Gateway (Scoring, gap analysis, bullet rewrites)
   └── 4. PostgreSQL Database (Ephemeral state & 24h TTL)
   ▼
Interactive Results Dashboard
   • Match score gauges & must-have vs. nice-to-have skill breakdown
   • Company intelligence briefing cards
   • Tailored CV rewrites & interview talking points
