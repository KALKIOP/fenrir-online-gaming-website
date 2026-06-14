# FENRIR PROJECT — DEVELOPMENT & SETUP INSTRUCTIONS

This document outlines the system requirements, technical architecture, database schema, folder structure, API definitions, development workflow, and deployment pipelines for the Fenrir Online Gaming Website.

---

## 1. RECOMMENDED TECH STACK

To deliver a high-performance, real-time gaming ecosystem, the following technical architecture is specified:

*   **Frontend Framework:** React (built with Vite for fast HMR and optimized production bundling).
*   **Styling System:** Vanilla CSS (structured with CSS Variables, Flexbox/Grid layouts, and glassmorphic designs to fit the modern gaming theme).
*   **Backend Framework:** Node.js with Express.js (REST API layer).
*   **Real-time Communication:** Socket.io (WebSocket framework for multiplayer lobbies, matchmaking notifications, and real-time chat).
*   **Database:** PostgreSQL (relational database for storing user accounts, financial transactions, profile details, match history, and tournament records).
*   **Object Relational Mapping (ORM):** Sequelize ORM or Prisma (for database schema migration, seeding, and secure queries).
*   **Caching & Queue Layer:** Redis (for session caching, active socket mappings, and queue handling for multiplayer matchmaking).
*   **Authentication:** JSON Web Tokens (JWT) stored in secure HTTP-only cookies, with bcrypt for password hashing.
*   **Payment Processor:** Stripe API integration for subscribing to "Fenrir Pro" and buying premium cosmetic packages.

---

## 2. FOLDER STRUCTURE

The project is structured as a full-stack monorepo separating concerns between client-side assets and server-side logic.

```
fenrir-gaming-platform/
│
├── backend/
│   ├── src/
│   │   ├── config/             # DB and external services config
│   │   │   ├── database.js     # Sequelize/Prisma initialization
│   │   │   ├── redis.js        # Redis client connection
│   │   │   └── passport.js     # OAuth strategies (Google/Discord)
│   │   │
│   │   ├── controllers/        # Request handlers & business logic
│   │   │   ├── authController.js
│   │   │   ├── userController.js
│   │   │   ├── gameController.js
│   │   │   ├── lobbyController.js
│   │   │   ├── tournamentController.js
│   │   │   └── adminController.js
│   │   │
│   │   ├── models/             # Database tables (Sequelize/Prisma schemas)
│   │   │   ├── User.js
│   │   │   ├── Profile.js
│   │   │   ├── Game.js
│   │   │   ├── Match.js
│   │   │   ├── Tournament.js
│   │   │   └── Transaction.js
│   │   │
│   │   ├── routes/             # API Endpoints
│   │   │   ├── authRoutes.js
│   │   │   ├── userRoutes.js
│   │   │   ├── gameRoutes.js
│   │   │   ├── tournamentRoutes.js
│   │   │   └── adminRoutes.js
│   │   │
│   │   ├── sockets/            # WebSocket logic
│   │   │   ├── socketManager.js
│   │   │   ├── lobbyHandler.js
│   │   │   └── chatHandler.js
│   │   │
│   │   ├── middlewares/        # Express request guards
│   │   │   ├── authMiddleware.js
│   │   │   └── adminMiddleware.js
│   │   │
│   │   ├── app.js              # Express app configurations
│   │   └── server.js           # Server startup (HTTP & WebSocket listener)
│   │
│   ├── tests/                  # Integration and unit tests
│   ├── package.json
│   └── .env.example
│
├── frontend/
│   ├── public/                 # Static assets (icons, fallback images)
│   ├── src/
│   │   ├── assets/             # Brand logos, design system graphic tokens
│   │   │
│   │   ├── components/         # Reusable UI components
│   │   │   ├── common/         # Buttons, Inputs, Cards, Loaders
│   │   │   ├── layout/         # Navbar, Sidebar, Footer
│   │   │   ├── lobby/          # Chatbox, ParticipantList, RoomCard
│   │   │   └── tournament/     # BracketNode, RegisterModal
│   │   │
│   │   ├── contexts/           # Global react states
│   │   │   ├── AuthContext.jsx
│   │   │   └── SocketContext.jsx
│   │   │
│   │   ├── hooks/              # Custom utilities
│   │   │   ├── useAuth.js
│   │   │   └── useSocket.js
│   │   │
│   │   ├── pages/              # View screens
│   │   │   ├── Home.jsx
│   │   │   ├── Login.jsx
│   │   │   ├── Register.jsx
│   │   │   ├── Profile.jsx
│   │   │   ├── Leaderboards.jsx
│   │   │   ├── Tournaments.jsx
│   │   │   └── AdminDashboard.jsx
│   │   │
│   │   ├── services/           # Network request clients
│   │   │   ├── api.js          # Axios wrapper
│   │   │   └── socket.js       # WebSocket wrapper
│   │   │
│   │   ├── styles/             # Application visual styles
│   │   │   ├── index.css       # Core typography and resetting
│   │   │   ├── theme.css       # CSS Variables (neon colors, spacing tokens)
│   │   │   └── glass.css       # Glassmorphism utility rules
│   │   │
│   │   ├── App.jsx             # React router configuration
│   │   └── main.jsx            # DOM React mount entry
│   │
│   ├── package.json
│   ├── vite.config.js
│   └── .env.example
│
└── README.md
```

---

## 3. DATABASE STRUCTURE (POSTGRESQL RELATIONAL ENTITIES)

```mermaid
erDiagram
    USERS ||--|| PROFILES : "has"
    USERS ||--o{ TRANSACTIONS : "pays"
    USERS ||--o{ TOURNAMENT_REGISTRATIONS : "enters"
    PROFILES ||--o{ MATCH_HISTORY : "plays"
    GAMES ||--o{ MATCH_HISTORY : "runs"
    GAMES ||--o{ TOURNAMENTS : "used_in"
    TOURNAMENTS ||--o{ TOURNAMENT_REGISTRATIONS : "has"
```

### Table 1: `users`
Stores login credentials, core authentication metadata, and administrative privileges.
*   `id`: `UUID` (Primary Key, Auto-generated)
*   `email`: `VARCHAR(255)` (Unique, Indexed, Not Null)
*   `password_hash`: `VARCHAR(255)` (Not Null, Hashed using bcrypt)
*   `role`: `ENUM('user', 'moderator', 'admin')` (Default: 'user')
*   `created_at`: `TIMESTAMP` (Not Null)
*   `updated_at`: `TIMESTAMP` (Not Null)

### Table 2: `profiles`
Holds gaming identity, stats, wallet balances, and active subscription status.
*   `id`: `UUID` (Primary Key, Auto-generated)
*   `user_id`: `UUID` (Foreign Key referencing `users.id`, Cascade Delete)
*   `username`: `VARCHAR(50)` (Unique, Indexed, Not Null)
*   `avatar_url`: `VARCHAR(512)` (Nullable)
*   `bio`: `TEXT` (Nullable)
*   `level`: `INTEGER` (Default: 1)
*   `xp`: `INTEGER` (Default: 0)
*   `mmr`: `INTEGER` (Default: 1000 - Matchmaking Rating)
*   `coins`: `INTEGER` (Default: 500 - In-game currency)
*   `premium_until`: `TIMESTAMP` (Nullable, controls subscription expiration)
*   `created_at`: `TIMESTAMP`

### Table 3: `games`
System metadata directory of games available on the Fenrir portal.
*   `id`: `UUID` (Primary Key)
*   `name`: `VARCHAR(100)` (Unique, Not Null)
*   `description`: `TEXT` (Not Null)
*   `category`: `VARCHAR(50)` (Not Null)
*   `max_players`: `INTEGER` (Not Null, e.g., 2 for chess, 8 for racing)
*   `url`: `VARCHAR(512)` (WebGL game bundle URI, Not Null)
*   `is_active`: `BOOLEAN` (Default: true)

### Table 4: `match_history`
Detailed records of finished multiplayer matches.
*   `id`: `UUID` (Primary Key)
*   `game_id`: `UUID` (Foreign Key referencing `games.id`)
*   `player_one_id`: `UUID` (Foreign Key referencing `profiles.id`)
*   `player_two_id`: `UUID` (Foreign Key referencing `profiles.id`)
*   `winner_id`: `UUID` (Foreign Key referencing `profiles.id`, Nullable in case of draw)
*   `player_one_score`: `INTEGER`
*   `player_two_score`: `INTEGER`
*   `mmr_change`: `INTEGER` (Rating points swapped between players)
*   `coins_awarded`: `INTEGER` (Virtual coins distributed)
*   `played_at`: `TIMESTAMP` (Default: NOW())

### Table 5: `tournaments`
Stores esports competition structures.
*   `id`: `UUID` (Primary Key)
*   `game_id`: `UUID` (Foreign Key referencing `games.id`)
*   `name`: `VARCHAR(150)` (Not Null)
*   `prize_pool`: `INTEGER` (Coin or fiat currency equivalent)
*   `max_participants`: `INTEGER` (Not Null, e.g., 16, 32, 64)
*   `status`: `ENUM('upcoming', 'active', 'completed', 'cancelled')`
*   `start_time`: `TIMESTAMP` (Not Null)
*   `end_time`: `TIMESTAMP` (Nullable)

### Table 6: `tournament_registrations`
Tracks users who have registered to play in a tournament.
*   `id`: `UUID` (Primary Key)
*   `tournament_id`: `UUID` (Foreign Key referencing `tournaments.id`, Cascade Delete)
*   `user_id`: `UUID` (Foreign Key referencing `users.id`, Cascade Delete)
*   `status`: `ENUM('registered', 'disqualified', 'eliminated', 'active')`
*   `joined_at`: `TIMESTAMP` (Default: NOW())

### Table 7: `transactions`
Immutable payment audits from Stripe fiat sales.
*   `id`: `UUID` (Primary Key)
*   `user_id`: `UUID` (Foreign Key referencing `users.id`)
*   `stripe_charge_id`: `VARCHAR(255)` (Unique)
*   `amount`: `INTEGER` (Amount in cents, e.g., 999 for $9.99)
*   `currency`: `VARCHAR(10)` (Default: 'usd')
*   `description`: `VARCHAR(255)` (e.g., 'Fenrir Pro Subscription - 1 Month')
*   `status`: `ENUM('pending', 'succeeded', 'failed', 'refunded')`
*   `created_at`: `TIMESTAMP`

---

## 4. API SPECIFICATIONS (REST LOGICAL ENDPOINTS)

### Authentication Module (`/api/auth`)
*   `POST /register`
    *   *Payload:* `{ email, username, password }`
    *   *Response:* `201 Created` with secure HTTP-only session cookie.
*   `POST /login`
    *   *Payload:* `{ email, password }`
    *   *Response:* `200 OK` with JSON user object.
*   `POST /logout`
    *   *Response:* `200 OK` with cleared cookie header.
*   `GET /me`
    *   *Headers:* Requires session cookie.
    *   *Response:* `200 OK` with user verification details.

### Profile Module (`/api/users`)
*   `GET /profile/:username`
    *   *Response:* `200 OK` with profile fields, levels, badges, and recent matches.
*   `PUT /profile`
    *   *Payload:* `{ bio, avatar_url, country }` (Multipart upload supported for avatar)
    *   *Response:* `200 OK` with updated profile.

### Game Registry (`/api/games`)
*   `GET /`
    *   *Params:* `?category=arcade&search=run&sort=popular`
    *   *Response:* `200 OK` with matching games array.
*   `GET /:id`
    *   *Response:* `200 OK` with game parameters.
*   `POST /` (Admin-Only)
    *   *Payload:* `{ name, description, category, max_players, WebGL_bundle_url }`
    *   *Response:* `210 Created`.

### Esports Tournaments (`/api/tournaments`)
*   `GET /`
    *   *Params:* `?status=upcoming`
    *   *Response:* `200 OK` with tournaments list.
*   `POST /register/:tournament_id`
    *   *Response:* `200 OK` confirming registry verification.
*   `GET /bracket/:tournament_id`
    *   *Response:* `200 OK` with structured nested JSON representing match nodes.

### Payments & Premium Shop (`/api/wallet`)
*   `POST /checkout/subscription`
    *   *Payload:* `{ planId }` (e.g., monthly, yearly)
    *   *Response:* `200 OK` with Stripe Checkout Session redirect URL.
*   `POST /webhook/stripe`
    *   *Payload:* Raw Stripe Event Object (validated using signature header).
    *   *Response:* `200 OK` (Updates User's premium status and creates transaction record).

---

## 5. SYSTEM SETUP INSTRUCTIONS

Follow these steps to spin up the development environment locally.

### Prerequisites
1.  **Node.js** (LTS version 18.x or 20.x installed)
2.  **PostgreSQL** (Active locally or hosted externally on Neon/RDS)
3.  **Redis** (Active local instance on port 6379, or external link)

### Step A: Database Initialization
Create an empty database named `fenrir_db` inside PostgreSQL:
```bash
createdb -U postgres fenrir_db
```

### Step B: Backend Server Setup
1.  Navigate to the backend directory:
    ```bash
    cd backend
    ```
2.  Install all required server dependencies:
    ```bash
    npm install
    ```
3.  Create your environment configuration file:
    ```bash
    cp .env.example .env
    ```
4.  Open `.env` and configure your credentials:
    ```env
    PORT=5000
    DATABASE_URL=postgres://postgres:password@localhost:5473/fenrir_db
    REDIS_URL=redis://localhost:6379
    JWT_SECRET=super_secret_jwt_token_for_fenrir
    STRIPE_SECRET_KEY=sk_test_...
    STRIPE_WEBHOOK_SECRET=whsec_...
    GOOGLE_CLIENT_ID=...
    GOOGLE_CLIENT_SECRET=...
    ```
5.  Run Sequelize migrations and seed database with sample game modules:
    ```bash
    npm run db:migrate
    npm run db:seed
    ```
6.  Start the Express + Socket.io server:
    ```bash
    npm run dev
    ```
    *The server will start listening on port 5000.*

### Step C: Frontend Client Setup
1.  Navigate to the frontend directory:
    ```bash
    cd ../frontend
    ```
2.  Install UI dependencies:
    ```bash
    npm install
    ```
3.  Configure your environment configuration file:
    ```bash
    cp .env.example .env
    ```
4.  Open `.env` and set the api endpoints link:
    ```env
    VITE_API_URL=http://localhost:5000
    VITE_SOCKET_URL=http://localhost:5000
    ```
5.  Launch Vite development server:
    ```bash
    npm run dev
    ```
    *The frontend dashboard will run on `http://localhost:5173`.*

---

## 6. DEVELOPMENT WORKFLOW & PROCEDURES

To maintain code health, enforce these practices in Git:

### Branch Management
*   **`main`:** The production-ready codebase. Protected branch (requires approved PRs to write to).
*   **`develop`:** Integration branch for active features.
*   **`feature/[name-of-feature]`:** Sandbox branch created from `develop`. Merge back via Pull Requests.

### Commits Convention
Enforce Conventional Commits patterns:
*   `feat: add real-time matchmaking queue via redis`
*   `fix: resolve socket disconnection mmr memory leak`
*   `docs: update API endpoints definitions for tournaments`
*   `style: design glassmorphic profile achievements cards`

### Quality Gates
*   **Linting:** Run `npm run lint` on pre-commit hooks to avoid formatting mistakes.
*   **Testing:** Run `npm test` to verify integration tests for the authentication and transaction API endpoints before merging to `develop`.

---

## 7. DEPLOYMENT INSTRUCTIONS

The application utilizes continuous deployment configurations for live releases.

### Database Hosting
*   Utilize **Neon.tech** or **Supabase** for fully managed PostgreSQL instances.
*   Ensure the database connection string uses SSL connection requirements (`?sslmode=require`).

### Backend Service (Deploying to Render / Heroku / AWS)
1.  Connect your GitHub repository to **Render** and create a **Web Service**.
2.  Select environment **Node**, build command: `cd backend && npm install`, start command: `cd backend && node src/server.js`.
3.  Add environment variables in the dashboard corresponding to your production keys.
4.  Provision a **Render Redis** instance and copy the connection string into the backend `REDIS_URL`.

### Frontend Web (Deploying to Vercel / Netlify)
1.  Connect your repository to **Vercel** and select **Vite** configuration.
2.  Root Directory: `frontend`
3.  Build Command: `npm run build`
4.  Output Directory: `dist`
5.  Set Environment Variable: `VITE_API_URL` and `VITE_SOCKET_URL` to point to the live Render Backend URL.

================================================================================
