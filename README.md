# RoleReady AI

**AI-powered interview preparation for your next role.**

Interview AI is a full-stack web application that helps job seekers prepare for interviews by analyzing a target job description together with their resume or self-description. Google Gemini generates a personalized interview strategy—including likely questions, skill gap analysis, a day-by-day preparation plan, and an optional tailored resume PDF.

![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-7-646CFF?logo=vite&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-Express_5-339933?logo=nodedotjs&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose-47A248?logo=mongodb&logoColor=white)
![Gemini](https://img.shields.io/badge/Google-Gemini-4285F4?logo=google&logoColor=white)

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [How It Works](#how-it-works)
- [Screenshots / Demo](#screenshots--demo)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [API Documentation](#api-documentation)
- [Authentication Flow](#authentication-flow)
- [AI Integration](#ai-integration)
- [Database Design](#database-design)
- [Environment Variables](#environment-variables)
- [Installation & Local Setup](#installation--local-setup)
- [CORS / Local Development Configuration](#cors--local-development-configuration)
- [Usage](#usage)
- [Error Handling](#error-handling)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [Future Improvements](#future-improvements)
- [Challenges / Engineering Highlights](#challenges--engineering-highlights)
- [Contributing](#contributing)
- [License](#license)
- [Author](#author)

---

## Overview

Interview AI is built for **job seekers, career switchers, and students** who want structured, role-specific interview preparation instead of generic question lists.

### What problem it solves

Preparing for a specific role is difficult when practice material is generic. Interview AI combines your profile (resume or self-description) with a target job description and uses AI to produce a focused preparation plan: likely questions, skill gaps, and a day-by-day roadmap.

### How AI is used

After authentication, the user submits context through the home page. The backend extracts resume text (from PDF uploads), combines it with the job description and optional self-description, and sends that context to **Google Gemini**. Gemini returns a **structured JSON report** that is validated against Zod schemas, saved in MongoDB, and displayed in the frontend.

### How the workflow works

This is an **interview preparation planner**, not a live mock interview. Users do not answer questions inside the app and receive automated scoring. Instead, the app generates questions with model answers, identifies skill gaps, and optionally produces a tailored resume PDF.

---

## Key Features

### User registration, login, and logout

Users register with username, email, and password, then log in to access the app. The backend exposes a logout API that blacklists the JWT and clears the session cookie. There is **no logout button in the current UI**, but the `/api/auth/logout` endpoint is implemented.

### JWT authentication

Sessions use JSON Web Tokens signed with `JWT_SECRET`, stored in cookies and sent automatically by Axios with `withCredentials: true`.

### Protected routes

The home page (`/`) and interview report page (`/interview/:interviewId`) are wrapped in a `Protected` component that redirects unauthenticated users to `/login`.

### Resume upload and processing

Users can upload a resume from the home page. The backend accepts files via Multer (in-memory storage, **3 MB** limit) and extracts text using **pdf-parse**. The UI also advertises `.docx` acceptance, but only PDF parsing is implemented server-side.

### Job description input

Users paste a target job description (textarea with a 5,000-character limit in the UI). This text is a primary input for AI question generation, match scoring, and resume tailoring.

### AI-generated interview questions

Gemini generates **technical** and **behavioral** questions tailored to the role. Each question includes the interviewer's **intention** and a **model answer** describing how to approach the response.

### Interview reports

Each generation produces a persisted report containing match score, questions, skill gaps, preparation plan, and inferred job title. Reports are stored per user in MongoDB.

### Strength / weakness identification (skill gaps)

The AI returns a `skillGaps` array with severity levels (`low`, `medium`, `high`), helping users understand where their profile may fall short for the target role.

### AI-generated feedback (preparation guidance)

Rather than scoring live answers, the app provides **preparation feedback**: model answers, a multi-day roadmap, and an overall **match score** (0–100).

### Interview history

The home page lists recent reports with title, date, and match score. The list API omits heavy fields (full question arrays, raw resume/JD text) for lighter responses.

### PDF generation

Users can download an AI-tailored resume PDF from a report page. Gemini generates HTML; Puppeteer converts it to a downloadable PDF.

### Dashboard-style home page

The home page serves as the main hub for creating new interview plans and browsing past reports.

### MongoDB persistence

Users, interview reports, and blacklisted tokens are stored in MongoDB via Mongoose.

### Responsive frontend

The React UI uses SCSS with a card-based home layout, sidebar navigation on report pages, and styled auth forms.

### Not implemented (verified from code)

The following are **not** present in the current codebase:

- Real-time or live AI interview sessions
- Voice / audio interaction
- Turn-by-turn Q&A with answer submission and evaluation
- Automated scoring of user responses during an interview
- Dedicated admin panel or separate dashboard beyond the home page

---

## How It Works

```text
User
  ↓
Authentication (register / login → JWT cookie)
  ↓
Home — Job Description + Resume PDF and/or Self Description
  ↓
Interview report creation (POST /api/interview/)
  ↓
PDF text extraction + AI processing (Gemini)
  ↓
Structured report saved to MongoDB
  ↓
Report viewer — questions, skill gaps, preparation roadmap
  ↓
Optional — tailored resume PDF download (Puppeteer)
```

There is **no live interview session or post-session evaluation step** in the current flow. Preparation guidance is generated upfront and reviewed on the report page.

### Lifecycle from the user's perspective

1. Create an account or log in.
2. On the home page, paste the target job description.
3. Upload a resume PDF and/or enter a self-description.
4. Click **Generate My Interview Strategy** and wait while Gemini processes the request (~30 seconds per UI hint).
5. Review the report: match score, skill gaps, technical/behavioral questions, and preparation roadmap.
6. Optionally click **Download Resume** for a tailored PDF.
7. Return home to open previous reports from **My Recent Interview Plans**.

---

## Screenshots / Demo

Screenshots are not included in the repository yet.

<!-- Add dashboard screenshot here -->

<!-- Add home page screenshot here -->

<!-- Add interview report screenshot here -->

<!-- Add preparation roadmap screenshot here -->

---

## Tech Stack

### Frontend

| Technology | Purpose |
| --- | --- |
| **React 19** | UI components and page rendering |
| **Vite 7** | Dev server (default port 5173) and production bundling |
| **React Router 7** | Client-side routing |
| **Axios** | HTTP client with cookie credentials |
| **SCSS / Sass** | Component and page styling |

### Backend

| Technology | Purpose |
| --- | --- |
| **Node.js** | Server runtime |
| **Express 5** | REST API, middleware, route mounting |
| **Multer** | Multipart file upload for resumes |
| **cookie-parser** | Read JWT from cookies |
| **CORS** | Cross-origin requests from the Vite dev server |
| **dotenv** | Load environment variables from `Backend/.env` |

### Database

| Technology | Purpose |
| --- | --- |
| **MongoDB** | Persistent document storage |
| **Mongoose** | User, report, and blacklist schemas/models |

### AI

| Technology | Purpose |
| --- | --- |
| **@google/genai** | Google GenAI SDK (`GoogleGenAI` client) |
| **gemini-3-flash-preview** | Model for interview reports and resume HTML |
| **Puppeteer** | Headless browser PDF rendering for resume export |

### Authentication

| Technology | Purpose |
| --- | --- |
| **jsonwebtoken** | JWT sign/verify (1-day expiry) |
| **bcryptjs** | Password hashing (10 salt rounds) |
| **HTTP cookies** | Session token storage after login/register |

### Validation / Utilities

| Technology | Purpose |
| --- | --- |
| **Zod** | Defines expected Gemini output shapes |
| **zod-to-json-schema** | Converts Zod schemas to Gemini `responseSchema` |
| **pdf-parse** | Extracts text from uploaded PDF resumes |

---

## Architecture

```text
┌──────────────────────────┐
│   React Frontend (Vite)  │
│   http://localhost:5173  │
└────────────┬─────────────┘
             │ HTTP + Axios (withCredentials)
             ▼
┌──────────────────────────┐
│   Express REST API       │
│   http://localhost:3000  │
└────────────┬─────────────┘
             │
      ┌──────┴──────┐
      ▼             ▼
  MongoDB      Google Gemini API
  (Mongoose)   (@google/genai)
                    │
                    ▼
               Puppeteer (PDF export)
```

### Data flow

1. Frontend sends authenticated requests with cookies to Express.
2. `authUser` middleware verifies JWT and checks the token blacklist.
3. Resume uploads are parsed; text is combined with job and self descriptions.
4. `ai.service.js` calls Gemini with JSON schema constraints.
5. Parsed AI output is saved as an `InterviewReport` document.
6. Resume PDF flow reuses stored report data, generates HTML via Gemini, then renders PDF with Puppeteer.

---

## Project Structure

```text
RoleReady-ai-main/
├── Backend/
│   ├── server.js                 # Entry point, DNS config, port 3000
│   ├── package.json
│   └── src/
│       ├── app.js                # Express app, CORS, routes
│       ├── config/
│       │   └── database.js       # MongoDB connection (MONGO_URI)
│       ├── controllers/
│       │   ├── auth.controller.js
│       │   └── interview.controller.js
│       ├── middlewares/
│       │   ├── auth.middleware.js
│       │   └── file.middleware.js
│       ├── models/
│       │   ├── user.model.js
│       │   ├── interviewReport.model.js
│       │   └── blacklist.model.js
│       ├── routes/
│       │   ├── auth.routes.js
│       │   └── interview.routes.js
│       └── services/
│           └── ai.service.js
│
├── Frontend/
│   ├── index.html
│   ├── vite.config.js
│   ├── package.json
│   ├── public/
│   └── src/
│       ├── main.jsx
│       ├── App.jsx
│       ├── app.routes.jsx
│       ├── style.scss
│       └── features/
│           ├── auth/             # Login, register, Protected route
│           └── interview/        # Home, report viewer, API hooks
│
└── README.md
```

| Directory | Purpose |
| --- | --- |
| `Backend/src/controllers` | Auth and interview HTTP handlers |
| `Backend/src/services` | Gemini integration and Puppeteer PDF export |
| `Backend/src/models` | Mongoose schemas |
| `Frontend/src/features/auth` | Auth context, hooks, pages, route guard |
| `Frontend/src/features/interview` | Report creation, listing, viewing, PDF download |

---

## API Documentation

**Base URL (local):** `http://localhost:3000`

### Authentication APIs

| Method | Endpoint | Authentication | Description |
| --- | --- | --- | --- |
| `POST` | `/api/auth/register` | No | Register a new user |
| `POST` | `/api/auth/login` | No | Authenticate user |
| `GET` | `/api/auth/logout` | No* | Logout and blacklist token |
| `GET` | `/api/auth/get-me` | Yes | Get logged-in user |

\* Logout is public but reads the cookie if present.

#### `POST /api/auth/register`

**Body (JSON):** `username`, `email`, `password`

**Response (`201`):** `{ message, user: { id, username, email } }` + JWT cookie

#### `POST /api/auth/login`

**Body (JSON):** `email`, `password`

**Response (`200`):** `{ message, user: { id, username, email } }` + JWT cookie

#### `GET /api/auth/get-me`

**Response (`200`):** `{ message, user: { id, username, email } }`

---

### Interview & Report APIs

| Method | Endpoint | Authentication | Description |
| --- | --- | --- | --- |
| `POST` | `/api/interview/` | Yes | Generate and save a new interview report |
| `GET` | `/api/interview/` | Yes | List user's reports (summary fields) |
| `GET` | `/api/interview/report/:interviewId` | Yes | Fetch full report by ID |
| `POST` | `/api/interview/resume/pdf/:interviewReportId` | Yes | Generate tailored resume PDF |

#### `POST /api/interview/`

**Content-Type:** `multipart/form-data`

| Field | Type | Description |
| --- | --- | --- |
| `resume` | File | Resume file (parsed as PDF on server) |
| `jobDescription` | String | Target job description |
| `selfDescription` | String | Optional profile summary |

**Response (`201`):** `{ message, interviewReport }`

**Note:** The controller parses `req.file.buffer` as PDF. Requests without an uploaded file may fail at runtime even if `selfDescription` is provided.

#### `GET /api/interview/`

**Response (`200`):** `{ message, interviewReports }` — excludes `resume`, `selfDescription`, `jobDescription`, `technicalQuestions`, `behavioralQuestions`, `skillGaps`, and `preparationPlan`.

#### `GET /api/interview/report/:interviewId`

**Response (`200`):** `{ message, interviewReport }` — full report for the authenticated owner.

**Errors:** `404` if not found or not owned by user.

#### `POST /api/interview/resume/pdf/:interviewReportId`

**Response:** Binary PDF (`Content-Type: application/pdf`)

**Errors:** `404` if report ID not found.

---

## Authentication Flow

1. **Registration** — Validates fields, checks duplicate username/email, bcrypt-hashes password, creates user, signs JWT, sets cookie.
2. **Login** — Finds user by email, compares password hash, signs JWT, sets cookie.
3. **Session restore** — On load, frontend calls `GET /api/auth/get-me` via `useAuth`.
4. **Protected API routes** — `authUser` middleware reads `req.cookies.token`, rejects blacklisted tokens, verifies JWT, sets `req.user`.
5. **Protected UI routes** — `Protected` component redirects to `/login` if no user.
6. **Logout** — Token added to `blacklistTokens` collection; cookie cleared.

**Token details:** JWT payload includes `{ id, username }`, expires in **1 day**.

**Cookie details:** Set via `res.cookie("token", token)` without explicit `httpOnly`, `secure`, or `sameSite` configuration.

---

## AI Integration

### SDK and model

| Item | Value |
| --- | --- |
| SDK | `@google/genai` |
| Client | `new GoogleGenAI({ apiKey: process.env.GOOGLE_GENAI_API_KEY })` |
| Model | `gemini-3-flash-preview` |
| Service file | `Backend/src/services/ai.service.js` |

### Where Gemini is called

| Function | Purpose |
| --- | --- |
| `generateInterviewReport` | Main interview preparation report |
| `generateResumePdf` | Tailored resume HTML → Puppeteer PDF |

### Information sent to the AI

**Interview report generation:**

- Extracted resume text (from uploaded PDF)
- `selfDescription` (form field)
- `jobDescription` (form field)

**Resume PDF generation:**

- Stored `resume`, `selfDescription`, and `jobDescription` from an existing report

### How interview questions are generated

Gemini receives the combined candidate and role context in a prompt and returns structured JSON constrained by a Zod schema. Questions are generated **proactively** as part of the report—not dynamically during a live session. Each question includes `question`, `intention`, and `answer` (model guidance).

### How responses are evaluated

**User responses are not evaluated.** There is no endpoint or UI for submitting answers to interview questions. The app does not score, grade, or provide feedback on user-provided answers. Feedback is limited to AI-generated model answers and preparation guidance in the initial report.

### Structured output handling

```text
Zod schema → zodToJsonSchema → Gemini responseSchema
           → responseMimeType: "application/json"
           → JSON.parse(response.text)
```

**Interview report fields:** `matchScore`, `technicalQuestions[]`, `behavioralQuestions[]`, `skillGaps[]`, `preparationPlan[]`, `title`

**Resume PDF fields:** `html` (rendered by Puppeteer)

### Resume and job description influence

- **Job description** drives role-specific questions, match score, skill gaps, and resume tailoring.
- **Resume / self-description** provides candidate background for personalized output.

### Real-time / live AI

Not implemented. All Gemini calls are synchronous request/response from Express controllers.

### Error handling and retries

No retry logic, fallback models, or dedicated AI error middleware. Failures bubble up unhandled from controllers.

---

## Database Design

```text
User (users)
  │
  └── InterviewReport
        ├── resume
        ├── selfDescription
        ├── jobDescription
        ├── matchScore
        ├── title
        ├── technicalQuestions[]
        ├── behavioralQuestions[]
        ├── skillGaps[]
        └── preparationPlan[]

BlacklistToken (blacklistTokens)
  └── token
```

### `users`

| Field | Notes |
| --- | --- |
| `username` | Unique, required |
| `email` | Unique, required |
| `password` | Bcrypt hash, required |

### `InterviewReport`

| Field | Notes |
| --- | --- |
| `user` | ObjectId ref to `users` |
| `matchScore` | Number 0–100 |
| `title` | Inferred job title |
| Nested arrays | Questions, skill gaps, daily plan |
| Timestamps | `createdAt`, `updatedAt` |

### `blacklistTokens`

Stores JWT strings invalidated on logout.

---

## Environment Variables

Create `Backend/.env`. **Never commit real values.**

### Backend

```env
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret
GOOGLE_GENAI_API_KEY=your_google_genai_api_key
```

| Variable | Required | Used in |
| --- | --- | --- |
| `MONGO_URI` | Yes | `Backend/src/config/database.js` |
| `JWT_SECRET` | Yes | Auth controller and middleware |
| `GOOGLE_GENAI_API_KEY` | Yes | `Backend/src/services/ai.service.js` |

The server listens on port **3000** (hardcoded in `Backend/server.js`). There is no `PORT` environment variable in the current code.

### Frontend

No `VITE_*` or other frontend env vars. API URL is hardcoded to `http://localhost:3000` in:

- `Frontend/src/features/auth/services/auth.api.js`
- `Frontend/src/features/interview/services/interview.api.js`

---

## Installation & Local Setup

### Prerequisites

- Node.js (LTS recommended)
- npm
- MongoDB (local or MongoDB Atlas)
- Google Gemini API key

### Clone

```bash
git clone <repository-url>
cd RoleReady-ai-yt-main
```

### Backend setup

```bash
cd Backend
npm install
```

Create `Backend/.env` with the variables above, then:

```bash
npm run dev
```

Runs on **http://localhost:3000** using `npx nodemon server.js`.

### Frontend setup

```bash
cd Frontend
npm install
npm run dev
```

Runs on **http://localhost:5173** (Vite default).

### Production build (frontend)

```bash
cd Frontend
npm run build
npm run preview
```

---

## CORS / Local Development Configuration

**Backend** (`Backend/src/app.js`):

```javascript
cors({ origin: "http://localhost:5173", credentials: true })
```

**Frontend** (Axios):

```javascript
axios.create({ baseURL: "http://localhost:3000", withCredentials: true })
```

Both sides must align for cookie auth to work. If the Vite dev port changes, update the backend CORS origin accordingly.

---

## Usage

1. Start MongoDB and the backend (`npm run dev` in `Backend/`).
2. Start the frontend (`npm run dev` in `Frontend/`).
3. Open `http://localhost:5173`.
4. Register or log in.
5. Paste a job description on the home page.
6. Upload a resume PDF and/or enter a self-description.
7. Click **Generate My Interview Strategy**.
8. Review technical questions, behavioral questions, match score, skill gaps, and the preparation roadmap.
9. Optionally download a tailored resume PDF.
10. Return home to browse **My Recent Interview Plans**.

---

## Error Handling

### Backend

| Area | Behavior |
| --- | --- |
| Auth validation | `400` with `{ message }` |
| Missing/invalid token | `401` with `{ message }` |
| Missing report | `404` with `{ message }` |
| DB connection | Logged in `connectToDB()`; server still starts |
| AI / PDF errors | No dedicated try/catch in controllers |
| Global handler | Not implemented |

### Frontend

| Area | Behavior |
| --- | --- |
| Auth/interview errors | Logged to `console` |
| User-facing errors | Not shown on login/register forms |
| Loading states | Shown during session restore and report generation |

---

## Troubleshooting

### `vite` is not recognized

```bash
cd Frontend
npm install
npm run dev
```

### MongoDB connection problems

- Verify `MONGO_URI` in `Backend/.env`.
- For Atlas, allow your IP in Network Access.
- `Backend/server.js` sets DNS to `8.8.8.8` and `1.1.1.1` to help resolve Atlas hostnames.

### CORS errors

- Frontend must run on `http://localhost:5173` (or update backend CORS).
- Axios must use `withCredentials: true`.

### Authentication cookie not sent

- Backend CORS must allow credentials.
- Frontend and backend must use matching origin/port configuration.

### Resume upload failures

- Backend limit: **3 MB** (UI mentions 5 MB).
- Only PDF text extraction is implemented.
- Missing file upload may crash report generation.

### Gemini API errors

- Verify `GOOGLE_GENAI_API_KEY`.
- Confirm model access for `gemini-3-flash-preview`.
- Rate limits (`429`) and temporary unavailability (`503`) are not retried automatically.

### Puppeteer PDF issues

- Ensure Chromium dependencies are available on your OS.
- Headless launch failures block resume PDF download.

---

## Security Considerations

### Implemented

- Bcrypt password hashing
- JWT authentication with blacklist on logout
- Report fetch by ID scoped to `req.user.id`
- Secrets via environment variables (`.env` gitignored in `Backend/`)

### Limitations (development considerations)

- JWT cookies lack explicit `httpOnly` / `secure` / `sameSite`
- Resume PDF endpoint does not verify report ownership
- No rate limiting or input sanitization library
- No automated security or integration tests

---

## Future Improvements

Suggested enhancements — **not currently implemented**:

- Live mock interviews with answer submission and AI evaluation
- Voice/audio interview practice
- Gemini retry/fallback handling for `429` / `503` errors
- DOCX resume parsing to match UI
- Validation when only self-description is provided (no file)
- Logout button and user-facing error messages
- Interview analytics and search/filter for history
- Production deployment (Docker, env-based CORS, HTTPS cookies)
- Automated tests and OpenAPI documentation
- Accessibility and performance optimizations

---

## Challenges / Engineering Highlights

- **Structured AI output** — Zod + Gemini JSON schema produces predictable, storable report objects.
- **Context-aware preparation** — Resume, self-description, and JD combined for role-specific output.
- **Dual AI pipelines** — Analytical report generation and presentational resume HTML/PDF export.
- **Cookie auth in a SPA** — JWT in cookies with Axios credentials and server-side blacklist logout.
- **Feature-based React architecture** — Separate auth/interview contexts, hooks, services, and pages.
- **PDF pipeline** — Resume text extraction (`pdf-parse`) and headless rendering (`Puppeteer`).

---

## Contributing

```text
Fork → Create branch → Make changes → Commit → Push → Pull Request
```

Keep changes focused and do not commit secrets or `.env` files.

---

## License

No project-level license file is specified in this repository. The backend `package.json` lists `"license": "ISC"`, but there is no root `LICENSE` file.

---

## Author

No author metadata is defined in repository configuration. The backend `package.json` `"author"` field is empty.
