# Bulk Certificate Generator

A scalable backend service that generates certificates in bulk using FastAPI, PostgreSQL, Redis, background workers, and Amazon S3.

## Features

- Bulk certificate generation through a REST API.
- Asynchronous PDF generation using background workers.
- PostgreSQL storage for jobs and certificate metadata.
- Redis-backed task queue.
- Amazon S3 storage for generated PDFs.
- Individual certificate status and job progress tracking.
- Failure isolation for individual certificates.
- Automatic API documentation with Swagger UI.
- Automated tests and Docker-based development.

## Architecture

Client → FastAPI → PostgreSQL
                  ↓
                 Redis → Background Worker
                               ↓
                         Generate PDF
                               ↓
                          Amazon S3

## Technology Stack

- Python 3.12+
- FastAPI, SQLAlchemy, Pydantic
- PostgreSQL
- Redis and background workers
- ReportLab (PDF generation)
- Amazon S3
- Pytest
- Docker and Docker Compose

## Prerequisites

Install Python 3.12+, Git, Docker Desktop, and Docker Compose. Configure an AWS S3 bucket or an S3-compatible storage service if required by your implementation.

## Installation

Clone the repository:

git clone https://github.com/pvtlabsest/certificate_initiator.git
cd certificate_initiator

Create and activate a virtual environment on Windows:

py -m venv .venv
.venv\Scripts\Activate.ps1

Install dependencies:
pip install --upgrade pip
pip install -r requirements.txt


## Environment Configuration

Create a local `.env` file from the example:

Copy-Item .env.example .env
Configure the database, Redis, AWS credentials, S3 bucket, and other required settings in `.env`. Never commit real credentials to GitHub.

## Run the Application

Start the required infrastructure if Docker Compose is configured:

```bash
docker compose up -d
```

Run database migrations if the project uses Alembic:

```bash
alembic upgrade head
```

Start the API:

```bash
uvicorn app.main:app --reload
```

Start the background worker in a separate terminal using the command configured by the project.

API documentation:

- Swagger UI: http://127.0.0.1:8000/docs
- ReDoc: http://127.0.0.1:8000/redoc

## API Workflow

1. Submit a bulk certificate-generation request.
2. Receive a job ID.
3. Check job progress and individual certificate statuses.
4. Download successfully generated certificates.

Typical endpoints (verify against the implementation):

```text
POST /api/v1/certificate-jobs
GET  /api/v1/certificate-jobs/{job_id}
GET  /api/v1/certificates/{certificate_id}
```

## Run Tests

```bash
pytest -v
```

## Security

- Keep `.env` and credentials out of version control.
- Use least-privilege AWS permissions.
- Validate incoming requests.
- Use temporary signed URLs for private S3 downloads.
- Configure authentication and HTTPS before production deployment.

## License

Add the appropriate license if this project is intended to be open source.
