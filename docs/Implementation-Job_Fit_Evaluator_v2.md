# Technical Implementation Document
## Project: Job Fit Evaluator

### 1. Technology Architecture

```text
Browser (React 19 / TanStack Start)
  ├── 4-Step Stepper (Resume -> Job -> Company -> Review)
  └── Local File Extraction (pdf.js / mammoth in src/lib/read-document.ts)
             │
             ▼ HTTPS / Server Function Call
Lovable Cloud Server Function (`evaluateJobFit`)
  ├── Step 1: Input Validation (src/lib/validation.ts)
  ├── Step 2: In-Depth Company Research (Tavily Search API)
  │     ├── Tech Stack & Infrastructure
  │     ├── Business Model & Revenue Drivers
  │     ├── User Flows & Customer Journeys
  │     └── Actors, Personas & Core Actions
  ├── Step 3: Candidate Evaluation (Lovable AI Gateway / Gemini 2.5 Flash)
  │     ├── Requirement & Resume Extraction
  │     ├── Multi-dimensional Fit Scoring
  │     ├── Evidence Mapping & Skill Gaps
  │     └── CV Rewrite Suggestions & Interview Talking Points
  └── Step 4: Storage & State Management (PostgreSQL via Lovable Cloud)
             │
             ▼ Real-time polling / subscription
Browser Results Dashboard
  ├── Tab 1: Fit Overview (Gauges, Scores, Top Gaps)
  ├── Tab 2: Company Intelligence (Tech Stack, Model, Flows, Actors)
  └── Tab 3: CV Action Plan (Tailored Bullets & Talking Points)
