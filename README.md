# Bulk Certificate Generator

A scalable backend service for generating and managing certificates in bulk.

The application accepts a certificate-generation request containing multiple recipients, processes certificates asynchronously through background workers, stores certificate metadata in PostgreSQL, and stores generated PDF files in Amazon S3.

## Features

- Bulk certificate generation from a single request.
- FastAPI REST API.
- PostgreSQL for jobs, recipients, and certificate metadata.
- Redis-backed background job processing.
- Asynchronous certificate generation.
- Amazon S3 storage for generated certificates.
- Individual certificate status tracking.
- Job-level progress tracking.
- Failure isolation between certificates.
- Certificate retrieval through an API endpoint.
- Automatic API documentation through OpenAPI.
- Automated tests.
- Docker-based local development support.

## Architecture

```text
Client
  |
  | HTTP request
  v
FastAPI API
  |
  | Create job and certificate records
  v
PostgreSQL
  |
  | Publish background task
  v
Redis
  |
  | Queue
  v
Background Worker
  |
  | Generate certificate PDF
  v
Amazon S3
  |
  | Store generated file
  v
Certificate download API
```

The API does not wait for every certificate to finish. It creates a generation job, places the work on the background queue, and immediately returns a job identifier.

## Technology Stack

- Python 3.12+
- FastAPI
- PostgreSQL
- SQLAlchemy
- Pydantic
- Redis
- Background worker system
- ReportLab or the PDF generation library used by the project
- Amazon S3
- Pytest
- Docker and Docker Compose

## Project Structure

The exact structure may vary slightly depending on the implementation, but the main components are organized approximately as follows:

```text
bulk-certificate-generator/
├── app/
│   ├── api/
│   │   └── routes/
│   ├── core/
│   ├── models/
│   ├── schemas/
│   ├── services/
│   ├── workers/
│   └── main.py
├── tests/
├── migrations/
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .env.example
└── README.md
```

## Prerequisites

Install the following tools:

- Python 3.12 or newer
- Docker Desktop
- Docker Compose
- Git
- An AWS account with an S3 bucket, or an S3-compatible local service for development

Verify the installations:

```bash
python --version
docker --version
docker compose version
git --version
```

## Clone the Repository

```bash
git clone [https://github.com/Mukul9112003/bulk-certificate-generator.git](https://github.com/Mukul9112003/bulk-certificate-generator.git)
cd bulk-certificate-generator
```

## Create a Virtual Environment

### Linux and macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Windows PowerShell

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
```

### Windows Command Prompt

```cmd
py -m venv .venv
.venv\Scripts\activate.bat
```

## Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

If the repository uses a different dependency file, install from that file instead:

```bash
pip install -r requirements-dev.txt
```

## Environment Configuration

Create a local environment file:

```bash
cp .env.example .env
```

On Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

Update `.env` with your local configuration.

Example:

```env
APP_NAME=Bulk Certificate Generator
APP_ENV=development
DEBUG=true

DATABASE_URL=postgresql+psycopg://certificate_user:certificate_password@localhost:5432/certificate_db

REDIS_URL=redis://localhost:6379/0

AWS_ACCESS_KEY_ID=your-access-key-id
AWS_SECRET_ACCESS_KEY=your-secret-access-key
AWS_REGION=ap-south-1
AWS_S3_BUCKET=your-certificate-bucket
AWS_S3_PREFIX=certificates

CERTIFICATE_TEMPLATE_PATH=templates/certificate_template.pdf

WORKER_QUEUE_NAME=certificate-generation
```

Do not commit `.env` to Git.

The `.gitignore` file should contain:

```gitignore
.env
.venv/
__pycache__/
.pytest_cache/
generated_certificates/
*.pyc
```

## Required Environment Variables

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `REDIS_URL` | Redis connection string |
| `AWS_ACCESS_KEY_ID` | AWS access key used for S3 |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key used for S3 |
| `AWS_REGION` | AWS region containing the S3 bucket |
| `AWS_S3_BUCKET` | S3 bucket used for generated certificates |
| `AWS_S3_PREFIX` | Optional folder-like prefix for S3 objects |
| `CERTIFICATE_TEMPLATE_PATH` | Path to the certificate template |
| `WORKER_QUEUE_NAME` | Name of the background queue |

Use an IAM user or role with only the permissions required by the application.

A minimum S3 policy normally needs permissions similar to:

```text
s3:PutObject
s3:GetObject
s3:ListBucket
```

Avoid granting full administrative access to the application.

## Start Local Infrastructure

If the repository contains Docker Compose configuration, start the local services:

```bash
docker compose up -d postgres redis
```

Check the running containers:

```bash
docker compose ps
```

View service logs:

```bash
docker compose logs -f postgres
docker compose logs -f redis
```

If the Compose file uses different service names, list them first:

```bash
docker compose config --services
```

Then start the correct services.

## Database Setup

Run the database migrations before starting the API.

For Alembic:

```bash
alembic upgrade head
```

If the project provides a custom migration command, use the command documented in the source code.

To create a new migration after modifying database models:

```bash
alembic revision --autogenerate -m "describe the database change"
alembic upgrade head
```

Verify the current migration:

```bash
alembic current
```

## Run the API Locally

The common FastAPI command is:

```bash
uvicorn app.main:app --reload
```

If the application object is located elsewhere, use the correct module path. For example:

```bash
uvicorn main:app --reload
```

The API should then be available at:

```text
http://127.0.0.1:8000
```

Open the interactive API documentation:

```text
http://127.0.0.1:8000/docs
```

Open the alternative documentation:

```text
http://127.0.0.1:8000/redoc
```

## Run the Background Worker

The background worker must run separately from the FastAPI process.

Use the worker command defined by the project. Common examples include:

```bash
python -m app.workers.worker
```

or:

```bash
celery -A app.worker.celery_app worker --loglevel=info
```

or:

```bash
rq worker certificate-generation
```

The exact command depends on the worker library used by the repository.

A complete local setup normally requires these processes:

```text
Terminal 1: PostgreSQL and Redis
Terminal 2: FastAPI application
Terminal 3: Background worker
```

Example:

```bash
# Terminal 1
docker compose up -d postgres redis

# Terminal 2
uvicorn app.main:app --reload

# Terminal 3
python -m app.workers.worker
```

## Run Everything with Docker Compose

If the repository includes API and worker services in `docker-compose.yml`, run:

```bash
docker compose up --build
```

Run in detached mode:

```bash
docker compose up --build -d
```

Check logs:

```bash
docker compose logs -f api
docker compose logs -f worker
```

Stop the services:

```bash
docker compose down
```

Stop the services and remove database volumes:

```bash
docker compose down -v
```

The `-v` option deletes local database data, so use it carefully.

## Health Check

If the project provides a health endpoint, test it with:

```bash
curl http://127.0.0.1:8000/health
```

Expected response:

```json
{
  "status": "healthy"
}
```

If the project uses a different endpoint, check the OpenAPI documentation at `/docs`.

## Certificate Generation Workflow

The certificate-generation process follows these steps:

```text
1. Client submits a request with event and recipient information.
2. API validates the request.
3. API creates a generation job.
4. API creates one certificate record per recipient.
5. API publishes a background task.
6. API returns the job ID.
7. Worker reads the job from the queue.
8. Worker generates certificates independently.
9. Worker uploads generated files to S3.
10. Worker updates certificate and job statuses.
11. Client checks job progress.
12. Client retrieves successful certificates.
```

## Submit a Certificate Generation Request

The exact endpoint and field names must match the routes implemented in the repository. A typical endpoint is:

```text
POST /api/v1/certificate-jobs
```

Example request:

```bash
curl -X POST http://127.0.0.1:8000/api/v1/certificate-jobs \
  -H "Content-Type: application/json" \
  -d '{
    "event_name": "Python Workshop 2026",
    "event_date": "2026-10-05",
    "issuer_name": "ABC Institute",
    "recipients": [
      {
        "name": "Rahul Sharma",
        "email": "rahul@example.com",
        "certificate_id": "CERT-001"
      },
      {
        "name": "Priya Reddy",
        "email": "priya@example.com",
        "certificate_id": "CERT-002"
      }
    ]
  }'
```

Example response:

```json
{
  "job_id": "2f2d99d2-1a52-4f5a-82f7-4122ecb1f2d2",
  "status": "PENDING",
  "total": 2,
  "message": "Certificate generation job created"
}
```

The API returns before the PDF files are generated. The client should save the returned `job_id`.

### Windows PowerShell Request

```powershell
$body = @{
    event_name = "Python Workshop 2026"
    event_date = "2026-10-05"
    issuer_name = "ABC Institute"
    recipients = @(
        @{
            name = "Rahul Sharma"
            email = "rahul@example.com"
            certificate_id = "CERT-001"
        },
        @{
            name = "Priya Reddy"
            email = "priya@example.com"
            certificate_id = "CERT-002"
        }
    )
} | ConvertTo-Json -Depth 5

Invoke-RestMethod `
    -Method Post `
    -Uri "http://127.0.0.1:8000/api/v1/certificate-jobs" `
    -ContentType "application/json" `
    -Body $body
```

## Request Validation

The API should validate at least the following:

- Event name is required.
- Event date must be valid.
- Issuer name is required.
- Recipient name is required.
- Recipient email must be valid.
- Certificate ID is required.
- Certificate IDs must be unique within a request.
- Certificate IDs should not conflict with existing records.

Example validation error:

```json
{
  "error": "ValidationError",
  "message": "Request validation failed",
  "details": [
    {
      "field": "recipients.0.email",
      "message": "Invalid email address"
    }
  ]
}
```

Invalid requests should not create partial jobs.

## Check Job Status

Use the job ID returned from the creation request:

```bash
curl http://127.0.0.1:8000/api/v1/certificate-jobs/2f2d99d2-1a52-4f5a-82f7-4122ecb1f2d2
```

Example response while processing:

```json
{
  "job_id": "2f2d99d2-1a52-4f5a-82f7-4122ecb1f2d2",
  "status": "PROCESSING",
  "total": 2,
  "successful": 1,
  "failed": 0,
  "progress_percentage": 50,
  "certificates": [
    {
      "certificate_id": "CERT-001",
      "recipient_name": "Rahul Sharma",
      "status": "SUCCESS",
      "download_url": "/api/v1/certificates/CERT-001",
      "error": null
    },
    {
      "certificate_id": "CERT-002",
      "recipient_name": "Priya Reddy",
      "status": "PROCESSING",
      "download_url": null,
      "error": null
    }
  ]
}
```

Example response after successful completion:

```json
{
  "job_id": "2f2d99d2-1a52-4f5a-82f7-4122ecb1f2d2",
  "status": "COMPLETED",
  "total": 2,
  "successful": 2,
  "failed": 0,
  "progress_percentage": 100,
  "certificates": [
    {
      "certificate_id": "CERT-001",
      "recipient_name": "Rahul Sharma",
      "status": "SUCCESS",
      "download_url": "/api/v1/certificates/CERT-001",
      "error": null
    },
    {
      "certificate_id": "CERT-002",
      "recipient_name": "Priya Reddy",
      "status": "SUCCESS",
      "download_url": "/api/v1/certificates/CERT-002",
      "error": null
    }
  ]
}
```

## Job Status Values

Typical job states are:

| Status | Meaning |
|---|---|
| `PENDING` | Job has been created but processing has not started |
| `PROCESSING` | Worker is processing certificates |
| `COMPLETED` | All certificates were generated successfully |
| `COMPLETED_WITH_ERRORS` | Some certificates succeeded and some failed |
| `FAILED` | The job could not complete successfully |

Typical certificate states are:

| Status | Meaning |
|---|---|
| `PENDING` | Certificate is waiting for processing |
| `PROCESSING` | Certificate is currently being generated |
| `SUCCESS` | PDF was generated and stored successfully |
| `FAILED` | Certificate generation or storage failed |

## Retrieve a Generated Certificate

After a certificate reaches `SUCCESS`, retrieve it using the certificate ID:

```bash
curl http://127.0.0.1:8000/api/v1/certificates/CERT-001 \
  -o CERT-001.pdf
```

Open the downloaded file using a PDF viewer.

### Windows PowerShell

```powershell
Invoke-WebRequest `
    -Uri "http://127.0.0.1:8000/api/v1/certificates/CERT-001" `
    -OutFile "CERT-001.pdf"
```

Depending on the implementation, the endpoint may:

1. Stream the PDF directly.
2. Redirect to a temporary S3 signed URL.
3. Return JSON containing a signed download URL.

Example signed URL response:

```json
{
  "certificate_id": "CERT-001",
  "download_url": "[https://s3.amazonaws.com/](https://s3.amazonaws.com/)..."
}
```

Do not attempt to download a certificate before its status is `SUCCESS`.

## Failed Certificate Behavior

Each certificate is processed independently.

For example:

```text
Certificate A → SUCCESS
Certificate B → FAILED
Certificate C → SUCCESS
```

The worker must continue processing Certificate C even if Certificate B fails.

The failed record should contain an error message:

```json
{
  "certificate_id": "CERT-002",
  "status": "FAILED",
  "error": "Unable to generate or upload certificate"
}
```

A failed certificate should not provide a valid download URL.

## Running Tests

Install development dependencies if required:

```bash
pip install pytest pytest-asyncio httpx
```

Run the complete test suite:

```bash
pytest
```

Run with detailed output:

```bash
pytest -v
```

Run a specific test file:

```bash
pytest tests/test_jobs.py -v
```

Run a specific test:

```bash
pytest tests/test_jobs.py::test_create_certificate_job -v
```

Run tests with coverage:

```bash
pip install pytest-cov
pytest --cov=app --cov-report=term-missing
```

The test suite should cover:

- Valid job creation.
- Missing required fields.
- Invalid email addresses.
- Duplicate certificate IDs.
- Job status retrieval.
- Certificate generation.
- Successful certificate retrieval.
- Missing certificate retrieval.
- Failed certificate download rejection.
- Individual failure handling.
- Job progress calculation.
- Worker retry or failure behavior where applicable.

## Database Design

The application uses a relational model containing at least two main entities.
### Certificate Record

A certificate record represents one recipient inside a job.

Typical fields include:

```text
id
job_id
certificate_id
recipient_name
recipient_email
status
storage_key
download_url
error_message
created_at
generated_at
```

Relationship:

```text
One generation job
    |
    └── many certificate records
```

Recommended database constraints:

- `job_id` must reference an existing job.
- `certificate_id` should be unique.
- Certificate status should use a controlled set of values.
- Counts should not be negative.
- Indexes should exist on `job_id`, `certificate_id`, and status fields where appropriate.

## Important Design Decisions

### Asynchronous processing

Certificate generation is performed in a background worker instead of inside the HTTP request.

This prevents the client from waiting for a large batch to finish and allows multiple jobs to be processed independently.

### Redis as a message broker

Redis is used to transfer background work from the API process to worker processes.

The API creates the job and publishes a task. The worker consumes that task and performs certificate generation.

### PostgreSQL as the system of record

PostgreSQL stores the authoritative state of jobs and certificates.

The database allows the application to:

- Track progress.
- Query failed certificates.
- Prevent duplicate certificate IDs.
- Recover metadata independently from file storage.
- Provide consistent API responses.

### Amazon S3 for file storage

Generated certificates are stored in S3 instead of only on the local filesystem.

This is useful because:

- API and worker containers do not need to share local files.
- Files remain available after container restarts.
- S3 is suitable for horizontal scaling.
- Signed URLs can provide temporary downloads.

### Per-certificate failure isolation

Each recipient is processed independently.

A failure for one recipient is saved in that certificate record, and processing continues for the remaining recipients.

### Job-level and certificate-level status

The application maintains both:

```text
Job status
Certificate status
```

This makes it possible to answer both:

- “Is the complete batch finished?”
- “What happened to this individual certificate?”

### Idempotency and duplicate protection

Certificate IDs should be unique.

A database unique constraint is more reliable than checking only in application code because concurrent requests could otherwise create duplicate records.

For production use, the API should also support an idempotency key for job creation.

### API returns immediately

The creation endpoint should return `202 Accepted` because processing continues asynchronously.

The response contains the job ID, which the client uses to poll the job-status endpoint.

### Error handling

The API should return safe error responses without exposing internal stack traces.

Detailed exceptions should be logged on the server, while clients should receive a concise message.

## Security Considerations

- Do not commit `.env` files.
- Do not hard-code AWS keys.
- Use IAM roles where possible.
- Give the S3 user minimum required permissions.
- Validate all request fields.
- Restrict certificate access if authentication is added.
- Avoid exposing internal exception traces.
- Use HTTPS in production.
- Use signed S3 URLs with short expiration times.
- Validate file names before using them as storage keys.
- Add rate limiting for public endpoints.
- Add authentication and authorization before production deployment.

## Production Considerations

The current architecture is suitable for a small or medium workload.

For very large workloads, consider:

- Multiple worker replicas.
- Durable task queues.
- Worker retry policies.
- Dead-letter queues.
- S3 multipart uploads where appropriate.
- Database connection pooling.
- Batch inserts.
- Job pagination.
- Structured logging.
- Metrics and tracing.
- Health checks for API, PostgreSQL, Redis, and S3.
- Job cancellation.
- Idempotent task processing.
- Monitoring for jobs stuck in `PROCESSING`.

## Troubleshooting

### API cannot connect to PostgreSQL

Check that PostgreSQL is running:

```bash
docker compose ps
```

Check the database URL:

```env
DATABASE_URL=postgresql+psycopg://certificate_user:certificate_password@localhost:5432/certificate_db
```

Run migrations:

```bash
alembic upgrade head
```

### Worker does not process jobs

Check that Redis is running:

```bash
docker compose ps
```
Check the worker logs:

```bash
docker compose logs -f worker
```

If running the worker locally, make sure the virtual environment is active and the `REDIS_URL` value is correct.

### Certificate is stuck in `PENDING`

Possible causes:

- Worker is not running.
- Redis connection is unavailable.
- The queue name does not match.
- The task was not published successfully.
- The worker is listening to a different queue.

### Certificate is stuck in `PROCESSING`

Possible causes:

- The worker crashed.
- A PDF-generation operation raised an unhandled exception.
- An S3 upload timed out.
- The status update transaction was not committed.

Production systems should include a recovery process for stale `PROCESSING` records.

### S3 upload fails

Check:

- AWS credentials.
- AWS region.
- S3 bucket name.
- Bucket permissions.
- Network connectivity.
- Object key generation.
- Bucket policy and IAM policy.

### Download returns `404`

Check:

- The certificate ID is correct.
- The certificate status is `SUCCESS`.
- The S3 object exists.
- The database storage key matches the S3 object key.

## API Documentation

When the API is running, open:

```text
http://127.0.0.1:8000/docs
```

or:

```text
http://127.0.0.1:8000/redoc
```

The documentation provides:

- Request schemas.
- Response schemas.
- Endpoint descriptions.
- Available HTTP status codes.
- Interactive API testing.

## Example Complete Workflow

### 1. Start infrastructure

```bash
docker compose up -d postgres redis
```
### 3. Start the API

```bash
uvicorn app.main:app --reload
```

### 4. Start the worker

```bash
python -m app.workers.worker
```

### 5. Submit a job

```bash
curl -X POST http://127.0.0.1:8000/api/v1/certificate-jobs \
  -H "Content-Type: application/json" \
  -d '{
    "event_name": "Python Workshop 2026",
    "event_date": "2026-10-05",
    "issuer_name": "ABC Institute",
    "recipients": [
      {
        "name": "Rahul Sharma",
        "email": "rahul@example.com",
        "certificate_id": "CERT-001"
      }
    ]
  }'
```

### 6. Check the job

```bash
curl http://127.0.0.1:8000/api/v1/certificate-jobs/<JOB_ID>
```

### 7. Download the certificate

```bash
curl http://127.0.0.1:8000/api/v1/certificates/CERT-001 \
  -o CERT-001.pdf
```
