# School Client Application

Parent and student portal for school self-service access.

## Contents

- `frontend/` React client interface
- `backend/` Node.js + Express API
- `.env.example` repository-level environment template (copy relevant values into `backend/.env`)

## Quick Start

### 1) Backend

```bash
cd backend
cp .env.example .env
npm install
npm run dev
```

Backend runs on `http://localhost:5001` by default.

### 2) Frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend runs on `http://localhost:3000` by default.

## Required Environment Variables

This repository includes two templates:

- `./.env.example` (submission-level template)
- `./backend/.env.example` (runtime backend template)

For actual execution, use `backend/.env`.

## Main Features

- Parent/student registration and login with device verification flow
- JWT access tokens (15 min) + refresh token rotation for session continuity
- Inactivity timeout: sessions expire after 30 minutes of no user interaction (configurable via `SESSION_IDLE_TIMEOUT_MINUTES`)
- Fee balance, payment submission, and refund requests
- Grades, attendance, and timetable viewing
- Low-balance alerts and account linking flow
- Swagger API docs at `/api-docs`

## Shared Database

Both the client app and the admin app connect to the **same MongoDB instance and the same `users` collection**. The `ClientUser` model in the admin backend and the `User` model in the client backend both reference the `users` collection. This is intentional — it allows the admin to verify devices and manage accounts that were created through the client registration flow.

Make sure both backends point to the same `MONGO_URI` value in their respective `.env` files.

## Push Notifications

This application is a **web client** (React.js), not a mobile app. Push notifications (as described in the spec for React Native) are therefore not implemented. In-app alerts (low balance warnings, payment status) are shown directly in the dashboard UI instead. A mobile implementation using React Native or Flutter would add native push support via Firebase Cloud Messaging (FCM).

## Testing

```bash
cd backend
npm test                  # Run all tests
npm test -- --coverage    # With coverage report
```

Coverage is collected from `src/services/` and `src/dtos/` — the testable business logic layer. Routes and controllers require a live server and are not included in unit test coverage.
