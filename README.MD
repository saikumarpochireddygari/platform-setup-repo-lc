## Airflow + MLflow + Jenkins Dev Stack

This repository contains a self-contained Docker Compose environment for running an end-to-end MLOps playground. The stack bundles:

- **Postgres 13** – shared metadata DB for Airflow plus a dedicated `mlflow` DB.
- **Apache Airflow 2.9 (Python 3.11)** – webserver + scheduler built from `airflow/Dockerfile` with extra ML libraries.
- **MLflow Tracking Server** – served from a lightweight Python image and backed by Postgres + MinIO (S3-compatible artifact store).
- **MinIO** – local S3-compatible bucket for MLflow artifacts.
- **Jenkins LTS** – optional CI agent with Python tooling preinstalled.

Use it to prototype workflows, demo pipelines, or validate Airflow DAGs that interact with MLflow and object storage.

---

### Prerequisites

- Docker Engine 24+ and Docker Compose V2 (`docker compose` CLI).
- ~6 GB of disk for images and named volumes (`postgres_data`, `minio_data`, `airflow_logs`, `airflow_plugins`, `jenkins_home`).
- Ports `5432`, `8080`, `8081`, `9000-9001`, and `5001` available on the host.

---

### Quick Start

```bash
# 1. Clone the repo (or open it inside Codespaces/Dev Container)
git clone <this-repo> && cd into the repo folder

# 2. Build images (Airflow + Jenkins) and start everything
docker compose up --build

# 3. Tear down containers but keep the persistent volumes
docker compose down
```

The `airflow-init` and `postgres-mlflow-init` services are one-shot jobs that prepare databases. They run automatically the first time (and re-run safely when needed).

---

### Accessing Services

| Service | URL | Notes / Credentials |
| --- | --- | --- |
| Airflow Webserver | `http://localhost:8080` | User: `admin` / Pass: `admin` |
| MLflow UI | `http://localhost:5001` | Artifacts stored in MinIO bucket `mlflow-artifacts` |
| MinIO Console | `http://localhost:9001` | User: `minioadmin` / Pass: `minioadmin` |
| MinIO S3 Endpoint | `http://localhost:9000` | Configure clients with the same access/secret keys |
| Jenkins | `http://localhost:8081/jenkins` | Initial admin password lives in `jenkins_home/secrets/initialAdminPassword` volume |
| Postgres | `localhost:5432` | User: `airflow`, Pass: `airflow`, DBs: `airflow`, `mlflow` |

Airflow and MLflow containers come pre-configured to talk to Postgres and MinIO via the environment variables defined in `docker-compose.yml`.

---

### Repository Layout

```
.
├── airflow/
│   ├── Dockerfile         # Extends apache/airflow with ML/observability deps
│   └── dags/              # Mount point for your DAG files (empty by default)
├── jenkins/
│   └── Dockerfile         # Installs Python + git inside Jenkins LTS image
├── docker-compose.yml     # Orchestrates all services + one-shot init jobs
└── README.MD
```

Drop your DAGs into `airflow/dags/` and they will hot-reload inside the scheduler/webserver containers.

---

### Common Tasks

#### Rebuilding only Airflow or Jenkins
```bash
docker compose build airflow-webserver airflow-scheduler airflow-init
docker compose build jenkins
```

#### Resetting Everything
```bash
docker compose down -v   # removes containers + **all** named volumes
```

#### Viewing Logs
```bash
docker compose logs -f airflow-scheduler
docker compose logs -f mlflow
```

---

### Jenkins Integration Tips

- Jenkins home is persisted in the `jenkins_home` volume; configure jobs once and they survive restarts.
- The Airflow DAG folder is mounted read-only into `/shared/airflow_dags` so Jenkins jobs can lint/test DAG code.
- The repository root is mounted read-only at `/workspace` inside Jenkins for pipeline scripts.

---

### Airflow Credentials & Fernet Key

The Airflow image uses a fixed Fernet key defined in `docker-compose.yml`. Update `AIRFLOW__CORE__FERNET_KEY` and user credentials before deploying to a shared environment.

To create more Airflow users:
```bash
docker compose run --rm airflow-webserver \
  airflow users create --username <user> --password <pass> \
  --firstname <fn> --lastname <ln> --role Admin --email <email>
```

---

### Troubleshooting

- **Containers exit immediately** – ensure Docker Desktop has enough memory (4+ GB). Restart the stack with `docker compose up`.
- **Ports already in use** – adjust the host-side ports in `docker-compose.yml`.
- **Airflow cannot reach Postgres** – the init container waits for Postgres, but if you see repeated failures, wipe the volumes with `docker compose down -v`.
- **MLflow artifact upload fails** – confirm MinIO is running and the `AWS_*` env vars are propagated to MLflow/Airflow containers.

---

### Extending the Stack

- Add extra Python dependencies for Airflow inside `airflow/Dockerfile`.
- Add DAG-specific environment variables directly inside `docker-compose.yml` or via `.env` + `${VAR}` entries.
- Add more Jenkins tooling via `jenkins/Dockerfile` (e.g., kubectl, docker CLI, etc.).

---

Happy shipping!
