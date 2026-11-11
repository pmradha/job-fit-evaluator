# Technical Implementation Document
## Project: Job Fit Evaluator

### 1. Technology Architecture
- Frontend: React 19 / TanStack Start v1 / Tailwind CSS v4 / shadcn/ui
- Document Extraction: Client-side parsing via `pdf.js` and `mammoth` (PRD Privacy Rule: raw files never uploaded)
- Backend: Lovable Cloud Server Functions (`createServerFn`)
- Web Research: Tavily Search API directly via server function
- AI Engine: Google Gemini 2.5 Flash via Lovable AI Gateway
- Database: Lovable Cloud PostgreSQL with 24-hour TTL expiration

---

### 2. Data Models & Database Schema

#### `evaluations` Table
- `id` (UUID, Primary Key, `gen_random_uuid()`)
- `created_at` (TIMESTAMPTZ, default `now()`)
- `expires_at` (TIMESTAMPTZ, default `now() + interval '24 hours'`)
- `company_name` (TEXT, NOT NULL)
- `company_website` (TEXT)
- `status` (TEXT: `pending` | `researching` | `evaluating` | `completed` | `failed`)
- `tavily_research` (JSONB)
- `fit_scores` (JSONB)
- `skill_gaps` (JSONB)
- `bullet_rewrites` (JSONB)
- `interview_points` (JSONB)
- `error_message` (TEXT)

---

### 3. Server Pipeline Flow (`src/server/evaluate.ts`)

1. **Validation:** Validate incoming resume, job description, and company name using shared schemas in `src/lib/validation.ts`.
2. **Tavily Research Service:** Query Tavily API for company tech stack, user flows, and business model:
   - Query 1: `"{company_name} tech stack engineering architecture"`
   - Query 2: `"{company_name} business model customers pricing"`
   - Query 3: `"{company_name} product features user workflows"`
3. **AI Synthesis & Scoring:** Call Lovable AI Gateway with Gemini 2.5 Flash using structured output formats from `docs/prompts.md`.
4. **Persistence:** Save structured evaluation results into the PostgreSQL `evaluations` table.
5. **Client Response:** Return complete evaluation payload to browser dashboard.

---

### 4. Implementation Phases

- **Phase 1: Database & Lovable Cloud Backend:** Provision `evaluations` table and configure RLS.
- **Phase 2: Tavily Search Integration:** Implement server-side company research module using `TAVILY_API_KEY`.
- **Phase 3: AI Gateway & Scoring Engine:** Wire Gemini 2.5 Flash server function with structured JSON schemas.
- **Phase 4: Results Dashboard:** Build tabbed dashboard (Fit Overview, Company Deep Dive, CV Action Plan).
- **Phase 5: Privacy Purge & Polish:** Wire "Clear Everything" instant purge and 24h auto-expiration cleanup.
