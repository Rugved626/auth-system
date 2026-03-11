# 🔐 User Authentication System

A full-stack Login & Registration system built with **Node.js + Express**, **PostgreSQL**, and **React**.

---

## Tech Stack

| Layer     | Tech                          |
|-----------|-------------------------------|
| Backend   | Node.js, Express              |
| Database  | PostgreSQL                    |
| Auth      | JWT (jsonwebtoken) + bcryptjs |
| Frontend  | React 18, React Router v6     |
| HTTP      | Axios                         |

---

## Project Structure

```
auth-system/
├── backend/
│   ├── src/
│   │   ├── config/db.js          # PostgreSQL connection pool
│   │   ├── middleware/auth.js     # JWT verification middleware
│   │   ├── controllers/
│   │   │   └── authController.js  # register, login, getProfile
│   │   ├── routes/auth.js         # Route definitions
│   │   └── index.js               # Express server entry
│   ├── schema.sql                 # Database schema
│   ├── .env.example               # Environment variable template
│   └── package.json
│
└── frontend/
    ├── public/index.html
    ├── src/
    │   ├── context/AuthContext.js  # Global auth state
    │   ├── components/
    │   │   └── ProtectedRoute.js   # Route guard
    │   ├── pages/
    │   │   ├── Register.js
    │   │   ├── Login.js
    │   │   └── Profile.js
    │   ├── utils/api.js            # Axios instance + interceptors
    │   ├── App.js                  # Router setup
    │   ├── App.css                 # Global styles
    │   └── index.js
    └── package.json
```

---

## Database Schema

```sql
CREATE TABLE users (
  id          SERIAL PRIMARY KEY,
  name        VARCHAR(100)  NOT NULL,
  email       VARCHAR(255)  UNIQUE NOT NULL,
  password    VARCHAR(255)  NOT NULL,
  created_at  TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at  TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

## Setup Instructions

### Prerequisites
- Node.js v18+
- PostgreSQL v14+

---

### 1. Database Setup

```bash
# Connect to PostgreSQL
psql -U postgres

# Create the database
CREATE DATABASE auth_db;
\c auth_db

# Run the schema
\i backend/schema.sql
```

---

### 2. Backend Setup

```bash
cd backend

# Install dependencies
npm install

# Create your .env file
cp .env.example .env
```

Edit `.env` with your actual values:

```env
PORT=5000
DB_HOST=localhost
DB_PORT=5432
DB_NAME=auth_db
DB_USER=postgres
DB_PASSWORD=your_postgres_password
JWT_SECRET=some_long_random_secret_string
JWT_EXPIRES_IN=7d
```

```bash
# Start the backend
npm run dev      # development (with nodemon)
npm start        # production
```

Backend will run at: `http://localhost:5000`

---

### 3. Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Start the React app
npm start
```

Frontend will run at: `http://localhost:3000`

> The `"proxy": "http://localhost:5000"` in `package.json` handles CORS in dev.

---

## API Reference

### POST `/api/register`
Register a new user.

**Request Body:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "secret123"
}
```

**Response (201):**
```json
{
  "message": "Account created successfully.",
  "token": "<JWT>",
  "user": { "id": 1, "name": "John Doe", "email": "john@example.com" }
}
```

---

### POST `/api/login`
Login with email and password.

**Request Body:**
```json
{
  "email": "john@example.com",
  "password": "secret123"
}
```

**Response (200):**
```json
{
  "message": "Login successful.",
  "token": "<JWT>",
  "user": { "id": 1, "name": "John Doe", "email": "john@example.com" }
}
```

---

### GET `/api/profile`
Get the authenticated user's profile.

**Headers:**
```
Authorization: Bearer <JWT>
```

**Response (200):**
```json
{
  "user": { "id": 1, "name": "John Doe", "email": "john@example.com", "createdAt": "..." }
}
```

---

## Approach & Design Decisions

### Security
- **Password hashing**: bcryptjs with salt rounds of 12 — industry standard, computationally expensive for attackers.
- **JWT**: Stateless authentication. Token signed with a secret key, expires in 7 days. Sent as `Bearer` token in `Authorization` header.
- **No password in response**: The `password` field is never returned in any API response.
- **Generic error messages**: Login returns `"Invalid email or password"` for both wrong email and wrong password — prevents user enumeration.

### Backend Architecture
- **Controller-Route separation**: Business logic stays in controllers, routes only define HTTP mappings — clean and scalable.
- **Input validation**: All inputs validated before hitting the DB (format, length, required fields).
- **Global error handling**: Express `use()` error middleware catches unhandled errors gracefully.
- **Connection pooling**: `pg.Pool` reuses database connections efficiently.

### Frontend Architecture
- **React Context API**: Global `AuthContext` stores user state. Avoids prop-drilling.
- **Axios interceptors**: Automatically attaches JWT to every request. Handles 401/403 by logging out the user.
- **Protected Routes**: `ProtectedRoute` component redirects unauthenticated users to `/login`.
- **Session persistence**: On app load, if a token exists in `localStorage`, the profile is fetched to restore the session.
- **React Router v6**: Declarative routing with `<Navigate>` for redirects and nested route structure.

---

## License
MIT
