# Detagenix AI CRM HRMS

Production-style AI CRM platform for lead management, lead intelligence, and sales operations.

This project combines:
- A React + Vite frontend (dashboard and workflows)
- A FastAPI backend (core CRM APIs)
- ML-based lead scoring (Hot, Warm, Cold)
- AI workflows (chatbot, follow-up email generation, company enrichment, insights)
- Forecasting and analytics (sales forecast, conversion scoring, CLV, conversation intelligence)

---

## 1. What This Project Does

Detagenix AI CRM HRMS helps teams:
- Capture and manage leads
- Score lead quality with ML
- Enrich lead/company profiles with AI and web context
- Generate follow-up emails with LLMs
- Run follow-up optimization recommendations
- Predict conversion probability and client lifetime value
- Analyze sales conversations and extract intelligence
- Forecast pipeline and revenue signals

---

## 2. Current Architecture

```text
Frontend (React + Vite, Port 3000)
        |
        | HTTP JSON / multipart
        v
Backend (FastAPI, Port 8000)
        |
        +--> MongoDB (users, leads, ai_insights, generated_leads, conversation intelligence data)
        +--> ML model artifacts (lead temperature + conversion scoring modules)
        +--> LLM providers (Gemini/OpenAI/Groq/Anthropic, configurable)
        +--> Optional scheduler utility (MongoDB -> Google Sheets sync)
```

Primary backend entrypoint: `backend/services/main.py`

Primary frontend entrypoint: `frontend/src/main.jsx`

---

## 3. Repository Layout (Important Folders)

```text
Detagenix AI CRM HRMS/
|-- README.md
|-- app.py                          # Legacy Flask app (not primary runtime)
|-- requirements.txt
|-- backend/
|   |-- requirements.txt
|   `-- services/
|       |-- main.py                 # Primary FastAPI app
|       |-- start_crm_services.py   # API + periodic sync runner
|       |-- ml_prediction_service.py
|       |-- auth_service.py
|       |-- auth_service_inmemory.py
|       |-- email_generator.py
|       |-- followup_service/
|       |-- client_ltv/
|       |-- chatbot/
|       |-- sales_forecasting/
|       |-- smart lead summary/
|       |-- conversation intelligence engine/
|       |-- lead data enrichment/
|       `-- Lead scoring engine/
|-- frontend/
|   |-- package.json
|   |-- vite.config.js
|   `-- src/
|       |-- App.jsx
|       |-- api/Api.jsx
|       |-- components/
|       |-- pages/
|       `-- services/
`-- ml_model/
    |-- train_model.py
    `-- models/
```

---

## 4. Core Features and How They Work

### 4.1 Authentication (Signup/Login)
- Frontend pages: `Login`, `Signup`
- Backend endpoints: `POST /auth/signup`, `POST /auth/login`
- Flow:
  1. User submits credentials in frontend form.
  2. Backend validates and authenticates.
  3. Token/user payload returned to frontend.
  4. Frontend stores auth context in local storage and navigates to app routes.

### 4.2 Lead Intake + ML Temperature Prediction
- Frontend page: `AddLead`
- Backend endpoint: `POST /predict`
- Flow:
  1. User enters lead details (name/email/role/etc.).
  2. Backend normalizes payload and validates schema.
  3. ML service predicts lead temperature (Hot/Warm/Cold) with confidence.
  4. Lead + prediction are persisted and returned to frontend.

### 4.3 Lead Listing and Drilldown
- Frontend pages: `Dashboard`, `LeadsDetail`, `CandidateProfile`
- Backend endpoints: `GET /leads`, `GET /lead/{id}`, `GET /candidate/{id}`
- Flow:
  1. Dashboard fetches lead list and summary context.
  2. User opens specific lead/candidate details.
  3. Backend returns normalized lead document with ML metadata.

### 4.4 ML Stats and Model Metadata
- Frontend page: `MlStateSample`
- Backend endpoints: `GET /stats`, `GET /model/info`
- Flow:
  1. Frontend requests prediction stats and model info in parallel.
  2. Backend returns aggregate counts/distribution and model metadata.

### 4.5 Lead Generation from Natural Language Query
- Frontend pages: `LeadGeneration`, `LeadDashboard`
- Backend endpoints:
  - `POST /lead-generation/search-query`
  - `POST /lead-generation/qualify-search-results`
  - `GET /lead-generation/dashboard`
- Flow:
  1. User enters a search query.
  2. Backend fetches business results (SerpApi integration).
  3. Conversion scoring module qualifies and categorizes prospects.
  4. Results are optionally persisted in `generated_leads` and shown in dashboard.

### 4.6 Lead Conversion Probability Scoring
- Frontend page: `ConversionLeadScoring`
- Backend endpoints:
  - `POST /lead-scoring/conversion/predict`
  - `POST /lead-scoring/conversion/train`
  - `GET /lead-scoring/conversion/model-info`
- Flow:
  1. User submits conversion factors (budget, response speed, visits, etc.).
  2. Service predicts conversion probability and supporting details.
  3. Optional training endpoint retrains model from historical lead outcomes.

### 4.7 AI Sales Insights
- Frontend page: `AIInsights`
- Backend endpoint: `POST /ai-insights/generate` (multipart/form-data)
- Flow:
  1. User submits transcript text and/or file.
  2. Backend generates structured insights.
  3. Insights are persisted.
  4. Optionally auto-chains into conversation intelligence analysis.

### 4.8 Conversation Intelligence
- Used in `Dashboard` and `CandidateProfile`
- Backend endpoints:
  - `POST /conversation-intelligence/analyze`
  - `GET /conversation-intelligence/lead/{lead_id}`
  - `GET /conversation-intelligence/overview`
- Flow:
  1. Conversation text/messages are analyzed.
  2. Intent, risk, sentiment, and coaching-style metrics are computed.
  3. Data can be fetched by lead and as an overview for dashboard widgets.

### 4.9 Follow-up Optimizer
- Frontend page: `FollowupOptimizer`
- Backend endpoint: `POST /followup/optimize`
- Flow:
  1. Frontend loads leads and interaction history payload.
  2. Backend processes time/channel patterns.
  3. Recommendation engine returns best day/time/channel with rationale.

### 4.10 Smart Email Generator
- Used from lead profile workflows
- Backend endpoint: `POST /email/generate-followup`
- Flow:
  1. User chooses lead and context (deal stage + past communication).
  2. Backend fetches lead details and builds prompt.
  3. LLM generates subject/body.
  4. Formatted response returns to frontend.

### 4.11 Company/Lead Enrichment
- Used in `CandidateProfile`
- Backend endpoint: `POST /lead-enrichment/enrich-company`
- Flow:
  1. User sends company data.
  2. Backend discovers website/domain (if needed), scrapes context, and applies AI summarization.
  3. Intelligence output returns with supporting search context metadata.

### 4.12 Sales Forecasting
- Frontend page: `SalesForecasting`
- Backend endpoint: `GET /sales-forecast`
- Flow:
  1. Frontend requests forecast with configurable lookback/options.
  2. Backend aggregates lead and pipeline data.
  3. Forecast payload returns with trend metrics.

### 4.13 Client Lifetime Value (CLV)
- Frontend page: `ClientLtvPrediction`
- Backend endpoint: `POST /clv/predict`
- Flow:
  1. User selects or enters customer profile + purchase behavior.
  2. CLV predictor computes expected value, confidence, upsell/cross-sell recommendations.
  3. Structured business guidance is returned.

### 4.14 Chatbot Tool Router
- Frontend page: `Chatbot`
- Backend endpoint: `POST /chatbot/chat`
- Flow:
  1. User asks natural-language CRM question.
  2. Gemini-based tool router selects CRM tool (`get_leads`, `add_lead`, etc.) or general assistant response.
  3. Result + conversation memory are returned for multi-turn interactions.

---

## 5. Frontend Route Map

Public routes:
- `/`
- `/login`
- `/signup`

Application routes:
- `/dashboard`
- `/addleads`
- `/mlstats`
- `/lead-generation`
- `/lead-dashboard`
- `/lead-scoring-conversion`
- `/ai-insights`
- `/sales-forecasting`
- `/client-ltv`
- `/followup-optimizer`
- `/chatbot`
- `/lead/:id`
- `/lead/edit/:id`
- `/candidate/:candidate_id`
- `/profile`

---

## 6. Backend API Catalog (Current)

### Health and System
- `GET /`
- `GET /health`

### Auth
- `POST /auth/signup`
- `POST /auth/login`
- Legacy aliases also supported:
  - `POST /api/auth/signup`
  - `POST /api/auth/login`

### Lead and ML Temperature
- `POST /predict`
- `GET /lead/{unique_id}`
- `GET /candidate/{candidate_id}`
- `GET /leads`
- `GET /leads/hot`
- `GET /leads/warm`
- `GET /leads/cold`
- `GET /leads/temperature/{temperature}`
- `POST /batch-predict`
- `GET /stats`
- `GET /model/info`

### Lead Generation + Conversion Scoring
- `POST /lead-generation/search-query`
- `POST /lead-generation/qualify-search-results`
- `GET /lead-generation/dashboard`
- `POST /lead-scoring/conversion/predict`
- `GET /lead-scoring/conversion/model-info`
- `POST /lead-scoring/conversion/train`

### Sales Intelligence
- `POST /ai-insights/generate`
- `POST /conversation-intelligence/analyze`
- `GET /conversation-intelligence/lead/{lead_id}`
- `GET /conversation-intelligence/overview`
- `GET /sales-forecast`

### CRM AI Workflows
- `POST /lead-enrichment/enrich-company`
- `POST /email/generate-followup`
- `POST /followup/optimize`
- `POST /clv/predict`
- `POST /chatbot/chat`

Interactive API docs:
- Swagger UI: `http://localhost:8000/docs`

---

## 7. Setup and Run (Windows)

### 7.1 Prerequisites
- Python 3.10+
- Node.js 18+
- npm 9+
- MongoDB (Atlas or local)

### 7.2 Create Virtual Environment and Install Dependencies

From project root:

```cmd
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
pip install -r backend\requirements.txt
```

Install frontend dependencies:

```cmd
cd frontend
npm install
cd ..
```

### 7.3 Start Backend (Primary)

Option A (from project root):

```cmd
python backend\services\main.py
```

Option B (as requested, from services folder):

```cmd
cd backend\services
python main.py
```

Backend available at:
- API: `http://localhost:8000`
- Docs: `http://localhost:8000/docs`

### 7.4 Start Frontend

```cmd
cd frontend
npm run dev
```

Frontend available at:
- `http://localhost:3000`

### 7.5 Run API + Scheduler Together (Optional)

```cmd
cd backend\services
python start_crm_services.py
```

This starts:
- FastAPI backend
- Periodic sync runner (MongoDB to Google Sheets utility)

---

## 8. Environment Variables

Create a `.env` file in project root.

### Required for normal backend operation

```env
MONGODB_URI=your_mongodb_connection_string
DB_NAME=ai_crm_db
JWT_SECRET_KEY=replace_with_a_secure_secret
```

### Useful optional variables

```env
# Dev fallback auth
SKIP_MONGODB=false

# LLM routing
LLM_PROVIDER=openai
LLM_API_KEY=
LLM_MODEL=gpt-4o-mini

# Provider-specific keys
OPENAI_API_KEY=
GEMINI_API_KEY=
GOOGLE_API_KEY=
GROQ_API_KEY=
ANTHROPIC_API_KEY=

# Gemini/chatbot tuning
GEMINI_MODEL=gemini-2.5-flash
GEMINI_TIMEOUT_SECONDS=30

# AI insights / transcription
OPENAI_LLM_MODEL=gpt-4o-mini
OPENAI_WHISPER_MODEL=whisper-1

# Conversation intelligence auto-chain
CONV_INTELLIGENCE_AUTO_FROM_AI_INSIGHTS=true

# Mongo connection tuning
MONGO_SERVER_SELECTION_TIMEOUT_MS=7000
MONGO_CONNECT_TIMEOUT_MS=7000
MONGO_SOCKET_TIMEOUT_MS=12000

# Google Sheets sync (optional)
GOOGLE_SPREADSHEET_NAME=CRM form (Responses)
GOOGLE_WORKSHEET_NAME=Form Responses 1
MONGODB_COLLECTION_NAME=leads
```

---

## 9. Developer Notes

### Primary runtime to use
- Use `backend/services/main.py` for backend runtime.
- `app.py` in project root is a legacy Flask app and not the primary service used by the frontend.

### Frontend API base behavior
- Axios base URL is configured as `http://localhost:8000`.
- Vite proxy for `/api` to `8000` exists, but current service layer primarily uses direct base URL calls.

### Known integration note
- `frontend/src/pages/EditLead.jsx` currently points to `http://localhost:8001` via direct `fetch`, which differs from the main backend port `8000`.

---

## 10. Troubleshooting

### Backend not reachable from frontend
- Confirm backend is running on `8000`.
- Open `http://localhost:8000/docs` directly.
- Check terminal for import/model/mongo errors.

### MongoDB errors at startup
- Verify `MONGODB_URI` and network access.
- For local UI testing without DB, set `SKIP_MONGODB=true` (auth fallback only).

### AI endpoints returning placeholder or provider errors
- Set provider keys in `.env` (`OPENAI_API_KEY`, `GEMINI_API_KEY`, etc.).
- Verify `LLM_PROVIDER` and model names are valid for your account.

### Frontend starts but data is empty
- Validate backend is up and database has records.
- Confirm API calls in browser dev tools return `success: true` payloads.

---

## 11. License

Licensed under the MIT License. See `LICENSE` for details.
