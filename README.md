# Job Fit Evaluator 🎯

An AI-powered evaluation platform that analyzes candidate resumes against specific Job Descriptions (JDs), deep-dives into employer context via real-time web intelligence, and produces actionable fit scores and tailored CV rewrites.
**Demo Video:**
https://www.dropbox.com/scl/fi/l2x44qp1ixcurbz240etq/Final-Demo-Job-Fit-Evaluator.mp4?rlkey=y8odmslovwvrqkprukgj3bouw&st=seg2a4ny&dl=0

---

## 🌟 Key Features

- **Privacy-First In-Browser Parsing:** Resumes (PDF, DOCX, TXT) are read directly inside the user's browser. Raw files are never uploaded to servers or stored.
- **Deep Company Intelligence (Tavily Search API):** Automatically extracts the target company's engineering tech stack, business model, primary user flows, and core customer actors.
- **Multi-Dimensional Fit Scoring:** Evaluates technical competencies, relevant scope of experience, and company domain alignment using AI.
- **Actionable CV Enhancements:** Generates ready-to-use resume bullet point rewrites aligned with the employer's specific product workflows and metrics.
- **Ephemeral Data Lifecycle:** All evaluation data expires automatically after 24 hours or can be purged instantly by the user.

---

## 📚 Documentation

- [Product Requirements Document (PRD)](./docs/PRD-Job_Fit_Evaluator_v2.md)
- [Technical Implementation Guide](./docs/Implementation-Job_Fit_Evaluator_v2.md)
- [AI Prompts & JSON Schemas](./docs/prompts.md)

---

## 🚀 Getting Started

### Prerequisites
- [Bun](https://bun.sh/) (or Node.js 18+)
- Tavily API Key ([Get one here](https://tavily.com))

### Setup & Run
1. Clone the repository and install dependencies:
   ```bash
   bun install
