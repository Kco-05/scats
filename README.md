# SCATS

**Student Character and Achievement Tracking System**

SCATS is a Rails application for institutes to record verified student achievements and conduct outcomes as a durable character ledger. Students (or authorised faculty on their behalf) submit requests with proof; configurable review chains verify them; approved outcomes contribute signed points to each student’s record — positive for achievements, negative for conduct. The student and faculty UIs show this as a 0–10 **SCATS Score**.

> Active development. Schema, permissions, and UI may still evolve.

---

## Features

- **Path A** — student submits a request with proof  
- **Path B** — eligible faculty raise a request on a student’s behalf  
- **Configurable review chains** — shared hierarchy templates (division / sub-division) with ordered review roles; staffing via role assignments  
- **Multi-step advance / revert / reject** — live chain resolution; unstaffed roles are skipped  
- **Hierarchy change safety** — in-flight requests are remapped onto the new staffed chain when templates change or owners are reattached (with history + notifications)  
- **SCATS Score** — 0–10 sigmoid of net approved points (`k` is admin-tunable); points are snapshotted at final approval  
- **Integrity Index** — student dashboard donut of achievement vs conduct points; 0/0 and “No data” when nothing is approved yet  
- **Admin console** — users (CSV import), departments, divisions / sub-divisions / categories (archive), review roles, role assignments, hierarchy templates, reason templates, settings  
- **Role permissions** — profile self-edit toggles; per review-role permission to create/import students  
- **Faculty student create / CSV import** — gated by review-role flags and live assignments  
- **Notifications & email** — in-app student notifications on approval; Action Mailer for review events (works in development with `.env`)  
- **Unsaved-changes guard** — leave confirm (Stay / Discard / Save & exit) on save surfaces  

---

## Tech stack

| Layer | Choice |
| --- | --- |
| Framework | Ruby on Rails ~> 8.1 |
| Language | Ruby 3.4.8 (see `.ruby-version`) |
| Database | PostgreSQL 16 |
| Auth | Devise (no self-registration; admin-provisioned accounts) |
| Authorization | Pundit |
| Frontend | Hotwire (Turbo + Stimulus), Tailwind CSS, Importmap |
| Jobs / cache | Solid Queue, Solid Cache (primary DB) |
| Uploads | Active Storage |
| Tests | RSpec, Capybara |

---

## Roles and review model

`User.role` is one of **`admin`**, **`faculty`**, or **`student`**.

**Dean**, **Supervisor**, and other titles are **`ReviewRole`s**, not separate user roles. Faculty hold them through **`RoleAssignment`s** on a division or sub-division. Chains are defined by **hierarchy templates** attached to those owners.

| Actor | Capabilities |
| --- | --- |
| **Admin** | Org structure, users/imports, review roles, hierarchies, assignments, reason templates, score scale, role permissions. Does not browse student character scores. |
| **Faculty** | Student directory and SCATS Score breakdowns; review queues when assigned; optional student create/import when their review role allows it. |
| **Student** | Submit / resubmit requests; view own SCATS Score, Integrity Index, timeline, and notifications. |

Default seeded Technical / Coding demo chain:

`Supervisor → Coordinator → Division Reviewer → Dean`

Final points are recorded only when the last staffed step approves.

---

## Prerequisites

- Ruby **3.4.8** (rbenv or equivalent)  
- Bundler  
- Docker Desktop (Postgres 16 via Compose)  
- Node is **not** required for day-to-day JS (Importmap); Tailwind is built via `tailwindcss-rails`  

On Windows, development is typically done in **WSL2** with Docker Desktop integration.

---

## Local setup

```bash
git clone <your-fork-or-remote> scats
cd scats

bundle install

# Start PostgreSQL (user/password/db: scats / scats / scats_development)
docker compose up -d

bin/rails db:prepare   # create + migrate
bin/rails db:seed      # idempotent demo data

bin/rails server
```

Open [http://localhost:3000](http://localhost:3000).

### Demo logins (from seeds)

Printed again at the end of `db:seed`. Defaults:

| Role | Email | Password |
| --- | --- | --- |
| Admin | `admin@scats.edu` | `admin123` |
| Faculty / others | e.g. `meera.nair@scats.edu`, `kavya.shetty@scats.edu`, `asha.kumar@scats.edu` | `password123` |

`db:seed` is safe to re-run; it does not duplicate structure or re-create requests for students who already have them.

### Tailwind

After changing Tailwind sources:

```bash
bin/rails tailwindcss:build
```

### Stop local Postgres

```bash
docker compose stop
```

---

## Configuration

### Environment

Copy [`.env.example`](.env.example) to `.env` for optional local mail (never commit `.env`):

```bash
cp .env.example .env
```

| Variable | Purpose |
| --- | --- |
| `GMAIL_USERNAME` / `GMAIL_APP_PASSWORD` | Development SMTP (`config/environments/development.rb`) |

Development uses Compose Postgres (`config/database.yml`). Do not commit `.env` or secrets.

---

## Tests

```bash
bundle exec rspec
```

Focused examples:

```bash
bundle exec rspec spec/services/hierarchy_bulk_save_spec.rb
bundle exec rspec spec/services/in_flight_request_remapper_spec.rb
bundle exec rspec spec/requests/faculty/student_create_spec.rb
```

---

## Repository layout (high level)

```text
app/
  controllers/     # admin, students, supervisors, deans, faculties, …
  models/          # User, AchievementRequest, Hierarchy, ReviewRole, …
  services/        # ReviewChainResolver, HierarchyBulkSave, InFlightRequestRemapper, …
  javascript/      # Stimulus controllers (hierarchies, leave-guard, …)
  views/
config/
db/migrate/ db/seeds.rb
spec/
docker-compose.yml # Local Postgres 16
```

---

## License / ownership

Internal institute project unless otherwise stated by the maintainers. Ask before redistributing.
