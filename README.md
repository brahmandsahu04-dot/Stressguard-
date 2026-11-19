# StressGuard – Web-Based Intelligent Stress Monitoring and Wellness Tracking System

Mini project (BCC-351), B.Tech IT, AKGEC Ghaziabad. Implements the system described in the project report:
**React.js** frontend · **Java Spring Boot** REST API · **PostgreSQL** database.

> StressGuard is a self-reported wellness indicator. It is **not** a medical diagnosis.

```
stressguard/
├── backend/            Spring Boot 3 (Java 17+) REST API
├── frontend/           React 18 + Vite single-page app
├── docs/schema.sql     PostgreSQL schema (also auto-created by the backend)
├── docs/screenshots/   Sample UI screenshots
└── docker-compose.yml  PostgreSQL for local development
```

## 1. Run it

Requirements: JDK 17+, Maven 3.9+, Node 18+ (Docker optional, for PostgreSQL).

### Quick demo (no database to install)
```bash
# terminal 1 – backend with an in-memory database
cd backend
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# terminal 2 – frontend
cd frontend
npm install
npm run dev          # open http://localhost:5173
```
The `dev` profile creates a demo administrator: **admin@stressguard.local / Admin@12345**.
Register any other account from the sign-up page.

### With PostgreSQL (as in the report)
```bash
docker compose up -d            # starts PostgreSQL on :5432
cd backend
export JWT_SECRET="$(openssl rand -base64 48)"        # required, 32+ characters
export ADMIN_EMAIL=admin@example.com ADMIN_PASSWORD='ChangeMe123'   # optional first admin
mvn spring-boot:run
```
Database settings come from `DB_URL`, `DB_USER`, `DB_PASSWORD` (defaults match `docker-compose.yml`).

### Tests
```bash
cd backend && mvn test
```
`StressScoringServiceTest` covers the scoring rules; `StressGuardApiTest` runs the report's test cases
**TC-01 … TC-09** plus lock-out, admin-only access, reminders and export against the real Spring context.
Record the pass/fail results in Table 7.1 of your report (replace “To be tested”).
TC-10 (responsive layout) is a manual check: open the site in a phone-sized browser window.

## 2. How the report maps to the code

| Report | Where |
|---|---|
| FR-1/FR-2 registration, login, logout, password reset | `AuthService`, `AuthController`, `Login/Register/ForgotPassword.jsx` |
| FR-3 consent | `ConsentService`, `Privacy.jsx`, consent gate in `CheckIn.jsx` and `AssessmentService.submit` |
| FR-4 questionnaire | `DataSeeder` (10 items, 5 categories), `QuestionController`, `CheckIn.jsx` |
| FR-5 transparent scoring (5.1) | `StressScoringService` |
| FR-6/FR-7 dashboard, history, trends | `AssessmentService.trends`, `Dashboard.jsx`, `History.jsx`, `TrendChart.jsx` |
| FR-8 wellness suggestions | `ResourceService`, `Wellness.jsx`, `BreathingGuide.jsx` |
| FR-9 optional reminders (off by default) | `ReminderService`, dashboard banner |
| FR-10 export / deletion | `PrivacyService`, `Privacy.jsx` |
| FR-11 admin (no access to private answers) | `AdminService`, `AdminController`, `Admin.jsx` |
| 5.2 controller, 5.3 validation | `AssessmentController`, `Dtos.AssessmentRequest` |
| 5.4 React question component | `StressQuestion` in `CheckIn.jsx` |
| 5.5 security list | below |
| Chapter 6 tables | `domain/*.java`, `docs/schema.sql` |
| 7.3 safety / crisis information | optional safety question + `SupportContacts` |

### Security measures implemented (Section 5.5)
BCrypt password hashing · stateless JWT in the `Authorization` header · server-side authorisation and
ownership checks (user id is read from the token, never from the request) · JPA parameterised queries ·
Bean Validation on every request · login lock-out after 5 failures (15 min) and generic login errors ·
no answers/passwords/tokens in logs · audit events with no sensitive content · CORS limited to the frontend origin ·
`ON DELETE CASCADE` so account deletion removes all answers. Serve the app over **HTTPS** in any real deployment.

## 3. Things to know before you submit

* **Scoring thresholds** (≤33 % low, ≤66 % moderate, above high; i.e. 0-13 / 14-26 / 27-40) are *illustrative*,
  exactly as the report states. The 10 question texts are original wording, not the PSS. If you switch to the PSS you
  must follow its scoring and licence.
* **Helplines** (Tele-MANAS 14416, KIRAN 1800-599-0019, 112) are in `AssessmentService.SUPPORT_CONTACTS` – please
  verify they are still correct before demonstrating.
* **Small differences from the report:** `assessments.max_score` column added (keeps history correct if the number
  of questions changes); `users` also has `role`, lock-out and reset-token columns; the safety question is never stored;
  password reset prints the token in the backend console in dev mode because there is no mail server; e-mail
  verification is not implemented (the `PENDING_VERIFICATION` status is reserved for it – listed as future scope).
* **Screenshots in `docs/screenshots/`** were captured from the React app running against a stand-in API, to show the
  layout. For Chapter 8 of your report, retake them from your own run of the real backend (Section 8 of the report
  calls its figures “mock-ups”; once you have the running app you can say they are screenshots).
