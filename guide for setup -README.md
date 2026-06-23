# Bookstore API

A simple FastAPI app that talks to PostgreSQL. Deploys on Kubernetes.

## What it does

- `GET /` — welcome message
- `GET /book/?name=&author=&year=&limit=` — search books
- `GET /db_info/` — check DB connection

## Project structure

```
.
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── app/
│   ├── main.py          # FastAPI routes
│   ├── config.py        # reads env vars
│   └── db.py            # postgres queries
├── initdb/
│   └── initdb.sql       # seeds the bookstore schema + 10 books
└── k8s/
    └── deploy.yaml      # all k8s resources in one file
```

## Local dev (without Kubernetes)

```bash
docker-compose up -d
```

Then open:
- API: http://localhost:8000
- Adminer (DB GUI): http://localhost:8080

Adminer login: System=PostgreSQL, Server=db, User=postgres, Password=bookstorepass, DB=book

## Deploy to Kubernetes

```bash
# 1. build the image (point to minikube if using it)
eval $(minikube docker-env)
docker build -t bookstore-api:latest .

# 2. apply everything
kubectl apply -f k8s/deploy.yaml

# 3. check
kubectl -n bookstore get pods

# 4. access
kubectl -n bookstore port-forward svc/bookstore-api 8000:8000
kubectl -n bookstore port-forward svc/adminer 8080:8080
```

## Config

All settings live in env vars, pulled from ConfigMap/Secret on K8s:

| Variable | Default | Where |
|----------|---------|-------|
| POSTGRES_DB | book | ConfigMap |
| POSTGRES_HOST | bookstore-db | ConfigMap |
| POSTGRES_PORT | 5432 | ConfigMap |
| POSTGRES_USER | postgres | ConfigMap |
| POSTGRES_PASSWORD | bookstorepass | Secret |
| POSTGRES_SCHEMA | bookstore | ConfigMap |

To change anything: `kubectl edit configmap bookstore-config -n bookstore`

## How the DB gets seeded

The `initdb.sql` is baked into a ConfigMap and mounted at `/docker-entrypoint-initdb.d/`. Postgres runs it automatically on first boot — no custom DB image needed.
