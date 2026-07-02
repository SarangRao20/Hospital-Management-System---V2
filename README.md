<p align="center">
  <img src="https://img.shields.io/badge/status-in%20development-22c55e" alt="Status">
  <img src="https://img.shields.io/badge/python-3.10+-2563eb" alt="Python">
  <img src="https://img.shields.io/badge/framework-Flask-059669" alt="Flask">
  <img src="https://img.shields.io/badge/frontend-Vue.js-4fc08d" alt="Vue.js">
  <img src="https://img.shields.io/badge/IITM-MAD2%20Project-7c3aed" alt="IITM">
  <img src="https://img.shields.io/badge/license-MIT-78716c" alt="License">
</p>

<h1 align="center">Hospital Management System — V2</h1>
<p align="center">
  <em>IIT Madras BS in Data Science — Modern Application Development (MAD2) Project</em>
</p>

<p align="center">
  A full-stack, multi-role hospital management platform connecting <strong>Admins</strong>, <strong>Doctors</strong>, and <strong>Patients</strong> through a secure, role-based system built with Flask, Vue.js, SQLite, Redis, and Celery.
</p>

---

## Overview

Hospital Management System V2 brings order to hospital chaos. From appointment booking to treatment history, it provides a unified interface for all three stakeholders while keeping automated reminders, monthly reports, and async exports running quietly in the background.

### Built for MAD2

This project was developed as the capstone for IIT Madras's **Modern Application Development (MAD2)** course, demonstrating:

- Full-stack web development with Flask and Vue.js
- RESTful API design with role-based access control (RBAC)
- Asynchronous task processing with Celery and Redis
- Automated email/SMS reminders and report generation
- JWT-based authentication and session management

---

## Features

### Role-Based Access

| Role | Capabilities |
|------|-------------|
| **Admin** | Manage doctors, patients, departments; view system-wide reports; oversee appointments |
| **Doctor** | View patient queue, access treatment history, update prescriptions, manage availability |
| **Patient** | Book/cancel appointments, view medical history, receive reminders, download reports |

### Core Modules

- **Appointment Management** — Book, reschedule, cancel with conflict detection and status tracking
- **Patient Records** — Treatment history, diagnoses, prescriptions, lab results
- **Doctor Scheduling** — Availability windows, slot management, leave tracking
- **Automated Reminders** — Celery + Redis scheduled tasks for appointment reminders via email/SMS
- **Monthly Reports** — Async PDF/CSV report generation for hospital administration
- **Authentication & Authorization** — JWT tokens with role-scoped API access
- **Dashboard Analytics** — Key metrics per role (patient count, appointments, revenue)

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Backend** | Flask 3.x + Flask-Restx | REST API with Swagger docs |
| **Frontend** | Vue.js 3 + Vite | Reactive SPA interface |
| **Database** | SQLite (SQLAlchemy ORM) | Relational data storage |
| **Cache** | Redis | Celery broker, session store |
| **Async** | Celery | Email, reports, exports |
| **Auth** | Flask-JWT-Extended | JWT-based RBAC |
| **Templates** | Jinja2 | Server-rendered pages |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Vue.js SPA (Frontend)                     │
│            Admin Dashboard · Doctor Portal · Patient Portal  │
└──────────────────────────┬──────────────────────────────────┘
                           │ REST API (JWT)
┌──────────────────────────▼──────────────────────────────────┐
│                   Flask REST API (Backend)                    │
│       Namespaces: /auth, /patients, /doctors, /appointments   │
│                   /reports, /dashboard, /admin                │
└─────────┬────────────────────────────┬───────────────────────┘
          │                            │
┌─────────▼─────────┐       ┌─────────▼─────────────────┐
│    SQLite / SQLAlchemy  │    Redis + Celery Workers      │
│   Patients · Doctors ·  │   Email · Reports · Reminders  │
│   Appointments · Users   │   Async Task Queue            │
└─────────────────────────┘   └───────────────────────────┘
```

---

## Repository Structure

```
Hospital-Management-System---V2/
│
├── backend/                   # Flask REST API + Celery workers
│   ├── app.py                 # Application factory
│   ├── config.py              # JWT, DB, Redis configuration
│   ├── models.py              # SQLAlchemy ORM models
│   ├── requirements.txt
│   ├── api/                   # REST namespaces (auth, patients, doctors, appointments, reports, dashboard, admin)
│   └── tasks/                 # Celery async tasks (email reminders, report generation)
│
├── frontend/                  # Vue.js 3 + Vite SPA
│   ├── src/
│   │   ├── components/        # Reusable UI components
│   │   ├── views/             # Page views per role
│   │   ├── router/            # Vue Router config
│   │   └── store/             # Pinia state management
│   └── package.json
│
└── README.md
```

---

## Setup

### Prerequisites

- Python 3.10+
- Node.js 18+
- Redis 7+
- SQLite

### Backend

```bash
git clone https://github.com/SarangRao20/Hospital-Management-System---V2.git
cd Hospital-Management-System---V2/backend

python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt

python app.py
# → API at http://localhost:5000
# → Swagger at http://localhost:5000/docs
```

### Frontend

```bash
cd ../frontend
npm install
npm run dev
# → SPA at http://localhost:5173
```

### Celery Worker

```bash
# Terminal 3 — requires Redis running
celery -A tasks.celery_tasks.celery worker --loglevel=info
```

---

## API Overview

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/auth/login` | POST | — | Login, returns JWT |
| `/api/auth/register` | POST | — | Register patient |
| `/api/patients` | GET | Admin | List all patients |
| `/api/patients/<id>` | GET | Doctor/Admin | Patient details + history |
| `/api/doctors` | GET | Public | List doctors |
| `/api/appointments` | GET/POST | All | List / book appointments |
| `/api/appointments/<id>` | PUT/DELETE | All | Update / cancel appointment |
| `/api/reports/monthly` | GET | Admin | Download monthly report |
| `/api/dashboard` | GET | Role-based | Role-specific analytics |

Full Swagger documentation: `http://localhost:5000/docs`

---

## Features in Detail

### Appointment Workflow

```
Patient selects doctor → Views available slots → Books appointment
    → JWT-verified request → Conflict check → DB insert
    → Celery schedules reminder email → Status: Pending
    → Doctor confirms → Status: Confirmed / Completed / Cancelled
```

### Automated Reminders (Celery + Redis)

Appointment reminders are scheduled as background Celery tasks:
- **24-hour reminder**: Email to patient before scheduled appointment
- **Missed appointment**: Notification to doctor if no-show
- **Monthly summaries**: Aggregated report emailed to admin

### Role-Based Access Control (RBAC)

JWT tokens encode the user's role at login. Every API request is validated against role-specific permissions:

```python
# Example: Only doctors can update prescriptions
@api.doc(security='jwt')
@jwt_required()
def put(self, patient_id):
    current_user = get_jwt_identity()
    if current_user['role'] != 'doctor':
        return {'message': 'Unauthorized'}, 403
```

---

## Author

**Sarang Rao**  
IIT Madras — BS in Data Science and Applications  
Roll No: 24f2000232  
[GitHub](https://github.com/SarangRao20) · [LinkedIn](https://linkedin.com/in/sarang-rao-262bbb324)

---

<p align="center">
  <sub>Built for the MAD2 Capstone — IIT Madras Online BS Degree Program</sub>
</p>
