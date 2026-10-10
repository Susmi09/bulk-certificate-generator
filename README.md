# Bulk Certificate Generator

A Python REST API that accepts a list of recipients, validates the data, generates one PDF certificate per valid recipient, tracks job status, and supports certificate downloads.

## Technology
- Python 3.11 or newer
- FastAPI
- SQLite + SQLAlchemy
- Pydantic validation
- ReportLab PDF generation
- Pytest

## Setup (Windows PowerShell)

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If PowerShell blocks activation, run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` in that terminal, then activate again.

## Run the application

```powershell
uvicorn app.main:app --reload
```

Open http://127.0.0.1:8000/docs to try the API interactively.

## Submit a bulk request

Use `POST /jobs` with JSON:

```json
{
  "event_name": "Python Workshop",
  "recipients": [
    {"name": "Anu", "email": "anu@example.com"},
    {"name": "Ravi", "email": "ravi@example.com"}
  ]
}
```

The response contains a `job_id`. The API accepts up to 500 recipients per request.

## Check job status and progress

Call `GET /jobs/{job_id}`. The response includes job status, total count, successful and failed counts, per-recipient statuses, and download URLs.

Possible job statuses:
- `queued`: waiting to start
- `processing`: certificates are being generated
- `completed`: all certificates succeeded
- `completed_with_errors`: some succeeded and some failed
- `failed`: all certificate generations failed

## List certificates

Call `GET /jobs/{job_id}/certificates`.

## Download a certificate

Call `GET /certificates/{certificate_id}/download`. The response is a PDF file.

Generated PDFs are stored in the `certificates/` directory. SQLite data is stored in `certificate_generator.db`.

## Run tests

```powershell
pytest -q
```

## Design decisions and limitations

- A single predefined PDF design is used; a template editor is outside the assignment scope.
- `POST /jobs` saves a job and its recipient records, then schedules generation as a FastAPI background task.
- Each recipient is processed independently. A failure for one certificate is recorded and does not stop later recipients.
- SQLite keeps setup simple for a local assignment. A production system with durable workloads should use a task queue (such as Celery/RQ), a production database, authentication, retention policies, and stronger file storage.
- FastAPI's in-process background tasks are suitable for a small demo but are not a durable distributed queue. A process restart can interrupt work.
- Email is validated but no email is sent; recipients retrieve PDFs through the API.
