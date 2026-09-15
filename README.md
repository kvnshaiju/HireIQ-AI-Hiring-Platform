# HireIQ — AI-Powered Hiring Platform

> An AI-powered, full-stack recruitment platform that helps recruiters automate resume screening, candidate matching, interview workflows, analytics, and candidate communication.

![Project Status](https://img.shields.io/badge/status-college%20project-blue)
![Frontend](https://img.shields.io/badge/frontend-React%20%2B%20Vite-61DAFB)
![Backend](https://img.shields.io/badge/backend-Fastify%20%2B%20TypeScript-000000)
![AI](https://img.shields.io/badge/AI-Python%20%2B%20FastAPI-orange)
![Database](https://img.shields.io/badge/database-PostgreSQL-336791)

---

## 📌 Overview

HireIQ is a full-stack AI hiring platform designed to streamline the recruitment process.

The platform combines a React-based web application, a Node.js API server, and a separate Python AI service. It can process resumes, extract candidate information, calculate job-candidate match scores, conduct conversational AI interviews, analyze job descriptions, provide recruitment analytics, and generate candidate feedback.

The project is designed as an ATS-style application with AI capabilities integrated throughout the hiring workflow.

### 🎯 Problem Statement

Traditional recruitment can involve:

- Manual resume screening
- Time-consuming candidate shortlisting
- Subjective candidate evaluation
- Back-and-forth interview scheduling
- Limited recruitment analytics
- Inconsistent candidate feedback
- Difficulty identifying potentially biased job descriptions

### 💡 Proposed Solution

HireIQ automates major parts of the recruitment workflow using LLMs, embeddings, vector search, rule-based checks, and structured application workflows.

---

## ✨ Key Features

### 👨‍💼 Recruiter Features

- Create, edit, publish, and manage job postings
- AI-assisted job description analysis and optimization
- Candidate management through a Kanban-style pipeline
- Automated resume parsing and skill extraction
- AI-assisted resume evaluation
- Semantic candidate matching
- Candidate ranking
- Interview scheduling and email notifications
- Conversational AI interviews
- Post-interview AI evaluation
- Recruitment analytics dashboard
- Bias monitoring using the 4/5ths (80%) rule
- AI-generated candidate rejection feedback
- Recruiter AI reasoning alongside evaluation scores

### 👤 Candidate Features

- Browse available jobs
- Apply using PDF or DOCX resumes
- Track application status
- View match-score information
- Manage candidate profile and skills
- Participate in conversational AI interviews
- Receive email notifications

---

## 🤖 AI Capabilities

HireIQ uses AI across several parts of the recruitment process:

| Capability | Description |
|---|---|
| Resume Parsing | Extracts structured candidate information from resumes |
| Skill Extraction | Identifies skills, experience, education, and certifications |
| Resume Evaluation | Evaluates candidates across multiple dimensions |
| Semantic Matching | Compares candidate and job meaning using embeddings |
| Candidate Ranking | Ranks candidates based on matching signals |
| AI Interviewer | Conducts structured conversational interviews |
| Interview Evaluation | Evaluates interview responses and provides evidence-based scores |
| JD Analysis | Detects potential bias and analyzes job descriptions |
| Bias Monitoring | Applies the 4/5ths rule to selection pass rates |
| Candidate Feedback | Generates personalized rejection feedback |
| AI Predictions | Provides candidate success-related predictions |

---

## 🏗️ System Architecture

```text
                         ┌─────────────────────────────┐
                         │          Browser            │
                         │    React + Vite + Tailwind  │
                         │                             │
                         │  Recruiter Dashboard        │
                         │  Candidate Portal           │
                         └──────────────┬──────────────┘
                                        │
                              HTTP / WebSocket
                                        │
                         ┌──────────────▼──────────────┐
                         │       Node.js API            │
                         │    Fastify + TypeScript      │
                         │                              │
                         │ Auth | Jobs | Applications   │
                         │ Interviews | Analytics       │
                         └───────┬─────────┬────────────┘
                                 │         │
                     ┌───────────┘         └──────────────┐
                     │                                    │
          ┌──────────▼──────────┐              ┌─────────▼─────────┐
          │ Python AI Service   │              │ PostgreSQL        │
          │ FastAPI + OpenAI    │              │ Prisma ORM        │
          │ Pinecone            │              │                   │
          └──────────┬──────────┘              └───────────────────┘
                     │
              ┌──────▼───────┐
              │ OpenAI API   │
              │ LLMs +       │
              │ Embeddings   │
              └──────────────┘

          Redis + BullMQ
          └── Asynchronous resume processing
```

---

## 🔄 Resume Processing Workflow

```text
Candidate uploads resume
          │
          ▼
API validates PDF/DOCX and file size
          │
          ▼
Resume stored locally
          │
          ▼
Application created
          │
          ▼
BullMQ job added to Redis
          │
          ▼
Background worker processes resume
          │
          ├── Extract resume text
          ├── Parse structured candidate data
          ├── Generate embeddings
          ├── Store vector in Pinecone
          ├── Evaluate candidate against job
          ├── Calculate match score
          └── Save results to PostgreSQL
          │
          ▼
Application status → SCREENED
          │
          ▼
Recruiter sees ranked candidates
```

### Hybrid Matching

The matching engine combines semantic similarity and skill coverage:

```text
Overall Score = (0.55 × Semantic Similarity)
              + (0.45 × Skill Coverage)
```

This approach combines meaning-based matching with explicit skill matching.

---

## 🧰 Tech Stack

### Frontend

| Technology | Purpose |
|---|---|
| React | User interface |
| Vite | Development and build tooling |
| TypeScript | Type safety |
| TailwindCSS | Styling |
| Zustand | Global state management |
| React Query | Server state and caching |
| React Router | Client-side routing |
| Recharts | Analytics charts |
| Radix UI | UI primitives |
| Socket.io Client | Real-time communication |
| Axios | HTTP requests |

### Backend

| Technology | Purpose |
|---|---|
| Node.js | Runtime |
| Fastify | API framework |
| TypeScript | Type safety |
| Prisma | ORM and database access |
| PostgreSQL | Primary database |
| BullMQ | Background job processing |
| Redis | Queue and caching layer |
| JWT | Authentication |
| bcrypt | Password hashing |
| Zod | Request validation |
| Nodemailer | Email delivery |
| Winston | Logging |

### AI Service

| Technology | Purpose |
|---|---|
| Python | AI service runtime |
| FastAPI | AI service API |
| OpenAI SDK | LLM and embedding integration |
| pdfplumber | PDF text extraction |
| docx2txt | DOCX text extraction |
| Pinecone | Vector database |
| scikit-learn | Similarity calculations |
| NumPy | Numerical operations |
| Pydantic | Data validation |

### Infrastructure

| Service | Purpose |
|---|---|
| Neon PostgreSQL | Cloud database |
| Upstash Redis | Cloud Redis |
| Pinecone | Vector search |
| Gmail SMTP | Transactional email |
| Local filesystem | Development resume storage |

---

## 📂 Project Structure

A simplified project structure:

```text
hireiq/
├── apps/
│   ├── api/                 # Node.js/Fastify backend
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   ├── plugins/
│   │   │   └── server.ts
│   │   └── prisma/
│   │
│   ├── ai/                  # Python/FastAPI AI service
│   │   ├── routers/
│   │   ├── services/
│   │   ├── main.py
│   │   └── requirements.txt
│   │
│   └── web/                 # React frontend
│       └── src/
│           ├── pages/
│           ├── components/
│           ├── stores/
│           ├── lib/
│           └── types/
│
├── README.md
└── .gitignore
```

---

## 🔐 Authentication & Authorization

HireIQ uses JWT-based authentication and role-based access control.

### Supported Roles

| Role | Access |
|---|---|
| Recruiter | Jobs, candidates, interviews, analytics |
| Hiring Manager | Recruiter-level hiring functionality |
| Admin | Hiring functionality plus organization settings |
| Candidate | Jobs, own applications, profile, interviews |

The API validates authentication and authorization before protected operations.

---

## ⚙️ Asynchronous Processing

Resume processing can involve file extraction, LLM requests, embedding generation, vector database operations, and database updates.

To avoid blocking API requests, HireIQ uses:

```text
API Request
    │
    ▼
Validate + Save Application
    │
    ▼
Add BullMQ Job
    │
    ▼
Return 202 Accepted
    │
    ▼
Background Worker
    │
    ▼
AI Processing
    │
    ▼
Database Update
```

The queue also supports retry attempts with exponential backoff.

---

## 🗄️ Database Design

The application uses PostgreSQL with Prisma ORM.

Major entities include:

```text
Organization
    │
    ├── Users
    └── Jobs
          │
          └── Applications
                 │
                 ├── Candidate
                 ├── Interviews
                 └── AI Evaluations

Skills
    ├── Candidate Skills
    └── Job Skills
```

The application model stores match scores, rankings, AI reasoning, and candidate feedback.

---

## 📊 Application Pipeline

```text
APPLIED
   │
   ▼
SCREENED
   │
   ▼
SHORTLISTED
   │
   ▼
INTERVIEWING
   │
   ▼
OFFER
   │
   ▼
HIRED
```

Alternative outcomes include:

```text
SCREENED → REJECTED
APPLIED  → WITHDRAWN
```

---

## 📈 Analytics

The recruiter dashboard includes information such as:

- Total jobs
- Total applications
- Average match score
- Hiring funnel data
- Applications over time
- Candidate sources
- Offer acceptance information
- Bias/fairness reports
- Candidate success predictions

---

## 🛡️ Security Considerations

The project includes several security-related measures:

- JWT-based authentication
- bcrypt password hashing
- Role-based authorization
- Zod request validation
- File type and size validation
- Prisma parameterized database access
- Organization-scoped recruiter queries
- Protected environment variables
- AI-generated feedback safety checks

> **Important:** AI-generated hiring decisions should be treated as decision-support rather than a replacement for qualified human judgment. Fairness checks are monitoring tools and do not guarantee the absence of bias.

---

## 🚀 Getting Started

### Prerequisites

Install the following:

- Node.js
- npm
- Python 3.11+
- PostgreSQL or a PostgreSQL-compatible cloud database
- Redis
- Required API credentials for OpenAI and Pinecone if those services are enabled

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd hireiq
```

### 2. Configure Environment Variables

Create environment files from the provided example files.

For example:

```text
apps/api/.env
apps/ai/.env
apps/web/.env
```

**Never commit real API keys, passwords, database credentials, SMTP credentials, or JWT secrets to GitHub.**

Use placeholder values in `.env.example`.

### 3. Install Backend Dependencies

```bash
cd apps/api
npm install
```

### 4. Install AI Service Dependencies

```bash
cd ../ai
python -m venv venv
```

Windows:

```powershell
venv\Scripts\activate
pip install -r requirements.txt
```

### 5. Install Frontend Dependencies

```bash
cd ../web
npm install
```

### 6. Start the API

From `apps/api`:

```bash
npm run dev
```

The development API is configured to run on:

```text
http://localhost:3001
```

### 7. Start the AI Service

From `apps/ai`:

```bash
python -m uvicorn main:app --reload --port 8001
```

AI service:

```text
http://localhost:8001
```

FastAPI documentation:

```text
http://localhost:8001/docs
```

### 8. Start the Frontend

From `apps/web`:

```bash
npm run dev
```

The frontend is configured to run on:

```text
http://localhost:5173
```

---

## 🔑 Environment Variables

Create a `.env.example` file containing placeholders such as:

```env
# API
PORT=3001
JWT_SECRET=your_jwt_secret

# Database
DATABASE_URL=your_database_url

# Redis
REDIS_URL=your_redis_url

# AI
AI_SERVICE_URL=http://localhost:8001
OPENAI_API_KEY=your_openai_api_key
OPENAI_MODEL=your_model_name

# Pinecone
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_INDEX=your_pinecone_index

# Email
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=your_email
SMTP_PASS=your_app_password
EMAIL_FROM=your_email

# Frontend
APP_URL=http://localhost:5173
CORS_ORIGIN=http://localhost:5173
```

### ⚠️ Never Upload Secrets

Do **not** commit:

```text
.env
.env.local
API keys
Database passwords
SMTP passwords
JWT secrets
Private credentials
Real candidate resumes or personal data
```

Add them to `.gitignore`.

---

## 🔌 API Overview

Base URL:

```text
http://localhost:3001/api/v1
```

### Authentication

```text
POST /auth/register
POST /auth/login
GET  /auth/me
POST /auth/verify-email
POST /auth/resend-verification
POST /auth/forgot-password
POST /auth/reset-password
```

### Jobs

```text
GET    /jobs
GET    /jobs/public
POST   /jobs
GET    /jobs/:id
PATCH  /jobs/:id
DELETE /jobs/:id
POST   /jobs/analyze-jd
GET    /jobs/:id/candidates
POST   /jobs/:id/shortlist
GET    /jobs/:id/bias-report
```

### Applications

```text
POST  /applications
GET   /applications/my
GET   /applications/:id
PATCH /applications/:id/status
GET   /applications/:id/evaluation
POST  /applications/:id/feedback
```

### Candidates

```text
GET  /candidates
GET  /candidates/me
POST /candidates/me/resume
POST /candidates/me/skills
```

### Interviews

```text
GET  /interviews
POST /interviews
GET  /interviews/:id
GET  /interviews/my
WS   /interviews/:id/stream
```

### Analytics

```text
GET /analytics/overview
GET /analytics/bias
GET /analytics/predictions/:jobId
GET /analytics/applications-over-time
```

---

## 🧪 Testing

Before publishing the repository, verify:

- [ ] Frontend starts successfully
- [ ] API starts successfully
- [ ] AI service starts successfully
- [ ] Database connection works
- [ ] Redis connection works
- [ ] Resume upload works
- [ ] AI processing works
- [ ] Candidate matching works
- [ ] Authentication works
- [ ] Recruiter and candidate roles work correctly
- [ ] No secrets are present in Git history
- [ ] No real candidate/resume data is included

---

## 📸 Screenshots

Add screenshots of the application here after uploading them to the repository.

Recommended screenshots:

1. Recruiter Dashboard
2. Job Management
3. Candidate Pipeline
4. AI Resume Evaluation
5. Candidate Profile
6. AI Interview
7. Analytics Dashboard
8. Candidate Portal

Example:

```markdown
![Recruiter Dashboard](docs/screenshots/dashboard.png)
```

---

## 🔮 Future Improvements

Possible future improvements include:

- Cloud-based resume storage
- Production deployment
- More advanced candidate recommendation models
- Improved fairness and bias evaluation
- Automated testing and CI/CD
- Monitoring and observability
- More integrations with recruitment platforms
- More robust evaluation and model validation
- Improved data privacy controls

---

## 📚 Project Highlights

HireIQ demonstrates practical implementation of:

- Full-stack web development
- REST API development
- Microservice architecture
- LLM integration
- Resume/document processing
- Embeddings and vector search
- Semantic similarity
- Background job processing
- Database design
- Authentication and authorization
- Real-time AI interaction
- Recruitment analytics
- AI-assisted fairness monitoring

---

## ⚠️ Disclaimer

HireIQ is a college/academic project created to demonstrate full-stack development and AI engineering concepts.

AI-generated candidate scores, predictions, interview evaluations, and fairness indicators should not be treated as definitive hiring decisions. Real-world recruitment systems require appropriate human oversight, validation, privacy protections, legal review, and responsible AI practices.

---

## 👨‍💻 Project

**HireIQ — AI-Powered Hiring Platform**

Built as a college project using React, Node.js, Python, FastAPI, PostgreSQL, Redis, Pinecone, and OpenAI technologies.

---

## ⭐ If You Find This Project Interesting

Feel free to explore the code, review the architecture, and learn from the implementation.

