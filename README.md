# Django Machine Test – Clients & Projects API

Implements the required REST APIs using **Django 5 + Django REST Framework + JWT** and **PostgreSQL** (no SQLite).

## Quick Start

### 0) Clone & enter
```bash
git clone <your-repo-url> nimap-api
cd nimap-api
```

### 1) Python env & dependencies
```bash
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 2) Database (PostgreSQL)
Use Docker (recommended) or your local Postgres.

**Option A: Docker**
1. Copy env & start DB
```bash
cp .env.example .env
docker compose up -d db
```
This starts a Postgres 16 DB on port 5432 with defaults from `.env`.

**Option B: Local Postgres**
Create DB & user manually (example):
```sql
CREATE DATABASE nimapdb;
CREATE USER nimapuser WITH PASSWORD 'nimappass';
GRANT ALL PRIVILEGES ON DATABASE nimapdb TO nimapuser;
```

Update `.env` to match your local credentials.

### 3) Django setup
```bash
# Migrate
python manage.py migrate

# Create a superuser (for admin-only user creation)
python manage.py createsuperuser
```

### 4) Run
```bash
python manage.py runserver
```

Open Swagger docs: `http://127.0.0.1:8000/docs/`

---

## Authentication
- Obtain JWT:
```
POST /api/auth/token/
{ "username": "<admin_or_user>", "password": "<password>" }
```
- Use: `Authorization: Bearer <access>`

> You may create users in Django Admin: `/admin/` (allowed per instructions).

---

## Endpoints (spec-matching)

### List all clients
```
GET /api/clients/
```
**Sample response**
```json
[
  { "id": 1, "client_name": "Nimap", "created_at": "2019-12-24T11:03:55.931739+05:30", "created_by": "Rohit" }
]
```

### Create a new client
```
POST /api/clients/
{ "client_name": "company A" }
```
**Response**
```json
{ "id": 3, "client_name": "company A", "created_at": "...", "created_by": "Rohit" }
```

### Retrieve client (with projects)
```
GET /api/clients/:id/
```
**Response**
```json
{
  "id": 2,
  "client_name": "Infotech",
  "projects": [ { "id": 1, "project_name": "project A" } ],
  "created_at": "...",
  "created_by": "Rohit",
  "updated_at": "..."
}
```

### Update client
```
PUT/PATCH /api/clients/:id/
{ "client_name": "company A" }
```
**Response**
```json
{
  "id": 3,
  "client_name": "company A",
  "created_at": "...",
  "created_by": "Rohit",
  "updated_at": "..."
}
```

### Delete client
```
DELETE /api/clients/:id/
```
**Response**: status 204 No Content

### Create a project for a client
```
POST /api/clients/:id/projects/
{
  "project_name": "Project A",
  "users": [ { "id": 1, "name": "Rohit" } ]   // name is ignored, id is required
}
```
**Response**
```json
{
  "id": 3,
  "project_name": "Project A",
  "client": "Nimap",
  "users": [ { "id": 1, "username": "Rohit", "first_name": "", "last_name": "" } ],
  "created_at": "...",
  "created_by": "Ganesh"
}
```

> `users` can be a list of objects with `id` or a list of integer IDs.

### List projects assigned to logged-in user
```
GET /api/projects/
```
**Response**
```json
[
  { "id": 1, "project_name": "Project A", "client": "Nimap", "users": [...], "created_at": "...", "created_by": "Ganesh" }
]
```

---

## Notes / Decisions
- **Users**: Create in Django Admin (allowed). You can also use the `/api/auth/token/` endpoints for JWT.
- **created_by**: Automatically set to the authenticated user on create for both `Client` and `Project`.
- **Timestamps**: `created_at`/`updated_at` are auto managed.
- **DB**: PostgreSQL only, configured via `.env`.
- **Docs**: Swagger UI at `/docs/`.

---

## Run DB Design
- The DB schema is defined by Django models in `core/models.py`.
- To inspect: `python manage.py makemigrations --dry-run --verbosity 3`.
- To generate SQL: `python manage.py sqlmigrate core 0001` (after first migration exists).

---

## Tests (optional quick sanity)
Create at least one user and token, then hit endpoints:
```bash
# Create client
curl -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json"  -d '{"client_name":"Nimap"}' http://127.0.0.1:8000/api/clients/
```

---

## Project Structure
```
.
├── core/
│   ├── admin.py
│   ├── apps.py
│   ├── migrations/
│   ├── models.py
│   ├── serializers.py
│   ├── urls.py
│   └── views.py
├── nimap_api/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
├── requirements.txt
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## MySQL (alternative)
If you must use MySQL, set env:
```
DB_ENGINE=django.db.backends.mysql
DB_NAME=nimapdb
DB_USER=nimapuser
DB_PASSWORD=nimappass
DB_HOST=localhost
DB_PORT=3306
```
and add `mysqlclient` to `requirements.txt` then `pip install -r requirements.txt`.
```
pip install mysqlclient==2.2.4
```
