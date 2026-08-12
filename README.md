# RoleReady AI

**AI-powered interview preparation for your next role.**

Interview AI is a full-stack web application that helps job seekers prepare for interviews by analyzing a target job description together with their resume or self-description. Google Gemini generates a personalized interview strategy—including likely questions, skill gap analysis, a day-by-day preparation plan, and an optional tailored resume PDF.

![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-7-646CFF?logo=vite&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-Express_5-339933?logo=nodedotjs&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose-47A248?logo=mongodb&logoColor=white)
![Gemini](https://img.shields.io/badge/Google-Gemini-4285F4?logo=google&logoColor=white)

---

## Overview

Interview AI is built for candidates who want structured, role-specific interview preparation instead of generic question lists. After signing in, a user provides a job description and either uploads a resume or writes a short self-description. The backend sends that context to **Google Gemini**, which returns a structured interview report. The report is saved in **MongoDB** and can be reviewed anytime from the dashboard-style home page.

The application focuses on **interview planning and preparation**, not live interviewing. There is no real-time Q&A session, voice interaction, or automated scoring of spoken or typed answers during an interview. Instead, Gemini produces:

- A job match score
- Technical and behavioral questions with interviewer intent and model answers
- Skill gaps with severity levels
- A multi-day preparation roadmap
- An AI-tailored resume PDF downloadable from the report page

---

## Key Features

### User authentication

Users can register with a username, email, and password, log in, and access protected pages. Sessions are managed with JWT tokens stored in cookies. Logout invalidates the token by adding it to a server-side blacklist.

### Protected routes

The home page and interview report pages require authentication. Unauthenticated users are redirected to the login page.

### Job description input

Users paste a target job description (up to 5,000 characters in the UI) that drives question generation and match scoring.

### Resume upload and self-description

Users can upload a resume file or provide a quick self-description. PDF resumes are processed server-side using `pdf-parse`, while the self-description option provides additional candidate context for personalized preparation.

### AI-generated interview reports

Gemini analyzes resume text, self-description, and job description to produce a structured report with:

- **Match score** (0–100)
- **Technical questions** with intention and suggested answer approach
- **Behavioral questions** with intention and suggested answer approach
- **Skill gaps** tagged as low, medium, or high severity
- **Preparation plan** organized by day with focus areas and tasks
- **Job title** inferred for the role

### Interview history

Authenticated users can view a list of previously generated reports on the home page, showing title, creation date, and match score. Full question lists are excluded from the list API response for lighter payloads.

### Detailed report viewer

Each report opens on a dedicated page with tabbed navigation for technical questions, behavioral questions, and the preparation roadmap. Expandable cards reveal interviewer intent and model answers.

### AI-generated resume PDF

From a report page, users can download a tailored resume PDF. Gemini generates ATS-friendly HTML content, which the backend converts to PDF using Puppeteer.

### Responsive frontend UI

The React frontend uses SCSS with a card-based layout, sidebar navigation on report pages, and styled form screens for login and registration.

---

## How It Works

```text
User
  ↓
Register / Login (JWT cookie)
  ↓
Home — enter Job Description + Resume or Self Description
  ↓
POST /api/interview/ — PDF parsed, context sent to Gemini
  ↓
Structured AI report saved to MongoDB
  ↓
Interview Report Page — questions, skill gaps, roadmap
  ↓
Optional — download AI-tailored resume PDF
```

### Lifecycle from the user's perspective

1. Create an account or log in.
2. On the home page, paste the target job description.
3. Upload a resume PDF or enter a self-description (the UI recommends providing at least one).
4. Click **Generate My Interview Strategy** and wait while Gemini processes the request.
5. Review the generated report: match score, skill gaps, technical/behavioral questions, and preparation roadmap.
6. Optionally download a tailored resume PDF from the report page.
7. Return to the home page to view past reports and generate new ones.

---

## Screenshots / Demo

Screenshots are not included in the repository yet. Add them here when available.

<!-- Add home page screenshot here -->

<!-- Add interview report screenshot here -->

<!-- Add preparation roadmap screenshot here -->

---

## Tech Stack

### Frontend

| Technology | Purpose |
| --- | --- |
| **React 19** | UI components and page rendering |
| **Vite 7** | Development server and production bundling |
| **React Router 7** | Client-side routing (`/login`, `/register`, `/`, `/interview/:id`) |
| **Axios** | HTTP requests to the backend with credentials |
| **SCSS / Sass** | Styling for forms, home page, and interview report layout |

### Backend

| Technology | Purpose |
| --- | --- |
| **Node.js** | Runtime environment |
| **Express 5** | REST API and middleware pipeline |
| **Multer** | In-memory resume file upload (3 MB limit) |
| **pdf-parse** | Extract text from uploaded PDF resumes |
| **Puppeteer** | Convert AI-generated HTML resumes into downloadable PDFs |
| **cookie-parser** | Read JWT cookies on protected requests |
| **CORS** | Allow frontend origin with credentials |

### Database

| Technology | Purpose |
| --- | --- |
| **MongoDB** | Persistent storage for users, reports, and blacklisted tokens |
| **Mongoose** | Schema definitions and model queries |

### AI

| Technology | Purpose |
| --- | --- |
| **@google/genai** | Google GenAI SDK client |
| **gemini-3-flash-preview** | Model used for interview reports and resume HTML generation |
| **Zod** | Schema definition for expected AI output shape |
| **zod-to-json-schema** | Converts Zod schemas into Gemini `responseSchema` constraints |

### Authentication

| Technology | Purpose |
| --- | --- |
| **jsonwebtoken** | Sign and verify JWTs (1-day expiry) |
| **bcryptjs** | Password hashing on registration and comparison on login |
| **HTTP cookies** | Store JWT after login/register; cleared on logout |

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

1. The frontend sends authenticated requests with cookies to the Express API.
2. Protected interview routes verify the JWT and check the token blacklist.
3. Resume uploads are parsed server-side; text context is combined with job and self descriptions.
4. The AI service calls Gemini with JSON schema constraints and parses the structured response.
5. Reports are persisted in MongoDB and returned to the frontend.
6. Resume PDF generation reuses stored report context, asks Gemini for HTML, then renders PDF via Puppeteer.

---

## Project Structure

```text
RoleReady-ai-main/
├── Backend/
│   ├── server.js                 # Entry point, DB connect, listens on port 3000
│   ├── package.json
│   └── src/
│       ├── app.js                # Express app, middleware, route mounting
│       ├── config/
│       │   └── database.js       # MongoDB connection
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
│           └── ai.service.js     # Gemini + Puppeteer integration
│
├── Frontend/
│   ├── index.html
│   ├── vite.config.js
│   ├── package.json
│   └── src/
│       ├── main.jsx
│       ├── App.jsx
│       ├── app.routes.jsx
│       ├── style.scss
│       └── features/
│           ├── auth/
│           │   ├── auth.context.jsx
│           │   ├── hooks/useAuth.js
│           │   ├── services/auth.api.js
│           │   ├── components/Protected.jsx
│           │   └── pages/Login.jsx, Register.jsx
│           └── interview/
│               ├── interview.context.jsx
│               ├── hooks/useInterview.js
│               ├── services/interview.api.js
│               └── pages/Home.jsx, Interview.jsx
│
└── README.md
```

| Directory | Purpose |
| --- | --- |
| `Backend/src/controllers` | Request handlers for auth and interview operations |
| `Backend/src/services` | Gemini prompts, schema validation, and PDF generation |
| `Backend/src/models` | Mongoose schemas for users, reports, and token blacklist |
| `Frontend/src/features/auth` | Login/register flow and route protection |
| `Frontend/src/features/interview` | Report creation, listing, viewing, and PDF download |

---

## API Documentation

Base URL (local development): `http://localhost:3000`

### Authentication APIs

| Method | Endpoint | Authentication | Description |
| --- | --- | --- | --- |
| `POST` | `/api/auth/register` | No | Register a new user |
| `POST` | `/api/auth/login` | No | Authenticate an existing user |
| `GET` | `/api/auth/logout` | No* | Log out and blacklist current token |
| `GET` | `/api/auth/get-me` | Yes | Get the logged-in user's profile |

\* Logout does not require the auth middleware, but it reads the cookie if present.

#### `POST /api/auth/register`

**Body (JSON):**

```json
{
  "username": "jane_doe",
  "email": "jane@example.com",
  "password": "your_password"
}
```

**Success (`201`):** Sets a JWT cookie and returns user metadata.

**Errors:** `400` if fields are missing or username/email already exists.

#### `POST /api/auth/login`

**Body (JSON):**

```json
{
  "email": "jane@example.com",
  "password": "your_password"
}
```

**Success (`200`):** Sets a JWT cookie and returns user metadata.

**Errors:** `400` for invalid credentials.

#### `GET /api/auth/get-me`

**Success (`200`):**

```json
{
  "message": "User details fetched successfully",
  "user": {
    "id": "...",
    "username": "jane_doe",
    "email": "jane@example.com"
  }
}
```

**Errors:** `401` if token is missing, invalid, or blacklisted.

---

### Interview APIs

| Method | Endpoint | Authentication | Description |
| --- | --- | --- | --- |
| `POST` | `/api/interview/` | Yes | Generate a new interview report |
| `GET` | `/api/interview/` | Yes | List the user's interview reports |
| `GET` | `/api/interview/report/:interviewId` | Yes | Fetch a single report by ID |
| `POST` | `/api/interview/resume/pdf/:interviewReportId` | Yes | Generate and download a tailored resume PDF |

#### `POST /api/interview/`

**Body (`multipart/form-data`):**

| Field | Type | Description |
| --- | --- | --- |
| `resume` | File | Resume PDF uploaded as `resume` |
| `jobDescription` | String | Target job description |
| `selfDescription` | String | Optional profile summary |

**Success (`201`):** Returns the full saved interview report.

**Notes:** The controller extracts text from uploaded PDF resumes before generating the interview report.

#### `GET /api/interview/`

**Success (`200`):** Returns a trimmed list of reports (title, match score, timestamps, etc.) without full question arrays or raw resume/JD text.

#### `GET /api/interview/report/:interviewId`

**Success (`200`):** Returns the complete report for the authenticated user.

**Errors:** `404` if the report does not exist or does not belong to the user.

#### `POST /api/interview/resume/pdf/:interviewReportId`

**Success:** Binary PDF response with `Content-Type: application/pdf`.

**Errors:** `404` if the report ID is not found.

---

## Authentication Flow

1. **Registration:** The backend validates input, checks for duplicate username/email, hashes the password with bcrypt (10 salt rounds), creates the user, signs a JWT, and sets it in a cookie.
2. **Login:** The backend finds the user by email, compares the password hash, signs a JWT, and sets the cookie.
3. **Session persistence:** On app load, the frontend calls `GET /api/auth/get-me` to restore the logged-in user from the cookie.
4. **Protected routes:** The `authUser` middleware reads `req.cookies.token`, rejects blacklisted tokens, verifies the JWT with `JWT_SECRET`, and attaches decoded user data to `req.user`.
5. **Frontend protection:** The `Protected` component redirects unauthenticated users to `/login`.
6. **Logout:** `GET /api/auth/logout` stores the token in the blacklist collection and clears the cookie.

**Implementation notes:**

- JWT expiry is set to **1 day**.
- Token blacklisting prevents reuse after logout.
- Cookies are not explicitly configured with `httpOnly`, `secure`, or `sameSite` options in the current code.

---

## AI Integration

### SDK and model

- **SDK:** `@google/genai` (`GoogleGenAI`)
- **Model:** `gemini-3-flash-preview`
- **API key env var:** `GOOGLE_GENAI_API_KEY`

### Where Gemini is called

All AI logic lives in `Backend/src/services/ai.service.js`:

1. **`generateInterviewReport`** — creates the main interview preparation report.
2. **`generateResumePdf`** — creates HTML for a tailored resume, then converts it to PDF.

### Input sent to the model

For interview reports, Gemini receives:

- Extracted resume text (if a PDF was uploaded)
- User self-description
- Job description

For resume PDF generation, Gemini receives the stored resume text, self-description, and job description from an existing report.

### Structured output handling

Both AI flows use:

- **Zod schemas** to define the expected response structure
- **`zod-to-json-schema`** to produce Gemini-compatible JSON schemas
- **`responseMimeType: "application/json"`** and **`responseSchema`** in the GenAI request config
- **`JSON.parse(response.text)`** on the model response

#### Interview report schema fields

| Field | Description |
| --- | --- |
| `matchScore` | Number from 0–100 |
| `technicalQuestions` | Array of `{ question, intention, answer }` |
| `behavioralQuestions` | Array of `{ question, intention, answer }` |
| `skillGaps` | Array of `{ skill, severity }` where severity is `low`, `medium`, or `high` |
| `preparationPlan` | Array of `{ day, focus, tasks[] }` |
| `title` | Job title for the target role |

#### Resume PDF schema fields

| Field | Description |
| --- | --- |
| `html` | Full HTML document rendered to PDF by Puppeteer |

### Resume and job description influence

- The **job description** steers question selection, match scoring, skill gap analysis, and resume tailoring.
- The **resume/self-description** provides candidate context for personalized questions and preparation tasks.
- The generated report is saved verbatim in MongoDB for later retrieval and PDF export.

### Real-time / live functionality

There is **no** real-time or streaming Gemini integration in the current codebase. Each operation is a standard request/response API call.

### Error handling and retries

AI service errors are surfaced through the application's API flow so generation failures can be handled by the client.

---

## Database Design

```text
User (users)
  │
  └── InterviewReport
        ├── resume (extracted text)
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

Linked to a user via `user` (`ObjectId` ref). Includes timestamps (`createdAt`, `updatedAt`). Nested arrays store questions, skill gaps, and daily preparation tasks.

### `blacklistTokens`

Stores invalidated JWT strings after logout.

---

## Environment Variables

Create a `.env` file inside `Backend/`. **Never commit real secrets.**

### Backend (`Backend/.env`)

```env
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret
GOOGLE_GENAI_API_KEY=your_google_genai_api_key
```

| Variable | Required | Description |
| --- | --- | --- |
| `MONGO_URI` | Yes | MongoDB connection string |
| `JWT_SECRET` | Yes | Secret used to sign and verify JWTs |
| `GOOGLE_GENAI_API_KEY` | Yes | Google Gemini API key for `@google/genai` |

### Frontend

No frontend environment variables are used. The API base URL is hardcoded to `http://localhost:3000` in the Axios service files.

---

## Installation & Local Setup

### Prerequisites

- **Node.js** (LTS recommended)
- **npm**
- **MongoDB** (local instance or MongoDB Atlas)
- **Google Gemini API key**

### Clone

```bash
git clone <repository-url>
cd RoleReady-ai-main
```

Replace `<repository-url>` with your repository URL.

### Backend setup

```bash
cd Backend
npm install
```

Create `Backend/.env` using the variables above, then start the server:

```bash
npm run dev
```

The backend runs on **http://localhost:3000**.

The dev script uses `nodemon` via:

```json
"dev": "npx nodemon server.js"
```

### Frontend setup

Open a second terminal:

```bash
cd Frontend
npm install
npm run dev
```

The frontend runs on **http://localhost:5173** by default.

### Build frontend for production

```bash
cd Frontend
npm run build
npm run preview
```

---

## CORS / Local Development Configuration

The backend enables CORS with:

- **Origin:** `http://localhost:5173`
- **Credentials:** `true`

The frontend Axios clients use:

- **baseURL:** `http://localhost:3000`
- **withCredentials:** `true`

Both settings are required for cookie-based authentication to work during local development. If you change the Vite dev server port, update the backend CORS origin in `Backend/src/app.js` accordingly.

---

## Usage

1. Start MongoDB and the backend server.
2. Start the frontend dev server.
3. Open `http://localhost:5173`.
4. Register a new account or log in.
5. Paste a job description on the home page.
6. Upload a resume PDF and/or enter a self-description.
7. Click **Generate My Interview Strategy**.
8. Explore technical questions, behavioral questions, and the preparation roadmap.
9. Review the match score and skill gaps in the sidebar.
10. Click **Download Resume** to generate a tailored PDF from the report page.
11. Return home to open previous reports from **My Recent Interview Plans**.

---

## Error Handling

### Backend

- Auth controllers return `400` for validation and credential errors, `401` for auth middleware failures, and `404` for missing reports.
- Database connection errors are logged through the database configuration.
- AI and PDF generation errors are surfaced through the API flow.

### Frontend

- Auth and interview hooks manage API requests and application state.

- Loading states are shown while auth/session restoration and report generation are in progress.

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
- For MongoDB Atlas, confirm your IP address is allowed in Network Access.
- The backend sets DNS servers to `8.8.8.8` and `1.1.1.1` in `server.js` to help resolve Atlas hostnames in some network environments.

### CORS errors in the browser

- Ensure the frontend is running on `http://localhost:5173`.
- Confirm Axios requests use `withCredentials: true`.
- Update backend CORS origin if your frontend dev port changed.

### Authentication cookie not sent

- Frontend and backend must run on different ports with CORS credentials enabled.
- Requests must include `withCredentials: true`.

### Resume upload failures

- Backend file size limit is **3 MB** (`multer` configuration).
- PDF resumes are parsed server-side using `pdf-parse`.
- For the most reliable resume analysis, upload a PDF resume within the configured file-size limit.

### Gemini API errors

- Verify `GOOGLE_GENAI_API_KEY` is valid and has access to the configured model.
- Check Google AI Studio / Cloud quota and billing status.
- Temporary Gemini service availability or rate limits can affect generation; retry the request after a short interval if needed.

### Puppeteer / PDF generation issues

- Puppeteer downloads Chromium on install and may require additional OS dependencies in some environments.
- Headless browser launch failures will prevent resume PDF download from working.

---

## Security Considerations

### Implemented

- Passwords hashed with bcrypt before storage
- JWT-based authentication with server-side token blacklist on logout
- Protected interview routes require a valid, non-blacklisted token
- Report fetch by ID checks ownership (`user: req.user.id`)
- Secrets loaded from environment variables via `dotenv`

### Recommended production practices

- `.env` files are gitignored in `Backend/`, but secrets must never be committed
- Use secure cookie settings and HTTPS when deploying to production
- Keep authentication and authorization checks enabled for protected application resources
- Add production-grade monitoring and validation as the application evolves
- Expand automated testing as the application evolves

---

## Future Improvements

These are suggested enhancements and are **not** currently implemented:

- Live or turn-based mock interview sessions with answer evaluation
- Voice/audio-based interview practice
- Expanded AI resilience and model fallback strategies
- Support for DOCX resume parsing
- Additional resume and profile input options
- Enhanced user feedback and account controls
- Interview analytics and progress tracking over time
- Deployment configuration (Docker, CI/CD, production CORS/origin setup)
- Expanded automated testing and API documentation tooling (OpenAPI/Swagger)
- Accessibility and production security enhancements

---

## Challenges / Engineering Highlights

- **Structured AI responses:** Zod schemas plus Gemini JSON schema constraints produce predictable report objects suitable for MongoDB storage and UI rendering.
- **Context-aware preparation:** Resume text, self-description, and job description are combined to generate role-specific questions and preparation plans.
- **Dual AI workflows:** One pipeline generates analytical interview strategy data; a second generates presentational HTML and exports a PDF through Puppeteer.
- **Cookie-based auth with blacklist logout:** JWT sessions integrate with SPA Axios requests while supporting token invalidation after logout.
- **Feature-based frontend architecture:** Auth and interview domains are separated into contexts, hooks, services, and pages for clearer ownership of state and API calls.

---

## Contributing

Contributions are welcome.

```text
Fork → Create branch → Make changes → Commit → Push → Pull Request
```

Please keep changes focused, match existing project conventions, and avoid committing secrets or environment files.

---

## License

No project-level license file is currently specified in this repository.

---

## Author

No author metadata is defined in the repository configuration. Update this section if you add a `LICENSE`, `package.json` author field, or repository owner information.