# Citizen Grievance & Service Request Portal

> A full-stack e-governance web application that enables citizens to submit, track, and resolve grievances with government departments — built on ShaktiDB, India's sovereign open-source database.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [System Architecture](#system-architecture)
- [Database Schema](#database-schema)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [API Endpoints](#api-endpoints)
- [User Roles](#user-roles)
- [Known Setup Notes](#known-setup-notes)
- [Screenshots](#screenshots)
- [Team](#team)
- [Institution](#institution)

---

## Overview

The **Citizen Grievance & Service Request Portal** is a mini project developed as part of the academic curriculum at **TKM College of Engineering (TKMCE)**. It is a web-based platform that bridges citizens and government officials for efficient grievance resolution.

Citizens can register, submit complaints with detailed information, and track real-time status updates. Government officials can log in, review tickets, assign them to specific departments, update resolution status with notes, and view comprehensive analytics dashboards.

The project is built entirely using **free and open-source technologies**, with **ShaktiDB** (a PostgreSQL 17.7.1.1 fork developed by IITM Pravartak / MeitY) as the primary database — deployed on **Ubuntu 24.04 LTS**.

---

## Features

### Citizen
- 🔐 Register and log in securely with JWT authentication
- 📝 Submit grievances with title, description, category, and location
- 📊 Track real-time status updates (Open → In Progress → Resolved → Closed)
- 📅 View full timeline of all actions taken on a ticket
- ⭐ Rate resolution satisfaction (1–5 stars)
- 🔑 Secure password recovery via forgot password flow
- 📋 Apply to become an official via upgrade request

### Official
- 🔐 Secure login with role-based access control
- 👁️ View all incoming grievances across the system
- 📋 Assign grievances to specific departments
- ✍️ Add resolution notes and update ticket status
- 📈 View analytics dashboard (total, open, in-progress, resolved counts)
- 👥 View read-only list of colleagues (other officials)

### Admin (`241030@tkmce.ac.in` only)
- 👑 All official capabilities
- ✅ Review and approve/reject official upgrade requests from citizens
- 👥 Manage officials — view all current officials and remove them if needed
- 🔒 Only account with admin privileges — set directly in the database
- 🏢 Create and manage departments

---

## Tech Stack

| Category | Technology | Version | License |
|---|---|---|---|
| Operating System | Ubuntu LTS | 24.04 | Free & Open Source |
| Database | ShaktiDB (PostgreSQL fork) | 17.7.1.1 | Open Source |
| Backend Language | Python | 3.12 | PSF License |
| Backend Framework | FastAPI | 0.138.0 | MIT |
| ORM | SQLAlchemy | 2.0.51 | MIT |
| DB Driver | psycopg2-binary | 2.9.12 | LGPL |
| Auth | JWT + python-jose | 3.5.0 | MIT |
| Password Hashing | bcrypt | 5.0.0 | Apache 2.0 |
| Input Validation | pydantic[email] | 2.13.4 | MIT |
| Frontend Language | TypeScript | 6.0.2 | Apache 2.0 |
| Frontend Framework | React | 19.2.6 | MIT |
| Styling | Tailwind CSS | 3.4.17 | MIT |
| HTTP Client | Axios | 1.18.0 | MIT |
| Build Tool | Vite | 8.0.12 | MIT |
| Package Manager | npm | 10.x | Artistic 2.0 |
| Routing | React Router | 7.18.0 | MIT |
| DB GUI | pgAdmin 4 | Latest | PostgreSQL License |
| API Testing | Bruno | Latest | MIT |
| Version Control | Git | Latest | GPL v2 |
| Code Editor | VS Code | Latest | MIT |

---

## System Architecture

```
┌─────────────────────────────────────────────────┐
│                   Browser                        │
│    React 19 + TypeScript + Tailwind CSS 3.4     │
│         (Vite Dev Server on :5173)              │
└──────────────────────┬──────────────────────────┘
                       │ HTTP (Axios + JWT Bearer)
                       ▼
┌─────────────────────────────────────────────────┐
│           FastAPI Backend (Python 3.12)          │
│        Uvicorn running on :8000                  │
│  /auth  /grievances  /admin  /requests          │
└──────────────────────┬──────────────────────────┘
                       │ SQLAlchemy ORM (psycopg2)
                       ▼
┌─────────────────────────────────────────────────┐
│      ShaktiDB (PostgreSQL 17.7.1.1 fork)         │
│      Ubuntu 24.04 LTS — Port 5433               │
│      Socket: /tmp/.s.PGSQL.5433                 │
│  users · grievances · departments ·             │
│  grievance_updates · ratings · official_requests│
└─────────────────────────────────────────────────┘
```

---

## Database Schema

**users**
```
id               SERIAL PRIMARY KEY
name             VARCHAR(100)
email            VARCHAR(150) UNIQUE
hashed_password  VARCHAR
role             ENUM(citizen, official, admin)
created_at       TIMESTAMP
```

**departments**
```
id    SERIAL PRIMARY KEY
name  VARCHAR(100) UNIQUE
```

Pre-seeded departments: Water Supply, Roads & Infrastructure, Electricity, Sanitation, Public Safety

**grievances**
```
id             SERIAL PRIMARY KEY
title          VARCHAR(200)
description    TEXT
category       VARCHAR(100)
location       VARCHAR(200)
status         ENUM(open, in_progress, resolved, closed)
citizen_id     FK → users.id
department_id  FK → departments.id  (nullable)
created_at     TIMESTAMP
updated_at     TIMESTAMP
```

**grievance_updates**
```
id            SERIAL PRIMARY KEY
grievance_id  FK → grievances.id
note          TEXT
status        ENUM(open, in_progress, resolved, closed)
updated_by    FK → users.id
created_at    TIMESTAMP
```

**ratings**
```
id            SERIAL PRIMARY KEY
grievance_id  FK → grievances.id
citizen_id    FK → users.id
score         INTEGER (1–5)
created_at    TIMESTAMP
```

**official_requests**
```
id            SERIAL PRIMARY KEY
user_id       FK → users.id (UNIQUE)
reason        TEXT
status        ENUM(pending, approved, rejected)
created_at    TIMESTAMP
reviewed_at   TIMESTAMP (nullable)
```

---

## Project Structure

```
ShaktiDB_Project/
│
├── backend/
│   ├── venv/                          # Python virtual environment
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── auth.py                    # JWT register/login, password hashing
│   │   ├── grievances.py              # Submit, track, timeline, rate
│   │   ├── admin.py                   # Dashboard, assign, status, analytics
│   │   ├── requests.py                # Official upgrade requests + management
│   │   └── dependencies.py            # JWT validation, role guards
│   ├── models.py                      # SQLAlchemy ORM models
│   ├── schemas.py                     # Pydantic request/response schemas
│   ├── database.py                    # ShaktiDB connection (psycopg2 on :5433)
│   ├── main.py                        # FastAPI app, CORS, router registration
│   ├── .env                           # Environment variables (not committed)
│   └── requirements.txt               # Python dependencies
│
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   │   └── axios.ts               # Axios instance with JWT interceptor
│   │   ├── pages/
│   │   │   ├── LoginPage.tsx          # Login form
│   │   │   ├── RegisterPage.tsx       # Citizen-only registration (no role picker)
│   │   │   ├── ForgotPasswordPage.tsx # Password recovery flow
│   │   │   ├── DashboardPage.tsx      # Citizen grievance list
│   │   │   ├── SubmitPage.tsx         # Submit new grievance form
│   │   │   ├── TrackPage.tsx          # Grievance detail + status timeline
│   │   │   ├── AdminPage.tsx          # Shared dashboard (admin + official views)
│   │   │   └── RequestOfficialPage.tsx# Apply to become official
│   │   ├── components/                # Reusable UI components
│   │   ├── App.tsx                    # Routes + PrivateRoute guard
│   │   └── main.tsx                   # React entry point
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── tsconfig.json
│   └── package.json
│
└── README.md
```

---

## Getting Started

### Prerequisites

- Ubuntu 24.04 LTS
- ShaktiDB installed and running (port 5433)
- Python 3.12
- Node.js 20.x+ with npm
- Git

### 1. Clone the repository

```bash
git clone https://github.com/Abhijith2005binu/ShaktiDB_Project.git
cd ShaktiDB_Project
```

### 2. Start ShaktiDB

```bash
sudo systemctl start shaktidb.service
sudo -u postgres /usr/lib/postgresql/17.7.1.1/bin/psql -p 5433
```

```sql
CREATE DATABASE grievance_portal;
CREATE USER portal_user WITH PASSWORD 'securepassword123';
GRANT ALL PRIVILEGES ON DATABASE grievance_portal TO portal_user;
\q
```

### 3. Set up the backend

```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install "pydantic[email]"
```

Create `.env` in `backend/`:

```env
DATABASE_URL=postgresql://portal_user:securepassword123@localhost:5433/grievance_portal
SECRET_KEY=your-super-secret-key-change-this
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

Run the backend:

```bash
uvicorn main:app --reload
# API:  http://localhost:8000
# Docs: http://localhost:8000/docs
```

### 4. Set up the frontend

```bash
cd ../frontend
npm install
npm run dev
# Frontend: http://localhost:5173
```

### 5. Seed departments

```sql
INSERT INTO departments (name) VALUES
  ('Water Supply'),
  ('Roads & Infrastructure'),
  ('Electricity'),
  ('Sanitation'),
  ('Public Safety');
```

### 6. Set up the admin account

Register normally at `/register` using `241030@tkmce.ac.in`, then run in pgAdmin or psql:

```sql
UPDATE users SET role = 'admin' WHERE email = '241030@tkmce.ac.in';
```

---

## API Endpoints

### Auth (`/auth`)
| Method | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/auth/register` | Register as citizen | Public |
| POST | `/auth/login` | Login, receive JWT token | Public |

### Grievances (`/grievances`)
| Method | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/grievances/` | Submit a new grievance | Citizen |
| GET | `/grievances/my` | List own grievances | Citizen |
| GET | `/grievances/{id}` | Get grievance details | Citizen |
| GET | `/grievances/{id}/timeline` | Get status update history | Citizen |
| POST | `/grievances/{id}/rate` | Rate a resolved grievance | Citizen |

### Admin (`/admin`)
| Method | Endpoint | Description | Auth |
|---|---|---|---|
| GET | `/admin/grievances` | List all grievances | Official/Admin |
| PUT | `/admin/grievances/{id}/assign` | Assign to department | Official/Admin |
| PUT | `/admin/grievances/{id}/status` | Update status with note | Official/Admin |
| GET | `/admin/analytics` | Analytics counts by status | Official/Admin |
| GET | `/admin/departments` | List all departments | Any logged in |
| POST | `/admin/departments` | Create a department | Official/Admin |

### Official Requests (`/requests`)
| Method | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/requests/official` | Submit upgrade request | Citizen |
| GET | `/requests/official/my` | Check own request status | Citizen |
| GET | `/requests/official/all` | View all requests | Admin |
| PUT | `/requests/official/{id}/approve` | Approve request | Admin |
| PUT | `/requests/official/{id}/reject` | Reject request | Admin |
| GET | `/requests/officials/all` | List all officials (with details) | Admin |
| PUT | `/requests/officials/{id}/remove` | Remove an official | Admin |
| GET | `/requests/officials/colleagues` | Read-only colleagues list | Official |

---

## User Roles

| Role | How Assigned | Dashboard Title | Capabilities |
|---|---|---|---|
| **Citizen** | Self-register at `/register` | — | Submit grievances, track status, view timeline, rate resolutions, apply to become official |
| **Official** | Admin approves upgrade request | Official Dashboard | View all grievances, assign departments, update status, view analytics, view colleagues list |
| **Admin** | Set manually in DB (`241030@tkmce.ac.in`) | Admin Dashboard | All official capabilities + approve/reject requests + manage/remove officials + create departments |

### Dashboard view by role

| Section | Admin | Official |
|---|---|---|
| Analytics cards | ✅ | ✅ |
| Pending official requests | ✅ | ❌ |
| Manage Officials (with Remove) | ✅ | ❌ |
| My Colleagues (read-only) | ❌ | ✅ |
| All Grievances + Update status | ✅ | ✅ |

---

## Known Setup Notes

| Item | Detail |
|---|---|
| ShaktiDB service name | `shaktidb.service` (not `postgresql`) |
| ShaktiDB version | 17.7.1.1 |
| Default port | **5433** (not 5432) |
| Socket location | `/tmp/.s.PGSQL.5433` |
| Data directory | `/data/sdb` |
| Connect command | `sudo -u postgres /usr/lib/postgresql/17.7.1.1/bin/psql -p 5433` |
| pgAdmin 4 host | `/tmp` with port `5433` |
| pydantic email | Run `pip install "pydantic[email]"` separately after requirements |
| bcrypt | Version 5.0.0 — do not use passlib |
| Node.js | Requires v20.x+ (Vite 8 requirement) |
| Tailwind CSS | Must use v3.4.17 — v4 has breaking changes |
| venv activation | Always run `source venv/bin/activate` before `uvicorn` |
| Admin account | Must be set manually in DB after registering normally |
| Role restriction | Registration is locked to citizen — no role picker in UI |

---

## Screenshots

- Login page (`/login`)
  <img width="614" height="758" alt="Screenshot from 2026-07-26 19-31-43" src="https://github.com/user-attachments/assets/93392185-19ba-4d84-86a7-54490265cc13" />

- Registration page (`/register`)
  <img width="523" height="609" alt="Screenshot from 2026-07-26 19-53-23" src="https://github.com/user-attachments/assets/2ec0a6bc-723e-48d7-a352-0232ba846db6" />

- Forgot password (`/forgot-password`)
  <img width="514" height="536" alt="Screenshot from 2026-07-26 19-54-05" src="https://github.com/user-attachments/assets/2397435a-a799-40a1-88d2-7b557aa44abf" />

- Citizen dashboard (`/dashboard`)
  <img width="1803" height="957" alt="Screenshot from 2026-07-26 19-31-11" src="https://github.com/user-attachments/assets/948e1f0c-4978-46fe-acc7-e4be56b6e316" />

- Submit grievance form (`/submit`)
  <img width="707" height="619" alt="Screenshot from 2026-07-26 19-55-47" src="https://github.com/user-attachments/assets/04ea05e1-de03-49e9-82ba-9f68e6656b70" />

- Grievance tracking & timeline (`/track/:id`)
  <img width="665" height="643" alt="Screenshot from 2026-07-26 19-59-01" src="https://github.com/user-attachments/assets/ca3839bd-3208-4691-b9d7-6a5a1bd65328" />

- Apply as official (`/request-official`)
  <img width="660" height="391" alt="image" src="https://github.com/user-attachments/assets/d6f9141c-a598-4e19-a4b9-87732c4f6dfc" />

- Admin dashboard — analytics, pending requests, manage officials, grievances (`/admin`)
  <img width="1793" height="946" alt="Screenshot from 2026-07-26 19-32-46" src="https://github.com/user-attachments/assets/9c7efcbf-e6b6-4682-9add-6fb402b191f6" />

- Official dashboard — analytics, colleagues list, grievances (`/admin`)
  <img width="1793" height="946" alt="Screenshot from 2026-07-26 19-45-03" src="https://github.com/user-attachments/assets/b2595853-2eb3-4287-8dac-a69c6a022afb" />

- pgAdmin 4 showing ShaktiDB tables
  <img width="1855" height="1048" alt="Screenshot from 2026-07-26 20-03-55" src="https://github.com/user-attachments/assets/a597a1b3-d642-49d2-8f38-06c44a4c4d29" />

  <img width="1855" height="1048" alt="Screenshot from 2026-07-26 20-04-05" src="https://github.com/user-attachments/assets/1476b75c-0ced-4817-8b74-19d8edca43e6" />


---

## Team

| Name | Role | Contact |
|---|---|---|
| Abhijith B | Full Stack Developer | [@Abhijith2005binu](https://github.com/Abhijith2005binu) |

---

## Institution

**TKM College of Engineering (TKMCE)**
Department of Computer Science & Engineering
Mini Project — Academic Year 2024–25

> Built with ShaktiDB — India's sovereign open-source RDBMS developed by IITM Pravartak in collaboration with MeitY and C-DAC.

---

## License

This project is built entirely on open-source technologies. Refer to the Tech Stack section for individual component licenses.

---

*Last Updated: July 2026 | Repository: [ShaktiDB_Project](https://github.com/Abhijith2005binu/ShaktiDB_Project)*
