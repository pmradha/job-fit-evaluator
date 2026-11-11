# AI Prompt Engineering & Schemas
## Project: Job Fit Evaluator
## Model: Google Gemini 2.5 Flash (via Lovable AI Gateway)

This document specifies the system prompts, input schemas, and expected structured JSON outputs for all AI stages in the Job Fit Evaluator pipeline.

---

## Architecture Overview

```text
Intake Data (Resume, JD, Company)
   │
   ├── Prompt 1: Tavily Research Synthesizer
   │     └── Produces structured Company Intelligence (tech stack, business model, actors, user flows)
   │
   ├── Prompt 2: Core Fit Evaluation & Gap Analysis
   │     └── Compares Resume vs. JD + Company Context -> Scores & Skill Gap Matrix
   │
   └── Prompt 3: Actionable CV Rewriter & Interview Talking Points
         └── Generates XYZ-format bullet rewrites aligned with target product workflows
