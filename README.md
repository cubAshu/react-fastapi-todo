# React + FastAPI + PostgreSQL (3-container demo)

Full-stack sample app with:

| Service    | Tech                         | Port (host) |
| ---------- | ---------------------------- | ----------- |
| Frontend   | React (Vite build) + nginx   | **3000**    |
| Backend    | Python FastAPI + Uvicorn     | **8000**    |
| Database   | PostgreSQL 16                | **5432**    |

The UI is a small **Todo** app that creates, lists, toggles, and deletes items in Postgres through the API.

---

## Project layout

```
reactapp/
├── frontend/          # React (Vite) + nginx
├── backend/           # FastAPI + SQLAlchemy
├── docker-compose.yml # 3 services: frontend, backend, db
├── k8s/               # Kubernetes Pods/Services/Ingress (not applied by default)
├── .env.example
└── README.md
```

- Architecture, reverse proxy, and request flow: [`docs/architecture.md`](docs/architecture.md)
- Kubernetes manifests (optional): [`k8s/README.md`](k8s/README.md)

---

## Quick start with Docker / Podman Compose (recommended)

From the repo root:

```bash
# Docker
docker compose up --build

# or Podman
podman compose up --build
# (or) podman-compose up --build
```

Then open:

- **App UI:** http://localhost:3000
- **API docs:** http://localhost:8000/docs
- **Health:** http://localhost:8000/api/health

Stop:

```bash
docker compose down
# remove DB volume too:
docker compose down -v
```

---

## Run locally (without full stack containers)

You still need **PostgreSQL** (container or install). Easiest: run only the DB container, then backend + frontend on the host.

### 1. Database

```bash
docker run --name reactapp-db \
  -e POSTGRES_USER=todos \
  -e POSTGRES_PASSWORD=todos \
  -e POSTGRES_DB=todos \
  -p 5432:5432 \
  -d postgres:16-alpine
```

### 2. Backend

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
export DATABASE_URL=postgresql://todos:todos@localhost:5432/todos
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

API: http://localhost:8000/docs

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

UI: http://localhost:5173  
Vite proxies `/api` → `http://localhost:8000`.

---

## API overview

| Method | Path                | Description      |
| ------ | ------------------- | ---------------- |
| GET    | `/api/health`       | Health + DB ping |
| GET    | `/api/todos`        | List todos       |
| POST   | `/api/todos`        | Create `{title}` |
| PATCH  | `/api/todos/{id}`   | Update todo      |
| DELETE | `/api/todos/{id}`   | Delete todo      |

---

## Environment

| Variable        | Where    | Default                                              |
| --------------- | -------- | ---------------------------------------------------- |
| `DATABASE_URL`  | backend  | `postgresql://todos:todos@localhost:5432/todos`      |
| `VITE_API_URL`  | frontend | empty (same-origin `/api` via proxy or nginx)        |

See `.env.example`.

---

## Notes

- Compose networking: **nginx** in the frontend container proxies `/api` to the `backend` service; browser only needs port **3000**.
- **Vite** is only used at *image build time* to compile React. At runtime, nginx serves `dist/` and proxies API calls (see `frontend/nginx.conf`).
- Default DB credentials are for **demo only** — change them before any real deployment.
- Backend auto-creates the `todos` table on startup.
- Stack verified with **rootless Podman** + `podman compose` (Docker Compose v2 plugin works the same).
