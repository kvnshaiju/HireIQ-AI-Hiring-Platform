# HireIQ — AI-Powered Hiring Platform

> A production-ready, full-stack AI hiring platform that replaces traditional ATS systems like Greenhouse and Lever. Built with React, Node.js, Python, and GPT-4o.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [System Architecture](#2-system-architecture)
3. [Tech Stack](#3-tech-stack)
4. [Features](#4-features)
5. [How to Run (Current Setup)](#5-how-to-run-current-setup)
6. [Backend — Part 1: API Server & Authentication](#6-backend--part-1-api-server--authentication)
7. [Backend — Part 2: AI Services Layer](#7-backend--part-2-ai-services-layer)
8. [Backend — Part 3: Database, Queues & Email](#8-backend--part-3-database-queues--email)
9. [Frontend Architecture](#9-frontend-architecture)
10. [Role-Based Access Control](#10-role-based-access-control)
11. [API Reference](#11-api-reference)
12. [Test Credentials](#12-test-credentials)
13. [Environment Variables](#13-environment-variables)

---

## 1. Project Overview

HireIQ is an AI-powered hiring platform designed to automate and improve the recruitment process for companies of all sizes. It goes beyond traditional Applicant Tracking Systems by embedding artificial intelligence at every stage of the hiring pipeline.

### The Problem It Solves

Traditional hiring is slow, biased, and manual:
- Recruiters spend hours reading resumes
- Shortlisting is subjective and inconsistent
- Scheduling interviews is a back-and-forth process
- Bias creeps in through job descriptions and selection decisions
- Candidates get no feedback after rejection

### How HireIQ Fixes This

| Traditional ATS | HireIQ |
|---|---|
| Manual resume screening | AI parses and scores every resume automatically |
| Subjective shortlisting | Hybrid semantic + skill matching with explainable scores |
| Phone/video interviews scheduled manually | AI conducts text-based screening interviews |
| No bias detection | EEOC 4/5ths rule monitoring + JD bias scanner |
| Generic rejection emails | AI-generated personalized feedback per candidate |
| No analytics | Predictive insights, funnel analytics, bias reports |

---

## 2. System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        BROWSER                                   │
│              React 18 + Vite + TailwindCSS                      │
│         Recruiter Dashboard  |  Candidate Portal                │
└──────────────────────┬──────────────────────────────────────────┘
                       │ HTTP / WebSocket
┌──────────────────────▼──────────────────────────────────────────┐
│                   NODE.JS API SERVER                             │
│              Fastify v4 + TypeScript                            │
│   Auth │ Jobs │ Applications │ Interviews │ Analytics           │
│              JWT + bcrypt + Zod validation                      │
└──────┬───────────────┬────────────────────┬─────────────────────┘
       │               │                    │
┌──────▼──────┐ ┌──────▼──────┐ ┌──────────▼──────────┐
│  PostgreSQL  │ │    Redis    │ │   Python AI Service  │
│  (Neon.tech) │ │  (Upstash)  │ │   FastAPI + GPT-4o  │
│  Prisma ORM  │ │  BullMQ     │ │   Pinecone Vectors  │
└─────────────┘ └─────────────┘ └─────────────────────┘
                                          │
                               ┌──────────▼──────────┐
                               │     OpenAI API       │
                               │  GPT-4o + Embeddings │
                               └─────────────────────┘
```

### Data Flow — Resume Submission

```
Candidate uploads resume
        ↓
API receives file → validates (PDF/DOCX, <10MB)
        ↓
File saved to local disk (uploads/resumes/)
        ↓
BullMQ job queued (async, non-blocking)
        ↓
Worker picks up job:
  1. Read file from disk
  2. Call Python AI: parse resume (GPT-4o extraction)
  3. Generate embedding (text-embedding-3-large, 1536 dims)
  4. Upsert to Pinecone vector DB
  5. Compute hybrid match score vs job
  6. Save AI evaluation to PostgreSQL
  7. Update application status → SCREENED
        ↓
Recruiter sees ranked candidates in Kanban pipeline
```

---

## 3. Tech Stack

### Frontend
| Technology | Version | Purpose |
|---|---|---|
| React | 18.3 | UI framework |
| Vite | 5.3 | Build tool & dev server |
| TypeScript | 5.5 | Type safety |
| TailwindCSS | 3.4 | Utility-first styling |
| Zustand | 4.5 | Global state management |
| React Query | 5.50 | Server state & caching |
| React Router | 6.25 | Client-side routing |
| Recharts | 2.12 | Analytics charts |
| Radix UI | latest | Accessible UI primitives |
| Socket.io-client | 4.7 | WebSocket for AI interviews |
| Axios | 1.7 | HTTP client |

### Backend (Node.js API)
| Technology | Version | Purpose |
|---|---|---|
| Node.js | 20 | Runtime |
| Fastify | 4.28 | HTTP framework |
| TypeScript | 5.0 | Type safety |
| Prisma | 5.0 | ORM + migrations |
| PostgreSQL | 15 | Primary database |
| BullMQ | 5.0 | Job queue (async processing) |
| ioredis | 5.4 | Redis client |
| JWT + bcrypt | latest | Authentication |
| Nodemailer | latest | Email sending |
| Zod | 3.23 | Runtime validation |
| Winston | 3.13 | Structured logging |

### AI Service (Python)
| Technology | Version | Purpose |
|---|---|---|
| Python | 3.11 | Runtime |
| FastAPI | 0.111 | HTTP framework |
| OpenAI SDK | 1.35 | GPT-4o + embeddings |
| pdfplumber | 0.11 | PDF text extraction |
| docx2txt | 0.8 | DOCX text extraction |
| Pinecone | 4.0 | Vector database |
| scikit-learn | 1.5 | Cosine similarity |
| numpy | 1.26 | Numerical operations |
| Pydantic | 2.7 | Data validation |

### Infrastructure
| Service | Provider | Purpose |
|---|---|---|
| PostgreSQL | Neon.tech (free) | Primary database |
| Redis | Upstash (free) | Queues + caching |
| Email | Gmail SMTP | Transactional emails |
| File Storage | Local disk | Resume files |
| Vector DB | Pinecone (free) | Semantic search |

---

## 4. Features

### For Recruiters
- **Job Management** — Create, edit, publish jobs with multi-step form
- **AI JD Optimizer** — Scans job descriptions for bias, rewrites them to be more inclusive
- **Candidate Pipeline** — Kanban board (Applied → Screened → Shortlisted → Interviewing → Offer)
- **AI Resume Screening** — Automatic parsing, skill extraction, and 5-dimension scoring
- **Semantic Matching** — Hybrid score: 55% semantic similarity + 45% skill coverage
- **AI Interview Scheduling** — One click schedules interview + sends email invite to candidate
- **Interview Evaluation** — AI evaluates completed interviews across 5 dimensions
- **Bias Monitoring** — EEOC 4/5ths rule applied to selection pass rates by source
- **Analytics Dashboard** — Hiring funnel, applications over time, source breakdown, bias reports
- **Candidate Feedback** — AI generates personalized rejection messages (bias-safe)
- **Recruiter Copilot** — AI reasoning shown next to every score

### For Candidates
- **Browse Jobs** — See all open positions across all companies
- **Apply with Resume** — Upload PDF/DOCX, AI processes it automatically
- **My Applications** — Track status, see AI match scores and evaluation breakdown
- **AI Interview** — Conversational AI interview via chat interface
- **My Profile** — Manage skills, upload resume, see extracted data
- **Email Notifications** — Verification, interview invites, offer notifications

### AI Capabilities
- Resume parsing with GPT-4o (structured JSON extraction)
- 1536-dimensional embeddings via text-embedding-3-large
- Semantic candidate ranking via Pinecone vector search
- Conversational AI interviewer (streaming via WebSocket)
- Post-interview evaluation with dimension scores + evidence quotes
- Job description bias detection (regex + LLM)
- Candidate success prediction (performance + retention)
- Rejection feedback generation (protected-characteristic safe)

---

## 5. How to Run (Current Setup)

### Prerequisites Already Installed
- Node.js 24 (your system)
- Python 3.11 (installed alongside 3.14)
- npm 11

### Services Already Configured
- **Database**: Neon PostgreSQL (cloud, already seeded)
- **Redis**: Upstash (cloud, already connected)
- **Email**: Gmail SMTP (`kartheesanjs26@gmail.com`)
- **OpenAI**: API key configured (check quota at platform.openai.com)

### Step 1 — Start the API Server

Open **Terminal 1**:
```powershell
cd hireiq/apps/api
npm run dev
```
Expected output:
```
🚀 HireIQ API running on port 3001
✅ Email: real SMTP connected  host=smtp.gmail.com
Redis connected
```

### Step 2 — Start the AI Service

Open **Terminal 2**:
```powershell
cd hireiq/apps/ai
venv\Scripts\activate
python -m uvicorn main:app --reload --port 8001
```
Expected output:
```
✅ OpenAI connection verified
✅ AI Service ready
Uvicorn running on http://127.0.0.1:8001
```

### Step 3 — Start the Frontend

Open **Terminal 3**:
```powershell
cd hireiq/apps/web
npm run dev
```
Expected output:
```
VITE v5.x  ready in 283ms
➜  Local:   http://localhost:5173/
```

### Step 4 — Open the App

Go to **http://localhost:5173**

### Verify All Services Are Running

```powershell
# API health
Invoke-RestMethod "http://localhost:3001/health"

# AI service health
Invoke-RestMethod "http://localhost:8001/health"

# Frontend
# Open http://localhost:5173 in browser
```

### If Something Fails

**API won't start:**
```powershell
# Check .env has correct values
cat hireiq/apps/api/.env
# Regenerate Prisma client
cd hireiq/apps/api && npx prisma generate
```

**AI service won't start:**
```powershell
# Make sure you're in the venv
cd hireiq/apps/ai
venv\Scripts\activate
pip install -r requirements.txt
```

**Database issues:**
```powershell
cd hireiq/apps/api
npx prisma db push
npm run db:seed
```

---

## 6. Backend — Part 1: API Server & Authentication

> **Teammate 1 covers this section in the presentation.**

### Overview

The API server is the central hub of HireIQ. It handles all HTTP requests from the frontend, enforces authentication and authorization, and coordinates between the database, AI service, and job queue.

**Technology:** Node.js 20 + Fastify v4 + TypeScript

**Entry point:** `apps/api/src/server.ts`

### Why Fastify?

Fastify was chosen over Express for three reasons:
1. **Performance** — Fastify is 2-3x faster than Express due to its schema-based serialization
2. **TypeScript-first** — Built-in TypeScript support with full type inference
3. **Plugin ecosystem** — Official plugins for JWT, CORS, WebSocket, multipart uploads

### Server Bootstrap (`server.ts`)

```typescript
// Plugins registered in order:
await fastify.register(fastifyCors, { origin: 'http://localhost:5173' });
await fastify.register(fastifyMultipart, { limits: { fileSize: 10MB } });
await fastify.register(fastifyWebsocket);
await fastify.register(fastifyStatic, { root: 'uploads/', prefix: '/uploads/' });
await fastify.register(authPlugin);  // JWT + decorators

// Routes
await fastify.register(authRoutes,         { prefix: '/api/v1/auth' });
await fastify.register(jobsRoutes,         { prefix: '/api/v1/jobs' });
await fastify.register(applicationsRoutes, { prefix: '/api/v1/applications' });
await fastify.register(interviewsRoutes,   { prefix: '/api/v1/interviews' });
await fastify.register(analyticsRoutes,    { prefix: '/api/v1/analytics' });
await fastify.register(candidatesRoutes,   { prefix: '/api/v1/candidates' });
```

### Authentication System

**File:** `apps/api/src/routes/auth.ts`

HireIQ uses **JWT (JSON Web Tokens)** for stateless authentication.

#### Registration Flow
```
POST /api/v1/auth/register
  → Validate input (Zod schema)
  → Check email uniqueness
  → Hash password (bcrypt, 12 rounds)
  → Generate email verification token (crypto.randomBytes(32))
  → Create user + org (if recruiter)
  → Send verification email
  → Return JWT token + user object
```

#### JWT Payload Structure
```json
{
  "id": "uuid",
  "email": "user@example.com",
  "role": "RECRUITER | CANDIDATE | HIRING_MANAGER | ADMIN",
  "orgId": "uuid or null",
  "iat": 1234567890,
  "exp": 1234567890
}
```

#### Auth Decorators (`plugins/auth.ts`)

Two Fastify decorators are added to every route handler:

```typescript
// Verifies JWT, attaches request.jwtUser
fastify.authenticate

// Verifies JWT + checks role
fastify.authorize(['RECRUITER', 'HIRING_MANAGER'])
```

Usage in routes:
```typescript
fastify.post('/jobs', {
  preHandler: [fastify.authorize(['RECRUITER', 'HIRING_MANAGER', 'ADMIN'])]
}, handler)
```

#### Email Verification Flow
```
Register → email sent with token (24h expiry)
  ↓
POST /auth/verify-email { token }
  → Find user by emailVerifyToken
  → Check expiry
  → Set isEmailVerified = true
  → Send welcome email
  ↓
POST /auth/forgot-password { email }
  → Generate reset token (1h expiry)
  → Send reset email
  ↓
POST /auth/reset-password { token, password }
  → Validate token + expiry
  → Hash new password
  → Clear reset token
```

### Jobs API (`routes/jobs.ts`)

The jobs module handles the full lifecycle of job postings.

#### Key Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | `/jobs` | Recruiter | Create job with skills |
| GET | `/jobs` | Recruiter | List org's jobs (filtered) |
| GET | `/jobs/public` | Any | All OPEN jobs (for candidates) |
| GET | `/jobs/:id` | Any | Job detail with skills |
| PATCH | `/jobs/:id` | Recruiter | Update job |
| DELETE | `/jobs/:id` | Recruiter | Soft-close job |
| POST | `/jobs/analyze-jd` | Recruiter | AI bias + effectiveness analysis |
| GET | `/jobs/:id/candidates` | Recruiter | Ranked candidates for job |
| POST | `/jobs/:id/shortlist` | Recruiter | Bulk shortlist candidates |

#### Important Design Decision: Public vs Org-Scoped

```typescript
// Recruiters see only their org's jobs
GET /jobs → WHERE orgId = request.jwtUser.orgId

// Candidates see all open jobs across all orgs
GET /jobs/public → WHERE status = 'OPEN'
```

This separation ensures recruiters can't see competitor job data while candidates can browse all opportunities.

### Applications API (`routes/applications.ts`)

Handles the full application lifecycle from submission to decision.

#### Application Submission (Async Pattern)
```typescript
POST /applications (multipart/form-data)
  1. Validate file (PDF/DOCX, <10MB)
  2. Save to disk → get file key
  3. Create Application record (status: APPLIED)
  4. Enqueue BullMQ job → return 202 Accepted immediately
  // AI processing happens asynchronously in background
```

This is critical — **we never block an HTTP request with an LLM call**. The API returns in ~200ms while AI processing happens in the background over 5-15 seconds.

### Candidates API (`routes/candidates.ts`)

Dual-purpose route — serves both recruiters and candidates:

```typescript
// Recruiter: see all candidates who applied to their org's jobs
GET /candidates → aggregates across all org jobs

// Candidate: manage own profile
GET  /candidates/me        → own profile
POST /candidates/me/resume → upload resume
POST /candidates/me/skills → add skill
```

### Analytics API (`routes/analytics.ts`)

Aggregates hiring data for the recruiter dashboard:

```typescript
GET /analytics/overview → {
  totalJobs, totalApplications, avgMatchScore,
  offerAcceptanceRate, hiringFunnelData, topSources, recentActivity
}

GET /analytics/bias → per-job EEOC fairness reports

GET /analytics/predictions/:jobId → AI success predictions per candidate

GET /analytics/applications-over-time → time series data (last 90 days)
```

### Input Validation with Zod

Every request body is validated with Zod before touching the database:

```typescript
const createJobSchema = z.object({
  title: z.string().min(2),
  description: z.string().min(10),
  requirements: z.record(z.unknown()).default({}),
  remoteType: z.enum(['remote', 'hybrid', 'onsite']).optional(),
  salaryMin: z.number().int().positive().optional(),
  // ...
});

const body = createJobSchema.safeParse(request.body);
if (!body.success) {
  return reply.status(400).send({
    error: 'Validation failed',
    code: 'VALIDATION_ERROR',
    details: body.error.flatten()
  });
}
```

### Error Handling Convention

All API errors follow a consistent structure:
```json
{
  "error": "Human-readable message",
  "code": "MACHINE_READABLE_CODE",
  "details": {}
}
```

HTTP status codes used correctly:
- `200` — Success
- `201` — Created
- `202` — Accepted (async job queued)
- `400` — Validation error
- `401` — Not authenticated
- `403` — Authenticated but not authorized
- `404` — Resource not found
- `409` — Conflict (duplicate email, duplicate application)
- `500` — Server error

---
## 7. Backend � Part 2: AI Services Layer

> **Teammate 2 covers this section in the presentation.**

### Overview

The AI Services Layer is a separate Python microservice that handles all machine learning and LLM operations. It is completely decoupled from the Node.js API � they communicate over HTTP.

**Technology:** Python 3.11 + FastAPI + OpenAI SDK + Pinecone

**Entry point:** `apps/ai/main.py`

**Base URL:** `http://localhost:8001`

**Interactive Docs:** `http://localhost:8001/docs` (Swagger UI auto-generated)

### Why a Separate Python Service?

Python has the best ecosystem for AI/ML:
- OpenAI's official SDK is Python-first
- pdfplumber and docx2txt for document parsing
- scikit-learn for cosine similarity
- Pinecone's Python client is more mature

Separating it also means the AI service can be scaled independently � if resume processing becomes a bottleneck, you can run multiple AI service instances without touching the API.

### Service Architecture

```
apps/ai/
+-- main.py                    # FastAPI app, startup events
+-- config.py                  # Environment config
+-- routers/
�   +-- resume.py              # Resume parsing + evaluation
�   +-- matching.py            # Semantic matching + ranking
�   +-- interview.py           # AI interview bot
�   +-- jd_optimizer.py        # Job description analysis
�   +-- analytics.py           # Bias monitoring + predictions
+-- services/
    +-- resume_pipeline.py     # Core resume processing logic
    +-- matching_engine.py     # Hybrid scoring algorithm
    +-- interview_bot.py       # GPT-4o interview conductor
    +-- jd_analyzer.py         # Bias detection + JD rewriting
    +-- bias_monitor.py        # EEOC fairness analysis
    +-- feedback_generator.py  # Rejection feedback writer
    +-- prediction_service.py  # Success prediction
```

### Resume Processing Pipeline (`services/resume_pipeline.py`)

This is the most complex service. It processes a resume file through multiple stages concurrently.

#### Stage 1: Text Extraction

```python
def extract_text(file_bytes: bytes, mime_type: str) -> str:
    if mime_type == "application/pdf":
        # pdfplumber handles complex PDF layouts, tables, columns
        with pdfplumber.open(io.BytesIO(file_bytes)) as pdf:
            return "\n".join(page.extract_text() for page in pdf.pages)
    else:
        # docx2txt handles Word documents
        return docx2txt.process(io.BytesIO(file_bytes))
```

#### Stage 2: Concurrent LLM Extraction + Embedding

```python
async def process_resume(file_bytes, mime_type):
    raw_text = extract_text(file_bytes, mime_type)

    # Run BOTH concurrently � saves ~2 seconds
    extraction_task = asyncio.create_task(extract_resume_data(raw_text))
    embedding_task  = asyncio.create_task(get_embedding(raw_text))

    extracted, embedding = await asyncio.gather(extraction_task, embedding_task)
    return { **extracted, "embedding": embedding }
```

#### Stage 3: GPT-4o Structured Extraction

The LLM is given a strict JSON schema to fill in:

```python
EXTRACTION_PROMPT = """Extract resume data and return EXACTLY this JSON:
{
  "contact": { "name", "email", "phone", "location", "linkedin" },
  "work_experience": [{ "company", "title", "start_date", "end_date",
                        "description", "achievements" }],
  "education": [{ "institution", "degree", "field", "graduation_year" }],
  "skills": ["string"],
  "certifications": ["string"],
  "total_years_experience": float
}"""

response = await client.chat.completions.create(
    model="gpt-3.5-turbo",  # or gpt-4o when quota available
    messages=[...],
    response_format={"type": "json_object"},  # Forces valid JSON output
    temperature=0.1  # Low temperature = consistent extraction
)
```

#### Stage 4: Embedding Generation

```python
async def get_embedding(text: str) -> list[float]:
    # Cache in Redis for 24 hours (SHA256 key)
    cache_key = f"embedding:{hashlib.sha256(text.encode()).hexdigest()}"
    cached = await redis_client.get(cache_key)
    if cached:
        return json.loads(cached)

    response = await client.embeddings.create(
        model="text-embedding-3-large",  # 1536 dimensions
        input=text[:8000]  # Truncate to avoid token limits
    )
    embedding = response.data[0].embedding
    await redis_client.setex(cache_key, 86400, json.dumps(embedding))
    return embedding
```

### Matching Engine (`services/matching_engine.py`)

The matching engine computes how well a candidate fits a job using a **hybrid scoring approach**.

#### Hybrid Score Formula

```
Overall Score = (0.55 � Semantic Score) + (0.45 � Skill Coverage)
```

**Why hybrid?**
- Pure keyword matching misses candidates who describe skills differently ("ML" vs "machine learning")
- Pure semantic matching can match unrelated but similar-sounding content
- The combination gives the best of both worlds

#### Semantic Score (55% weight)

```python
def compute_cosine_similarity(vec1, vec2):
    a = np.array(vec1).reshape(1, -1)
    b = np.array(vec2).reshape(1, -1)
    return float(cosine_similarity(a, b)[0][0])
```

Cosine similarity measures the angle between two embedding vectors. A score of 1.0 means identical meaning, 0.0 means completely unrelated.

#### Skill Coverage (45% weight)

```python
def compute_skill_coverage(candidate_skills, required_skills):
    total_weight = sum(s["weight"] for s in required_skills)
    matched_weight = 0

    for skill in required_skills:
        # Fuzzy matching � "Node" matches "Node.js"
        is_matched = any(
            skill["name"].lower() in cs.lower() or
            cs.lower() in skill["name"].lower()
            for cs in candidate_skills
        )
        if is_matched:
            matched_weight += skill["weight"]

    return matched_weight / total_weight
```

#### Pinecone Vector Search

For ranking candidates at scale, we use Pinecone:

```python
async def rank_candidates_for_job(job_embedding, top_k=100):
    index = get_pinecone_index()
    results = index.query(
        vector=job_embedding,
        top_k=top_k,
        namespace="candidates",
        include_metadata=True
    )
    return results["matches"]
```

This returns the top 100 most semantically similar candidates in milliseconds, even with millions of candidates in the index.

### AI Interview Bot (`services/interview_bot.py`)

The interview bot conducts structured conversational interviews using GPT-4o.

#### Question Bank Generation

Before the interview starts, 15 questions are generated covering 5 dimensions:

```python
dimensions = [
    "technical_knowledge",   # 3 questions
    "problem_solving",       # 3 questions
    "communication",         # 2 questions
    "cultural_fit",          # 3 questions
    "leadership_potential",  # 2 questions
    "motivation"             # 2 questions
]
```

#### Streaming Interview Turns

The interviewer's responses stream word-by-word via Server-Sent Events:

```python
async def interview_turn_stream(transcript, context):
    stream = await client.chat.completions.create(
        model="gpt-4o",
        messages=build_messages(transcript, context),
        stream=True,          # Enable streaming
        temperature=0.8,      # Slightly creative for natural conversation
        max_tokens=300        # Keep responses concise
    )
    async for chunk in stream:
        if chunk.choices[0].delta.content:
            yield chunk.choices[0].delta.content  # Yield each word/token
```

The Node.js API receives this stream and forwards it to the browser via WebSocket, creating the real-time typing effect.

#### Interview Evaluation

After the interview ends, GPT-4o evaluates the full transcript:

```python
EVALUATION_PROMPT = """Evaluate across 5 dimensions (0-100):
- technical_knowledge: Depth and accuracy
- problem_solving: Structured thinking
- communication: Clarity and listening
- cultural_fit: Values alignment
- leadership_potential: Initiative and ownership

For each dimension, cite a SPECIFIC QUOTE from the transcript as evidence."""
```

This ensures **explainability** � every score has a reason backed by what the candidate actually said.

### Job Description Analyzer (`services/jd_analyzer.py`)

Analyzes job descriptions for bias and effectiveness using a two-pass approach.

#### Pass 1: Rule-Based Regex Scan (Fast)

```python
BIAS_PATTERNS = {
    "gendered_language": [
        r"\b(rockstar|ninja|guru|wizard)\b",      # Masculine-coded
        r"\b(nurturing|supportive|collaborative)\b" # Feminine-coded
    ],
    "exclusionary": [
        r"\bnative (english|speaker)\b",           # Nationality bias
        r"\b(young|energetic|digital native)\b"    # Age bias
    ],
    "unnecessary_requirements": [
        r"\bdegree required\b",                    # Credential inflation
        r"\bivy league\b"                          # Elitism
    ]
}
```

#### Pass 2: GPT-4o Deep Analysis (Thorough)

The LLM catches nuanced issues the regex misses and rewrites the entire JD:

```python
response = await client.chat.completions.create(
    model="gpt-4o",
    messages=[{
        "role": "system",
        "content": "Analyze for bias, score effectiveness 0-100, rewrite inclusively"
    }],
    response_format={"type": "json_object"}
)
# Returns: bias_score, effectiveness_score, issues[], optimized_jd
```

### Bias Monitor (`services/bias_monitor.py`)

Implements the **EEOC 4/5ths Rule** (also called the 80% rule) to detect discriminatory selection patterns.

#### The 4/5ths Rule Explained

If the pass rate of any group is less than 80% of the highest group's pass rate, it's flagged as potential adverse impact.

```python
def apply_four_fifths_rule(pass_rates):
    highest_rate = max(pass_rates.values())
    threshold = highest_rate * 0.8  # 80% of highest

    flags = []
    for group, rate in pass_rates.items():
        if rate < threshold:
            flags.append({
                "group": group,
                "pass_rate": rate,
                "disparity_ratio": rate / highest_rate,
                "severity": "high" if rate / highest_rate < 0.6 else "medium"
            })
    return flags
```

**Example:** If LinkedIn candidates pass at 60% and direct applicants pass at 30%, the disparity ratio is 0.5 � below the 0.8 threshold � flagged as high severity.

### Feedback Generator (`services/feedback_generator.py`)

Generates personalized rejection feedback that is empathetic, actionable, and legally safe.

#### Safety Checks

```python
PROTECTED_CHARACTERISTICS = [
    r"\b(age|young|old|senior)\b",
    r"\b(gender|male|female|man|woman)\b",
    r"\b(race|ethnic|nationality)\b",
    r"\b(religion|faith|belief)\b",
    r"\b(disability|disabled)\b",
    # ... more patterns
]

def check_for_protected_characteristics(text):
    violations = []
    for pattern in PROTECTED_CHARACTERISTICS:
        if re.findall(pattern, text, re.IGNORECASE):
            violations.append(pattern)
    return violations
```

If violations are found, the feedback is regenerated. If it fails twice, a safe template is used. This ensures HireIQ never exposes the company to discrimination lawsuits.

### Redis Caching Strategy

All expensive AI calls are cached in Redis:

| Cache Key | TTL | What's Cached |
|---|---|---|
| `embedding:{sha256}` | 24 hours | Text embeddings |
| `resume_extract:{sha256}` | 1 hour | GPT-4o extraction results |
| `eval:{sha256}` | 1 hour | Resume evaluations |
| `jd_analysis:{sha256}` | 1 hour | JD analysis results |
| `predict:{sha256}` | 1 hour | Success predictions |

This means if the same resume is submitted twice, the second processing is nearly instant.

---
## 8. Backend � Part 3: Database, Queues & Email

> **Teammate 3 covers this section in the presentation.**

### Overview

This section covers the data persistence layer, asynchronous job processing, and the email notification system � the infrastructure that keeps HireIQ reliable and scalable.

### Database Design (PostgreSQL + Prisma)

HireIQ uses PostgreSQL 15 with the pgvector extension, managed through Prisma ORM.

**Database provider:** Neon.tech (serverless PostgreSQL, free tier)

#### Entity Relationship Overview

`
Organization (1) ---- (many) User
Organization (1) ---- (many) Job
User (1) ------------ (1) Candidate
Job (1) -------------- (many) Application
Candidate (1) -------- (many) Application
Application (1) ------- (many) Interview
Application (1) ------- (many) AIEvaluation
Interview (1) -------- (many) AIEvaluation
Skill (many) ----------- (many) Candidate  [via CandidateSkill]
Skill (many) ----------- (many) Job         [via JobSkill]
`

#### Schema Highlights

**Multi-tenancy via Organization:**
`prisma
model Organization {
  id    String @id @default(uuid())
  name  String
  plan  String @default("starter")
  users User[]
  jobs  Job[]
}
`
Every recruiter belongs to an org. Jobs are org-scoped. Candidates are global (no org).

**Application � the central entity:**
`prisma
model Application {
  id                String            @id @default(uuid())
  jobId             String
  candidateId       String
  status            ApplicationStatus @default(APPLIED)
  matchScore        Float?            // 0.0 to 1.0
  rank              Int?              // Position within job
  shortlistReason   String?           // AI explanation
  candidateFeedback String?           // Rejection message
  @@unique([jobId, candidateId])      // No duplicate applications
  @@index([jobId, matchScore])        // Fast ranked queries
  @@index([candidateId])              // Fast candidate lookup
}
`

**AI Evaluation � explainability built in:**
`prisma
model AIEvaluation {
  id            String  @id @default(uuid())
  applicationId String
  interviewId   String?
  evalType      String  // "resume_screening" | "interview"
  scores        Json    // { skill_match: 0.85, experience: 0.72, ... }
  reasoning     Json    // { skill_match: "Candidate has Python...", ... }
  confidence    Float?  // Model confidence 0.0-1.0
  flags         Json    // Red flags found
  rawResponse   String? // Full LLM response for debugging
}
`

Every score is stored with its reasoning. The frontend always shows "why" next to every number.

**Skill Graph:**
`prisma
model Skill {
  id       String @id @default(uuid())
  name     String @unique
  category String?  // "programming" | "devops" | "design" | ...
  aliases  Json     // ["JS", "JavaScript", "ECMAScript"]
}

model CandidateSkill {
  candidateId     String
  skillId         String
  proficiency     String?  // "beginner" | "intermediate" | "expert"
  yearsExperience Float?
  @@id([candidateId, skillId])
}

model JobSkill {
  jobId    String
  skillId  String
  required Boolean @default(true)
  weight   Float   @default(1.0)  // Higher = more important
  @@id([jobId, skillId])
}
`

#### Application Status State Machine

`
APPLIED ? SCREENED ? SHORTLISTED ? INTERVIEWING ? OFFER ? HIRED
                                                        ?
APPLIED ? SCREENED ? REJECTED
APPLIED ? WITHDRAWN (candidate withdraws)
`

Transitions are enforced at the API level. You can't skip from APPLIED to HIRED.

#### Database Indexes

Performance-critical indexes:
`sql
-- Fast ranked candidate queries per job
CREATE INDEX ON "Application"("jobId", "matchScore" DESC);

-- Fast lookup of all applications by candidate
CREATE INDEX ON "Application"("candidateId");

-- Fast interview lookup by application
CREATE INDEX ON "Interview"("applicationId");
`

#### Prisma Transactions

Multi-table writes use transactions to ensure consistency:

`	ypescript
// Creating a job with skills � atomic operation
const job = await prisma.(async (tx) => {
  const created = await tx.job.create({ data: jobData });

  for (const skill of skills) {
    const skillRecord = await tx.skill.upsert({
      where: { name: skill.name },
      update: {},
      create: { name: skill.name }
    });
    await tx.jobSkill.create({
      data: { jobId: created.id, skillId: skillRecord.id, weight: skill.weight }
    });
  }

  return tx.job.findUnique({ where: { id: created.id }, include: { jobSkills: true } });
});
// If ANY step fails, ALL changes are rolled back
`

### Async Job Queue (BullMQ + Redis)

Resume processing is the most expensive operation � it involves file I/O, two LLM calls, and a vector DB upsert. Doing this synchronously would make the API unresponsive for 10-15 seconds.

**Solution:** BullMQ job queue backed by Redis (Upstash).

#### Queue Architecture

`
HTTP Request (POST /applications)
        ?
API validates + saves file + creates Application record
        ?
Enqueue job to BullMQ ? return 202 Accepted (instant)
        ?
Worker picks up job (background process)
        ?
9-step processing pipeline
        ?
Application updated with scores
`

#### The 9-Step Resume Worker (workers/resumeWorker.ts)

`	ypescript
// Step 1: Read file from disk
const fileBuffer = readResumeFile(resumeKey);
const fileBase64 = fileBuffer.toString('base64');

// Step 2: Parse resume via AI service
const parsedResume = await aiClient.processResume(fileBase64, mimeType);

// Step 3: Update candidate with parsed data
await prisma.candidate.update({ data: { parsedResume, skills, experienceYears } });

// Step 4: Upsert skills to skill graph
for (const skillName of parsedResume.skills) {
  await prisma.skill.upsert({ where: { name: skillName }, ... });
  await prisma.candidateSkill.upsert({ ... });
}

// Step 5: Upsert candidate vector to Pinecone
await aiClient.upsertCandidateVector(candidateId, embedding, metadata);

// Step 6: Evaluate resume vs job (5-dimension scoring)
const evaluation = await aiClient.evaluateResume(parsedResume, jobRecord);

// Step 7: Save AI evaluation to database
await prisma.aIEvaluation.create({ data: { scores, reasoning, flags } });

// Step 8: Compute hybrid match score
const matchResult = await aiClient.computeMatchScore(embedding, [], skills, requiredSkills);

// Step 9: Update application + re-rank all candidates for this job
await prisma.application.update({ data: { status: 'SCREENED', matchScore, rank } });
`

#### Retry Logic

`	ypescript
await resumeQueue.add('process-resume', jobData, {
  attempts: 3,                              // Retry up to 3 times
  backoff: { type: 'exponential', delay: 5000 }  // 5s, 10s, 20s
});
`

If all 3 attempts fail, the application is marked with an error flag and the recruiter is notified.

#### Queue Configuration

`	ypescript
export const resumeQueue = new Queue('resume-processing', {
  connection: redis,
  defaultJobOptions: {
    removeOnComplete: 100,  // Keep last 100 completed jobs for debugging
    removeOnFail: 50        // Keep last 50 failed jobs for inspection
  }
});
`

### Email System (Nodemailer + Gmail)

HireIQ sends transactional emails for every important event in the hiring process.

**Provider:** Gmail SMTP (configured with App Password)
**Fallback:** Ethereal (fake SMTP for development � emails viewable at ethereal.email)

#### Email Templates

All emails use a consistent HTML layout with the HireIQ brand:

| Email | Trigger | Recipient |
|---|---|---|
| Email Verification | User registers | New user |
| Welcome | Email verified | New user |
| Password Reset | Forgot password | User |
| Interview Invitation | Recruiter schedules interview | Candidate |
| Interview Complete | Candidate finishes interview | Recruiter |

#### Smart SMTP Detection

`	ypescript
async function getTransporter() {
  const isConfigured =
    host && user && pass &&
    !user.includes('your_gmail') &&  // Not a placeholder
    user.includes('@');              // Looks like a real email

  if (isConfigured) {
    // Try real SMTP
    const t = nodemailer.createTransport({ host, port, auth: { user, pass } });
    await t.verify();  // Test connection
    return t;
  }

  // Fall back to Ethereal for development
  return getFallbackTransporter();
}
`

#### Interview Invitation Email

The most important email � sent when a recruiter schedules an AI interview:

`	ypescript
const interviewUrl = ${APP_URL}/portal/interview/;

sendEmail({
  to: candidate.email,
  subject: Interview Invitation:  at ,
  html: interviewInviteEmail({
    candidateName, jobTitle, companyName,
    interviewUrl,   // Direct link to start the interview
    recruiterName
  })
});
`

The email contains a direct link. The candidate clicks it, logs in, and starts the AI interview immediately � no scheduling, no back-and-forth.

#### Bias-Safe Rejection Feedback

The feedback generator has a two-layer safety system:

`	ypescript
// Layer 1: Post-generation regex check
const violations = checkForProtectedCharacteristics(feedback);

if (violations.length > 0) {
  // Layer 2: Regenerate with stricter prompt
  feedback = await regenerateWithStricterPrompt();

  // Layer 3: If still fails, use safe template
  if (checkForProtectedCharacteristics(feedback).length > 0) {
    feedback = generateSafeTemplate(jobTitle, strengths, gaps);
  }
}
`

This three-layer approach ensures the company is never exposed to discrimination claims from automated feedback.

### File Storage

For this college project, resumes are stored on the local filesystem instead of AWS S3:

`	ypescript
// Upload: save to disk
const key = esumes/-.;
fs.writeFileSync(path.join(UPLOAD_DIR, filename), file);

// Download: served as static files
// GET /uploads/resumes/filename.pdf
`

The API serves the uploads folder as static files via @fastify/static. In production, this would be replaced with S3 presigned URLs.

### Security Measures

| Threat | Mitigation |
|---|---|
| SQL Injection | Prisma parameterized queries (never raw SQL) |
| XSS | React escapes all output by default |
| CSRF | JWT in Authorization header (not cookies) |
| Brute force | bcrypt with 12 rounds (slow by design) |
| File upload attacks | MIME type + size validation before saving |
| Token theft | JWT expires in 7 days, stored in localStorage |
| Bias in AI | Post-generation regex checks on all feedback |
| Data leakage | Org-scoped queries (recruiters can't see other orgs) |

---

## 9. Frontend Architecture

### Component Structure

`
src/
+-- pages/
�   +-- auth/          LoginPage, RegisterPage, VerifyEmailPage,
�   �                  ForgotPasswordPage, ResetPasswordPage
�   +-- dashboard/     DashboardPage (recruiter home)
�   +-- jobs/          JobsPage, JobDetailPage, CreateJobModal, EditJobModal
�   +-- candidates/    CandidatesPage (recruiter view)
�   +-- interviews/    InterviewsPage (recruiter view)
�   +-- analytics/     AnalyticsPage
�   +-- settings/      SettingsPage
�   +-- candidate/     CandidatePortalPage, CandidateJobsPage,
�                      CandidateApplicationsPage, CandidateProfilePage,
�                      CandidateInterviewPage
+-- components/
�   +-- ui/            Button, Input, Badge, Card, Modal, Avatar,
�   �                  ScoreBar, SkillTag, Spinner, EmailVerifyBanner
�   +-- layout/        Sidebar, PageLayout
�   +-- candidates/    CandidateDetailPanel
+-- stores/
�   +-- authStore.ts   Zustand store (auth state, persisted to localStorage)
+-- lib/
�   +-- api.ts         All typed API functions (authApi, jobsApi, etc.)
+-- types/
    +-- index.ts       Shared TypeScript interfaces
`

### State Management

**Zustand** for global auth state:
`	ypescript
// Reads localStorage SYNCHRONOUSLY at module load
// Prevents logout-on-reload flash
const storedAuth = readStoredAuth();

export const useAuthStore = create((set) => ({
  user: storedAuth.user,
  isAuthenticated: storedAuth.isAuthenticated,
  // ...
}));
`

**React Query** for server state:
`	ypescript
const { data, isLoading } = useQuery({
  queryKey: ['jobs', search, statusFilter],
  queryFn: () => jobsApi.list({ search, status: statusFilter }),
  staleTime: 30_000,  // Cache for 30 seconds
});
`

### Role-Based UI

The same app serves two completely different experiences:

`	ypescript
// Router guards
const RecruiterRoute = ({ children }) => {
  const { isAuthenticated, user } = useAuthStore();
  if (!isAuthenticated) return <Navigate to="/login" />;
  if (user.role === 'CANDIDATE') return <Navigate to="/portal" />;
  return children;
};

// Sidebar shows different nav based on role
const navItems = isRecruiter ? RECRUITER_NAV : CANDIDATE_NAV;
`

---

## 10. Role-Based Access Control

### Roles

| Role | Access |
|---|---|
| RECRUITER | Full access to org's jobs, candidates, interviews, analytics |
| HIRING_MANAGER | Same as RECRUITER |
| ADMIN | Same as RECRUITER + org settings |
| CANDIDATE | Own applications, public jobs, own profile, own interviews |

### What Each Role Sees

**Recruiter Dashboard (/dashboard):**
- Hiring funnel stats
- Recent activity feed
- Avg match score, offer acceptance rate

**Recruiter Jobs (/jobs):**
- All jobs in their organization
- Create/edit/close jobs
- AI JD optimization

**Recruiter Job Detail (/jobs/:id):**
- Kanban pipeline with all candidates
- Click candidate ? detail panel with AI scores
- Schedule interview, shortlist, reject with feedback

**Recruiter Candidates (/candidates):**
- All candidates who applied to org's jobs
- Click ? modal with full profile + AI evaluation

**Recruiter Interviews (/interviews):**
- All scheduled/completed interviews
- Click ? detail modal with AI evaluation scores

**Recruiter Analytics (/analytics):**
- Hiring funnel chart
- Applications over time
- Source breakdown pie chart
- Bias monitoring with EEOC flags

**Candidate Portal (/portal):**
- Welcome dashboard with latest jobs
- Quick stats

**Candidate Browse Jobs (/portal/jobs):**
- All open jobs across all companies
- Search by title/location
- Apply with resume upload

**Candidate My Applications (/portal/applications):**
- All their applications with status
- AI match score breakdown
- Rejection feedback (if rejected)
- Shortlist/offer notifications

**Candidate My Profile (/portal/profile):**
- Upload/update resume
- Add skills manually
- See AI-extracted data

---

## 11. API Reference

### Base URL
`
http://localhost:3001/api/v1
`

### Authentication
All endpoints (except register/login) require:
`
Authorization: Bearer <jwt_token>
`

### Endpoints Summary

#### Auth
`
POST /auth/register          Create account
POST /auth/login             Login
GET  /auth/me                Get current user
POST /auth/verify-email      Verify email with token
POST /auth/resend-verification  Resend verification email
POST /auth/forgot-password   Request password reset
POST /auth/reset-password    Reset password with token
`

#### Jobs
`
GET  /jobs                   List org's jobs (recruiter)
GET  /jobs/public            List all open jobs (candidate)
POST /jobs                   Create job
GET  /jobs/:id               Get job detail
PATCH /jobs/:id              Update job
DELETE /jobs/:id             Close job (soft delete)
POST /jobs/analyze-jd        AI bias + effectiveness analysis
GET  /jobs/:id/candidates    Ranked candidates for job
POST /jobs/:id/shortlist     Bulk shortlist candidates
GET  /jobs/:id/bias-report   Fairness report for job
`

#### Applications
`
POST /applications           Submit application (multipart)
GET  /applications/my        Candidate's own applications
GET  /applications/:id       Get application detail
PATCH /applications/:id/status  Update status
GET  /applications/:id/evaluation  AI evaluation breakdown
POST /applications/:id/feedback    Generate rejection feedback
`

#### Candidates
`
GET  /candidates             List pipeline candidates (recruiter)
GET  /candidates/me          Own profile (candidate)
POST /candidates/me/resume   Upload resume (candidate)
POST /candidates/me/skills   Add skill (candidate)
`

#### Interviews
`
GET  /interviews             List org's interviews (recruiter)
POST /interviews             Schedule interview + send email
GET  /interviews/:id         Get interview detail
GET  /interviews/my          Candidate's own interviews
WS   /interviews/:id/stream  WebSocket AI interview stream
`

#### Analytics
`
GET /analytics/overview              Hiring funnel + stats
GET /analytics/bias                  Fairness reports
GET /analytics/predictions/:jobId    Success predictions
GET /analytics/applications-over-time  Time series data
`

---

## 12. Test Credentials

After running 
pm run db:seed:

| Role | Email | Password | Notes |
|---|---|---|---|
| Recruiter | recruiter@acme.com | password123 | Acme Corp org, 5 seeded jobs |
| Recruiter | recruiter2@acme.com | password123 | Acme Corp org |
| Candidate | candidate1@example.com | password123 | Alex Rivera, 5 yrs exp |
| Candidate | candidate2@example.com | password123 | Jordan Kim, 4 yrs exp |
| Candidate | candidate3@example.com | password123 | Taylor Morgan, 6 yrs exp |
| Your Account | kartheesanjs26@gmail.com | Test@1234 | Test Org, Gmail verified |

---

## 13. Environment Variables

File location: pps/api/.env

`env
# -- Core ----------------------------------------------------------
JWT_SECRET=hireiq_super_secret_jwt_key_college_project_2024
PORT=3001

# -- Database (Neon PostgreSQL) ------------------------------------
DATABASE_URL=postgresql://user:pass@host/db?sslmode=require

# -- Redis (Upstash) -----------------------------------------------
REDIS_URL=rediss://default:password@host:6379

# -- AI Service ----------------------------------------------------
AI_SERVICE_URL=http://localhost:8001
OPENAI_API_KEY=sk-proj-...
OPENAI_MODEL=gpt-3.5-turbo   # or gpt-4o (needs paid credits)

# -- Vector DB (Pinecone) ------------------------------------------
PINECONE_API_KEY=pcsk_...
PINECONE_INDEX=hireiq-candidates

# -- Email (Gmail SMTP) --------------------------------------------
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your@gmail.com
SMTP_PASS=xxxx xxxx xxxx xxxx   # 16-char App Password
EMAIL_FROM=HireIQ <your@gmail.com>

# -- App -----------------------------------------------------------
APP_URL=http://localhost:5173
CORS_ORIGIN=http://localhost:5173
`

### Getting API Keys

| Service | URL | Free Tier |
|---|---|---|
| OpenAI | platform.openai.com |  credit on new accounts |
| Pinecone | pinecone.io | 1 index, 100k vectors |
| Neon | neon.tech | 0.5 GB storage |
| Upstash | upstash.com | 10k commands/day |
| Gmail | myaccount.google.com/apppasswords | Free with 2FA |

---

*Built with React 18, Node.js 20, Python 3.11, GPT-4o, Pinecone, PostgreSQL, Redis*
*HireIQ � College Project 2026*
