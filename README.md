# School Client Application

Parent and student self-service portal for viewing academic records, managing fee payments, and tracking school activity.

## Tech Stack

| Layer | Technology |
| ----- | ---------- |
| Frontend | React.js 18, Vite, React Router, TanStack React Query, Lucide React |
| Backend | Node.js 20, Express.js 4 |
| Database | MongoDB 7 + Mongoose 8 |
| Auth | JWT access tokens (15m) + refresh token rotation (7d), SHA-512 hashing |
| Security | Helmet, CORS, express-rate-limit, express-validator, cookie-parser |
| Email | Nodemailer (SMTP) |
| File Storage | Cloudinary (payment proof uploads) |
| DevOps | Docker, Docker Compose, Nginx |

---

## Project Structure

```
school-client/
├── backend/
│   ├── src/
│   │   ├── config/         # DB connection, Swagger
│   │   ├── controllers/    # HTTP request handlers
│   │   ├── dtos/           # Strip sensitive fields before response
│   │   ├── middlewares/    # auth, validate, errorHandler
│   │   ├── models/         # Mongoose schemas
│   │   ├── routes/         # Express routers
│   │   └── services/       # Business logic
│   ├── .env.example
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/     # Layout, ProtectedRoute, RefreshBar
│   │   ├── pages/          # One file per route
│   │   ├── services/       # Axios API calls
│   │   ├── utils/          # auth.js, deviceId.js
│   │   └── styles/         # global.css
│   └── package.json
├── .env.example
├── docker-compose.yml
└── README.md
```

---

## Quick Start

### Prerequisites

- Node.js >= 18
- MongoDB (same instance as admin app)
- npm >= 9

### 1. Backend

```bash
cd backend
cp .env.example .env
# Edit .env — set MONGO_URI, JWT_SECRET, JWT_REFRESH_SECRET, CLIENT_ORIGIN
npm install
npm run dev
```

API runs on `http://localhost:5001`
Swagger docs: `http://localhost:5001/api-docs`

### 2. Frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend runs on `http://localhost:3000`

### 3. Docker (optional)

```bash
cp backend/.env.example backend/.env
# Edit backend/.env as needed
docker-compose up --build
```

---

## Environment Variables

See `backend/.env.example` for the full list. Key variables:

```env
MONGO_URI=mongodb://localhost:27017/school_db
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=15m
JWT_REFRESH_SECRET=your_refresh_secret_here
JWT_REFRESH_EXPIRES_IN=7d
SESSION_IDLE_TIMEOUT_MINUTES=30
PORT=5001
NODE_ENV=development
CLIENT_ORIGIN=http://localhost:3000
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX=100
```

> `MONGO_URI` must match the admin backend — both apps share the same database.

---

## API Endpoints

| Method | Endpoint | Access | Description |
| ------ | -------- | ------ | ----------- |
| POST | /api/auth/register | Public | Register parent or student |
| POST | /api/auth/login | Public | Login with device verification |
| POST | /api/auth/refresh | Public | Refresh access token |
| POST | /api/auth/logout | Auth | Logout and revoke refresh token |
| GET | /api/auth/me | Auth | Current user profile |
| GET | /api/fees/:studentId | Auth + Device | Fee balance and history |
| POST | /api/fees/:studentId/deposit | Auth + Device | Submit fee payment |
| POST | /api/fees/:studentId/withdraw | Auth + Device | Submit refund request |
| GET | /api/academic/:studentId/profile | Auth + Device | Student profile |
| GET | /api/academic/:studentId/grades | Auth + Device | Grades by subject and term |
| GET | /api/academic/:studentId/attendance | Auth + Device | Attendance history |
| GET | /api/academic/:studentId/timetable | Auth + Device | Weekly class schedule |
| POST | /api/linking | Auth + Device | Submit linking request |
| POST | /api/auth/forgot-password | Public | Request password reset |
| POST | /api/auth/reset-password | Public | Reset password with token |

Full interactive docs available at `/api-docs` (Swagger UI).

---

## Authentication Flow

1. User registers with email, password, role, and device ID
2. Account created — device marked as **unverified**
3. Admin approves the device in the admin portal
4. User logs in — device verification checked on every request via `X-Device-ID` header
5. Access token (15m) + refresh token (7d) issued
6. Frontend auto-refreshes access token using the refresh cookie
7. After 30 minutes of inactivity, session expires and user is redirected to login

---

## Shared Database

Both the client app and the admin app connect to the **same MongoDB instance**. The `User` model in this app and the `ClientUser` model in the admin app both use the `users` collection. This allows the admin to verify devices and manage accounts created through the client registration flow.

Both backends must use the same `MONGO_URI`.

---

## Account Linking

Parents and students must link their portal account to their school record (created by admin):

- **With invite token**: Admin sends a registration invite from the admin portal. User registers using the invite link — account is automatically linked.
- **Without invite token**: User submits a linking request with their student code. Admin reviews and approves in the Linking Requests section.

---

## Key Models

| Model | Collection | Purpose |
| ----- | ---------- | ------- |
| User | users | Parent and student accounts (shared with admin) |
| Student | students | Academic profiles (shared with admin) |
| Class | classes | Class definitions and timetable (shared with admin) |
| FeeTransaction | feetransactions | Payments and refunds |
| LinkingRequest | linkingrequests | Parent-student link requests |
| PasswordReset | passwordresets | Password reset tokens (auto-deleted on expiry) |

---

## Security

- Passwords hashed with SHA-512 (per project requirements)
- JWT access tokens expire after 15 minutes
- Refresh tokens rotate on every use (7-day expiry)
- Device verification required on all protected routes
- Session idle timeout enforced on both frontend and backend
- Rate limiting: 100 requests per 15 minutes
- HTTP security headers via Helmet
- Input validation on all routes via express-validator
- Refresh tokens stored as httpOnly cookies (not accessible via JavaScript)

---

## Push Notifications

This is a web application (React.js). In-app alerts for low balance, payment confirmation, and device verification status are shown directly in the UI. A React Native or Flutter implementation would add native push notifications via Firebase Cloud Messaging (FCM).
