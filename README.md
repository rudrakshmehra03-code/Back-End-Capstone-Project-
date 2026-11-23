# University Campus Management System – Backend

A REST + real-time backend for running university academic workflows. Administrators manage users, courses, sections and enrolments. Faculty mark and correct attendance and publish announcements. Students see their courses, their attendance and announcements.

**Stack:** Node.js · Express 5 · MongoDB · Mongoose 9 · JWT (access + refresh) · bcrypt · Socket.IO · EventEmitter · express-validator

Capstone Project 1 (Backend Development).

---

## Features

| Module | What it does |
|---|---|
| **Authentication** | Student self-registration, login, refresh-token rotation, logout that revokes every token. bcrypt password hashing. |
| **RBAC** | Three roles: `admin`, `faculty`, `student`. Role checks plus data-ownership checks (e.g. a faculty member can only act on sections assigned to them). |
| **User profiles** | View your own profile. Update only the permitted fields (name, phone). Change your password. Admins can activate or deactivate accounts. |
| **Students / Faculty / Courses / Sections** | Full CRUD with pagination, filtering, sorting and search. Admins assign faculty to sections. |
| **Enrolments** | Enrol a student in a course through a section. Duplicate, invalid and over-capacity enrolments are blocked. |
| **Attendance** | Bulk marking per class session (section + date + period). Duplicates are blocked. Corrections need a reason, are time-limited for faculty and are recorded in an audit trail. Students see a subject-wise percentage. |
| **Announcements** | University, course or section scope. Faculty can post only to their own course or section. History is available over REST and new posts are pushed live. |
| **Reports** | Four aggregation pipelines: attendance summary, low-attendance list (configurable threshold, CSV export), course enrolment counts, section summary. |
| **Real-time** | Socket.IO, authenticated with the same JWT. Users are placed in rooms by role, course and section. Events: announcements, attendance marked or corrected, low-attendance alerts, new enrolments. |
| **Events** | A Node.js `EventEmitter` bus separates controllers from the notifiers (Socket.IO and the event log). |

## Prerequisites

- Node.js 18 or newer (developed on Node 24)
- MongoDB 6 or newer running locally (or a MongoDB Atlas connection string)
- Postman (optional; the collection can also run from the CLI with Newman)

## Setup

```bash
git clone <repo-url> campus-management-system
cd campus-management-system
npm install
cp .env.example .env        # then edit the secrets
npm run seed                # creates demo data (drops the campus_management database first)
npm run dev                 # or: npm start
```

The server runs on `http://localhost:5000`. Check `GET /api/health`.

### Environment variables

| Variable | Default | Description |
|---|---|---|
| `NODE_ENV` | `development` | `production` hides stack traces |
| `PORT` | `5000` | HTTP port |
| `MONGODB_URI` | – (required) | MongoDB connection string |
| `JWT_ACCESS_SECRET` | – (required) | Secret for access tokens |
| `JWT_REFRESH_SECRET` | – (required) | Secret for refresh tokens (must be different) |
| `JWT_ACCESS_EXPIRES_IN` | `15m` | Access token lifetime |
| `JWT_REFRESH_EXPIRES_IN` | `7d` | Refresh token lifetime |
| `BCRYPT_SALT_ROUNDS` | `10` | bcrypt cost factor |
| `CORS_ORIGIN` | `*` | Allowed CORS origin |
| `ATTENDANCE_THRESHOLD` | `75` | Low-attendance threshold (%) |
| `ATTENDANCE_EDIT_WINDOW_DAYS` | `7` | Days within which faculty may correct attendance |

Generate a secret with: `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`

`.env` is listed in `.gitignore`. Never commit it.

## Demo accounts (after `npm run seed`)

Every account uses the password `Password@123`.

| Role | Email | Notes |
|---|---|---|
| Admin | `admin@campus.edu` | |
| Faculty | `ravi.kumar@campus.edu` | Teaches DBMS (CS201) sections A and B |
| Faculty | `meera.iyer@campus.edu` | Teaches Computer Networks (CS202) A |
| Faculty | `arjun.singh@campus.edu` | Teaches Discrete Maths (MA201) A |
| Student | `student001@campus.edu` … `student012@campus.edu` | `student003` and `student009` have low attendance |

The seed creates 4 courses, 5 sections (one with no faculty assigned), 35 enrolments, about 350 attendance records over the last two weeks, 1 audit entry and 3 announcements.

## npm scripts

| Script | Purpose |
|---|---|
| `npm start` | Start the server |
| `npm run dev` | Start with nodemon (auto-restart) |
| `npm run seed` | Reset and seed the database |
| `npm run test:api` | Run the Postman collection with Newman (the server must be running; seed first) |
| `npm run socket:demo -- <email> <password>` | CLI client that prints every real-time event it receives |

## Testing

1. `npm run seed`, then `npm start`
2. Import `postman/Campus-Management-System.postman_collection.json` and `postman/local.postman_environment.json` into Postman and run the collection in order. Or run `npm run test:api`.

The collection has **105 requests and 255 assertions** in 12 folders. It covers successful requests, validation failures (400), missing or invalid tokens (401), wrong roles (403), missing resources (404), duplicates or conflicts (409), and business rules (future dates, faculty acting on another faculty's section, students reading other students' data, token reuse after logout, and more).

The collection is generated by `scripts/generate-postman.js`. Edit that file and re-run it rather than editing the JSON by hand.

## Real-time demo

1. Open `http://localhost:5000/socket-demo.html` in the browser and connect as `student003@campus.edu`.
2. In Postman, log in as `ravi.kumar@campus.edu` and mark `student003` absent in DBMS section A (`POST /api/attendance`), or publish a course announcement.
3. The page shows `attendance:marked`, `attendance:low` and `announcement:new` as they arrive.

The CLI alternative is `npm run socket:demo -- student003@campus.edu Password@123`.

## Project structure

```
src/
  server.js            HTTP server, Socket.IO, DB connection, graceful shutdown (process signals)
  app.js               Express app and global middleware chain
  config/              env (dotenv), db connection, constants/enums
  models/              Mongoose schemas (10 collections)
  routes/index.js      All REST routes: Route -> auth -> RBAC -> validation -> controller
  controllers/         Request handling for each resource
  services/            Business logic shared by controllers (auth, access rules, attendance maths)
  middleware/          authenticate/authorize, validate, request logger, central error handler
  validators/          express-validator rule sets
  events/              EventEmitter bus and its listeners
  socket/              Socket.IO auth, rooms
  utils/               ApiError, response helper, pagination, CSV, dates, tokens
scripts/               seed.js, socket-client.js, generate-postman.js
postman/               API test collection and environment
public/                socket-demo.html (real-time demo page)
docs/                  API reference, database design, Socket.IO events, project report
```

## Documentation

- [docs/API.md](docs/API.md): every endpoint with method, auth, parameters, sample response and errors
- [docs/DATABASE.md](docs/DATABASE.md): collections, schemas, relationships, indexes, aggregation pipelines
- [docs/SOCKET_EVENTS.md](docs/SOCKET_EVENTS.md): Socket.IO authentication, rooms, event names and payloads
- [docs/PROJECT_REPORT.md](docs/PROJECT_REPORT.md): project report following the suggested structure
