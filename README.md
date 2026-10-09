# LearnPath AI — Adaptive Learning Dependency Graph

A full-stack hackathon MVP for **PS-30**. LearnPath AI estimates mastery for individual learning concepts, tracks prerequisite dependencies, and recommends the next eligible lesson rather than relying on one overall quiz score.

## MVP features

- Java Programming sample course with five concepts: Variables, Operators, Conditions, Loops, and Arrays.
- Fifteen seeded diagnostic questions, tagged to concepts.
- Per-concept mastery calculation from recorded attempts.
- Prerequisite dependency graph rendered with React Flow.
- Rule-based lesson recommendation that respects prerequisite readiness.
- Lesson explanations and Java examples.
- Quiz retesting that recalculates mastery and updates the path.
- Dashboard progress cards, concept statuses, and mastery bars.
- REST API built with Spring Boot, Spring Data JPA, and MySQL.

> This is a hackathon prototype. It does not implement production authentication, password storage, multi-tenant authorization, or privacy/security hardening. The initial mastery score is a transparent estimate: cumulative correct attempts divided by cumulative attempts. The 50% and 80% thresholds are configurable demo rules, not universal educational standards.

## Technology stack

- Frontend: React, Vite, Axios, `@xyflow/react`
- Backend: Java 17+, Spring Boot, Spring Web, Spring Data JPA, Bean Validation
- Database: MySQL 8+

## Project structure

```text
LearnPathAI/
├── frontend/              # React user interface
├── backend/               # Spring Boot REST API
├── database/setup.sql     # Create the database
├── docs/                  # Architecture and API notes
└── README.md
```

## Run locally on Windows

### 1. Create the database

Make sure the MySQL service is running. Open MySQL Workbench or the MySQL command line and run:

```sql
CREATE DATABASE IF NOT EXISTS learnpathai;
```

The backend connects to `localhost:3306`, database `learnpathai`, username `root` by default. Set environment variables if your MySQL credentials differ.

PowerShell example:

```powershell
$env:DB_USERNAME = "root"
$env:DB_PASSWORD = "your_mysql_password"
```

If your local MySQL account has no password, you can leave `DB_PASSWORD` unset. Never commit a real password to source control.

### 2. Start the backend

Open a terminal in `LearnPathAI\backend`:

```powershell
mvn spring-boot:run
```

Use Java 17 or newer and Maven 3.9+. Spring Boot creates/updates tables when the application starts. The first launch seeds a demo student, a Java subject, concepts, dependencies, lessons, and questions.

Backend base URL: `http://localhost:8080/api`

### 3. Start the frontend

Open a **second** terminal in `LearnPathAI\frontend`:

```powershell
Copy-Item .env.example .env
npm install
npm run dev
```

Open the URL printed by Vite, usually `http://localhost:5173`.

The frontend `.env` should contain:

```env
VITE_API_BASE_URL=http://localhost:8080/api
```

Restart Vite after editing `.env`.

## First-time demo flow

1. Open Dashboard. You should see the Java concept dependency graph and the recommendation to begin with Variables.
2. Select **Diagnostic quiz** and answer the fifteen questions.
3. Submit. LearnPath AI saves each answer and calculates mastery per concept.
4. Return to Dashboard. Review concept scores and the recommended next lesson.
5. Open the recommended lesson, read its explanation/example, then choose **Practice and retest**.
6. Submit another attempt. The dashboard and dependency graph refresh from the updated mastery data.

With cumulative scoring, a student's mastery is `correct attempts / total attempts × 100`. Repeating a question adds another attempt, so improvement happens through subsequent accurate answers rather than overwriting history.

## REST API

All routes are under `/api`.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/subjects` | List subjects |
| GET | `/students` | List demo students |
| GET | `/concepts?subjectId=1` | List subject concepts |
| GET | `/quiz/diagnostic?subjectId=1` | Get diagnostic questions (without answers) |
| POST | `/quiz/submit` | Submit quiz answers |
| POST | `/quiz/retest` | Submit a retest (same scoring path) |
| GET | `/students/1/mastery?subjectId=1` | Mastery by concept |
| GET | `/students/1/progress?subjectId=1` | Dashboard progress metrics |
| GET | `/students/1/recommendations?subjectId=1` | Next recommended learning action |
| GET | `/subjects/1/graph?studentId=1` | Graph nodes and prerequisite edges |
| GET | `/lessons/{lessonId}` | Read a lesson |

Example `POST /api/quiz/submit` body:

```json
{
  "studentId": 1,
  "subjectId": 1,
  "answers": [
    { "questionId": 1, "selectedOption": "A" },
    { "questionId": 2, "selectedOption": "B" }
  ]
}
```

The diagnostic question endpoint never sends the correct answer to the browser. The backend evaluates answers and returns total score plus concept-level mastery results.

## Recommendation logic

1. Load mastery scores and dependency edges for the selected subject.
2. Walk concepts in configured learning/display order.
3. Skip concepts at or above 80% mastery.
4. Do not recommend a concept until all its direct prerequisites have at least one attempt and at least 80% mastery.
5. Recommend the first unfinished, eligible concept: `START` for a new concept, `REVISE` below 50%, or `PRACTICE` from 50% to below 80%.
6. If all concepts meet the threshold, mark the learning path complete.

This is intentionally explainable. For a production release, consider confidence intervals, recency-weighted mastery, question difficulty, learning-objective validation, teacher review of prerequisite edges, and accessibility testing.

## Troubleshooting

- **Cannot connect to MySQL:** check the `MySQL80` Windows service, username/password, and port 3306. You can override `DB_URL`, `DB_USERNAME`, and `DB_PASSWORD`.
- **Unknown database:** run `CREATE DATABASE IF NOT EXISTS learnpathai;` in MySQL Workbench.
- **Port already in use:** set `SERVER_PORT` for Spring Boot, or update the frontend API URL accordingly.
- **Frontend says Network Error:** start the backend first and verify `http://localhost:8080/api/subjects` in a browser.
- **Tables exist but there is no sample content:** the seed runner only creates the initial concepts when the `concepts` table is empty. For a clean demo database, stop the app, drop/recreate `learnpathai`, and restart the backend. This deletes local demo data.

## Suggested next improvements

- Add proper registration/login and role-based authorization.
- Add instructor screens to edit the curriculum graph and question bank.
- Separate question difficulty and add more questions per concept.
- Store a learning-path recommendation history to explain changes over time.
- Add automated tests for scoring, prerequisite selection, and blocked paths.
- Add more subjects only after the Java workflow is stable.
