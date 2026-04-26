# EMRims SO Suite - Technical Architecture

## Overview

EMRims SO Suite is a full-stack Sales Operations platform built with modern, scalable technologies designed for medical professionals.

## Technology Stack

### Frontend
- **Framework**: React 18+ with TypeScript
- **Build Tool**: Vite (lightning-fast HMR)
- **State Management**: Zustand (lightweight auth) + Redux Toolkit (complex state)
- **Styling**: Tailwind CSS + Framer Motion (animations)
- **HTTP Client**: Axios with interceptors
- **Routing**: React Router v6
- **UI Notifications**: React Hot Toast
- **Form Handling**: React Hook Form

### Backend
- **Runtime**: Node.js 18+
- **Framework**: Express.js 4.x
- **Language**: TypeScript 5.x
- **Database**: PostgreSQL 14+
- **Authentication**: JWT (jsonwebtoken) + bcrypt
- **Validation**: express-validator
- **CORS**: Express CORS middleware
- **Logging**: Winston

### DevOps
- **Containers**: Docker & Docker Compose
- **Database**: PostgreSQL in Docker
- **Configuration**: .env files
- **Package Manager**: npm

---

## Architecture Layers

```
┌─────────────────────────────────────────┐
│     Frontend (React + TypeScript)       │
│  ├─ Pages (Login, Dashboard, etc)      │
│  ├─ Components (Zen Speed UI)          │
│  ├─ Store (Zustand/Redux)              │
│  └─ Services (API clients)             │
├─────────────────────────────────────────┤
│         HTTP / REST API                 │
├─────────────────────────────────────────┤
│     Backend (Express + TypeScript)      │
│  ├─ Routes (API endpoints)             │
│  ├─ Controllers (business logic)       │
│  ├─ Middleware (auth, validation)      │
│  ├─ Services (database operations)     │
│  └─ Models (TypeScript interfaces)     │
├─────────────────────────────────────────┤
│    PostgreSQL Database & Connection    │
└─────────────────────────────────────────┘
```

---

## Directory Structure

### Frontend (`frontend/src/`)
```
src/
├── components/
│   ├── auth/              # Login, logout components
│   ├── ui/                # Zen Speed UI library
│   │   ├── Button.tsx
│   │   ├── Card.tsx
│   │   ├── Modal.tsx
│   │   ├── Input.tsx
│   │   └── Toast.tsx
│   ├── layouts/           # Page layouts
│   └── common/            # Shared components
├── pages/
│   ├── LoginPage.tsx
│   ├── DashboardPage.tsx
│   └── NotFoundPage.tsx
├── services/
│   ├── api.ts             # Axios instance
│   ├── authService.ts     # Auth API calls
│   └── leadService.ts     # Lead API calls
├── store/
│   ├── authStore.ts       # Zustand auth store
│   └── redux/             # Redux slices
├── types/
│   ├── User.ts
│   ├── Lead.ts
│   └── Common.ts
├── utils/
│   ├── validators.ts
│   └── formatters.ts
├── styles/
│   ├── globals.css        # Tailwind + custom
│   └── animations.css     # Zen Speed animations
├── App.tsx
└── main.tsx
```

### Backend (`backend/src/`)
```
src/
├── routes/
│   ├── auth.ts            # /api/auth/* endpoints
│   ├── leads.ts           # /api/leads/* endpoints
│   └── index.ts           # Route aggregation
├── controllers/
│   ├── authController.ts
│   ├── leadController.ts
│   └── measurementController.ts
├── models/
│   ├── User.ts
│   ├── Lead.ts
│   ├── Measurement.ts
│   └── Quote.ts
├── middleware/
│   ├── auth.ts            # JWT verification
│   ├── validation.ts      # Input validation
│   ├── errorHandler.ts    # Error handling
│   └── cors.ts            # CORS configuration
├── services/
│   ├── authService.ts     # Auth logic
│   ├── leadService.ts     # Lead CRUD logic
│   └── measurementService.ts
├── database/
│   ├── connection.ts      # Pool & setup
│   ├── migrations/
│   │   ├── 001_init.sql
│   │   └── ...
│   ├── migrate.ts         # Migration runner
│   └── seed.ts            # Data seeding
├── config/
│   ├── environment.ts     # Env vars
│   └── constants.ts       # App constants
├── types/
│   ├── Express.d.ts       # Express augmentation
│   └── index.ts
└── server.ts              # Express app & startup
```

---

## Authentication Flow

### Login Process
```
1. User enters credentials (username/password)
   ↓
2. Frontend validates input format
   ↓
3. Frontend sends POST /api/auth/login (credentials)
   ↓
4. Backend finds user by username
   ↓
5. Backend compares password with bcrypt hash
   ↓
6. On match: Generate JWT tokens (access + refresh)
   ↓
7. Return tokens to frontend
   ↓
8. Frontend stores in localStorage
   ↓
9. Frontend adds Authorization header to all requests
   ↓
10. Redirect to dashboard ("Remember Me" shows 0.9s toast)
```

### Token Details
- **Access Token**: 24 hours validity, contains user ID + role
- **Refresh Token**: 30 days validity, used to get new access token
- **Storage**: localStorage (survives refresh) + memory (faster access)
- **Remember Me**: Sets localStorage flag for 30-day session

### Protected Routes
- All protected endpoints require Authorization header
- Backend middleware validates JWT signature
- Invalid/expired tokens return 401 Unauthorized
- Frontend automatically redirects to login

---

## Zen Speed UI Design System

### Color Palette
```
Primary:     #FF8C00 (EMR Orange)     - Buttons, CTAs, highlights
Secondary:   #008080 (Deep Teal)      - Data viz, text accents
Background:  #FDFDFD (Matte White)    - Main background
Text:        #333333 (Dark Gray)      - Body text
Border:      #E5E5E5 (Light Gray)     - Card borders, dividers
Success:     #10B981 (Green)          - Success states
Error:       #EF4444 (Red)            - Error states
```

### Animation Timing
- **Pop-out Modal**: `cubic-bezier(0.175, 0.885, 0.32, 1.275)` ~300ms (0.95→1.0 scale)
- **Toast Fade**: 0.9 seconds exact (fade to 0% opacity)
- **Login→Dashboard**: <200ms transition
- **Button Hover**: Spring animation (tension: 300, friction: 10)
- **Icon Tooltip**: Fade in 0.3s, fade out 0.5s
- **Backdrop Blur**: 10px (modal background)

### Component Specs

**Button**
- Primary: EMR Orange background, white text, spring hover
- Secondary: Teal border, teal text, subtle hover
- Ghost: Transparent, text-only, fade hover
- States: Normal, Hover, Active, Disabled, Loading

**Card**
- White background (#FDFDFD)
- Subtle shadow: `0 1px 3px rgba(0,0,0,0.1)`
- Rounded: 8px
- Hover: Slight lift, border highlight

**Modal**
- Backdrop: `backdrop-filter: blur(10px)`
- Background: rgba(0, 0, 0, 0.5)
- Scale: 0.95 → 1.0 over 300ms
- Close: Escape key, outside click, or X button

**Input**
- Matte white background
- Teal focus border: 2px
- Error: Red border + error message
- Success: Green border
- Placeholder: Light gray text

---

## Database Design

### Key Features
- **UUID Primary Keys**: Better for distributed systems and replication
- **JSONB Support**: Flexible measurement data storage
- **Referential Integrity**: Foreign keys prevent orphaned records
- **Audit Trail**: Activities table logs all actions
- **Indexing**: Strategic indexes on frequently queried columns
- **Timestamps**: created_at, updated_at on all tables

### Tables Overview

**users**
- id (UUID, PK)
- username (VARCHAR, UNIQUE)
- password (VARCHAR, bcrypt hash)
- email (VARCHAR, UNIQUE)
- full_name (VARCHAR)
- role (ENUM: admin, sales_officer, manager)
- created_at, updated_at

**leads**
- id (UUID, PK)
- user_id (UUID, FK → users)
- name (VARCHAR)
- phone (VARCHAR)
- email (VARCHAR)
- company (VARCHAR)
- status (ENUM: new, contacted, quoted, converted, lost)
- created_at, updated_at

**measurements**
- id (UUID, PK)
- lead_id (UUID, FK → leads)
- measurement_type (ENUM: manual, emr_recommended, motion_capture)
- data (JSONB) - stores chest, waist, length, etc.
- recommended_size (VARCHAR) - XS to 5XL
- created_at, updated_at

**quotes**
- id (UUID, PK)
- lead_id (UUID, FK → leads)
- user_id (UUID, FK → users)
- status (ENUM: draft, sent, accepted, rejected)
- total_amount (DECIMAL)
- created_at, updated_at

**quote_items**
- id (UUID, PK)
- quote_id (UUID, FK → quotes)
- product_name (VARCHAR)
- quantity (INT)
- unit_price (DECIMAL)
- created_at

**activities**
- id (UUID, PK)
- lead_id (UUID, FK → leads)
- user_id (UUID, FK → users)
- activity_type (ENUM: measurement, quote, call, email, message)
- description (TEXT)
- created_at

### Relationships
```
Users (1) ──→ (Many) Leads
Users (1) ──→ (Many) Quotes
Users (1) ──→ (Many) Activities

Leads (1) ──→ (Many) Measurements
Leads (1) ──→ (Many) Quotes
Leads (1) ──→ (Many) Activities

Quotes (1) ──→ (Many) Quote Items
```

---

## API Design

### REST Conventions
```
GET    /api/resource          - List all resources
GET    /api/resource/:id      - Get single resource
POST   /api/resource          - Create resource
PUT    /api/resource/:id      - Update resource
DELETE /api/resource/:id      - Delete resource
```

### Response Format
```json
{
  "success": true,
  "data": { /* actual data */ },
  "message": "Operation completed successfully"
}
```

### Error Format
```json
{
  "success": false,
  "error": "Validation failed",
  "details": { /* field-level errors */ }
}
```

### HTTP Status Codes
- 200: OK
- 201: Created
- 400: Bad Request (validation error)
- 401: Unauthorized (invalid token)
- 403: Forbidden (insufficient permissions)
- 404: Not Found
- 409: Conflict (duplicate entry)
- 500: Internal Server Error

---

## Security Measures

1. **Password Security**
   - Bcrypt hashing (salt rounds: 10)
   - Never stored in plain text
   - Never returned in API responses

2. **JWT Authentication**
   - Signed with strong secret
   - Time-limited (24h access, 30d refresh)
   - Stored securely in localStorage

3. **CORS Protection**
   - Restricted to whitelisted origins
   - Credentials included for same-domain requests

4. **Input Validation**
   - All endpoints validate input
   - XSS prevention via sanitization
   - SQL injection prevented via parameterized queries

5. **Error Handling**
   - Sensitive info not leaked in error messages
   - Stack traces only in development
   - Proper logging for debugging

6. **HTTPS**
   - Required in production
   - HSTS headers enabled
   - Secure cookies (httpOnly, sameSite)

---

## Development Workflow

### Local Setup
```bash
# 1. Clone and navigate
git clone https://github.com/abugaldis-tech/emrims-so-suite.git
cd emrims-so-suite

# 2. Start all services
docker-compose up

# 3. (First time) Run migrations and seed
cd backend
npm run migrate
npm run seed
```

### Without Docker
```bash
# 1. Start PostgreSQL (local installation)
psql -U postgres  # connect and create database
CREATE DATABASE emrims_db;

# 2. Frontend
cd frontend && npm install && npm run dev

# 3. Backend (new terminal)
cd backend && npm install
cp .env.example .env  # update with your DB config
npm run migrate
npm run seed
npm run dev
```

### Environment Variables

**Backend (.env)**
```
NODE_ENV=development
DB_HOST=localhost
DB_PORT=5432
DB_USER=emrims_user
DB_PASSWORD=emrims_password_secure
DB_NAME=emrims_db
JWT_SECRET=your-super-secret-key-min-32-chars
JWT_EXPIRE=24h
REFRESH_TOKEN_EXPIRE=30d
PORT=5000
CORS_ORIGIN=http://localhost:3000
```

**Frontend (.env)**
```
VITE_API_URL=http://localhost:5000
```

---

## Performance Optimization

### Frontend
- Code splitting (lazy routes)
- Image optimization
- Minification & asset bundling
- Tree-shaking unused code
- React.memo for component optimization

### Backend
- Connection pooling (pg-pool)
- Query optimization & EXPLAIN ANALYZE
- Caching strategies (Redis - future)
- Pagination on list endpoints
- Compression middleware (gzip)

### Database
- Strategic indexes on foreign keys
- Partitioning for large tables (future)
- Query statistics monitoring
- Backup & recovery procedures

---

## Deployment Checklist

- [ ] Set production environment variables
- [ ] Run database migrations on production
- [ ] Configure CORS for production domain
- [ ] Set strong JWT secrets (min 32 chars)
- [ ] Enable HTTPS/TLS
- [ ] Set up monitoring & alerting
- [ ] Configure automated backups
- [ ] Load testing & stress testing
- [ ] Security audit
- [ ] Documentation review

---

## Future Enhancements

- [ ] WebSocket support for real-time updates
- [ ] GraphQL API alongside REST
- [ ] File upload & cloud storage (AWS S3)
- [ ] Email/SMS notification service
- [ ] Advanced analytics & reporting
- [ ] Multi-tenant support
- [ ] API rate limiting (express-rate-limit)
- [ ] Redis caching layer
- [ ] Docker Swarm/Kubernetes deployment
- [ ] Mobile app (React Native)
