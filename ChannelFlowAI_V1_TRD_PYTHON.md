# ChannelFlowAI V1 — Technical Requirements Document

**Version:** 1.1  
**Architecture:** Python Telegram automation with bot control plane + authorized Telegram session data plane  
**Runtime:** Python 3.12+

## 1. Technical Objective

Build a resilient Telegram-to-Telegram forwarding service with a bot-based control plane and an authorized Telegram user-session data plane.

The Python implementation must be production-oriented and must not use simulated authentication, fake forwarding, JSON-file persistence, or placeholder queue workers.

## 2. Recommended Python Stack

### Core
- Python 3.12+
- `aiogram 3.x` — Telegram Bot API / bot UI
- `Telethon` — MTProto client for authorized Telegram user-session operations
- `FastAPI` — optional internal/admin HTTP API and health endpoints where useful
- `Pydantic Settings` — configuration and environment validation

### Data
- PostgreSQL
- `SQLAlchemy 2.x` — ORM
- `Alembic` — database migrations
- `asyncpg` — PostgreSQL async driver

### Queue / Reliability
- Redis
- `arq` — async Redis job queue, or an equivalent Redis-backed Python queue
- Persistent job state in PostgreSQL
- Exponential retry/backoff

### Operations
- Docker / Docker Compose
- `structlog` or standard structured Python logging
- Sentry SDK or equivalent
- `pytest` + `pytest-asyncio`
- Ruff
- MyPy or Pyright where practical

The exact package versions should be pinned in `requirements.txt` or `pyproject.toml` after compatibility verification.

## 3. High-Level Architecture

`Telegram User → aiogram Bot → Application/Service Layer → PostgreSQL/Redis → Worker → Telethon Authorized Session → Source → Destination`

The bot handles control-plane interactions. Workers handle message processing and delivery.

Keep Telegram-specific code behind adapter/service interfaces so future providers can be added without rewriting the core domain.

## 4. Suggested Project Structure

```text
channel_flow_ai/
├── app/
│   ├── bot/
│   │   ├── handlers/
│   │   ├── keyboards/
│   │   ├── states/
│   │   └── middleware/
│   ├── telegram/
│   │   ├── bot_client.py
│   │   ├── user_client.py
│   │   ├── auth.py
│   │   ├── resolver.py
│   │   └── delivery.py
│   ├── domain/
│   │   ├── models.py
│   │   ├── enums.py
│   │   └── interfaces.py
│   ├── services/
│   │   ├── accounts.py
│   │   ├── tasks.py
│   │   ├── filters.py
│   │   ├── transforms.py
│   │   └── statistics.py
│   ├── db/
│   │   ├── models.py
│   │   ├── session.py
│   │   └── repositories/
│   ├── queue/
│   │   ├── jobs.py
│   │   ├── worker.py
│   │   └── scheduler.py
│   ├── security/
│   │   ├── encryption.py
│   │   └── secrets.py
│   ├── config.py
│   └── main.py
├── migrations/
├── tests/
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
├── requirements.txt
├── .env.example
└── README.md
```

Do not require FastAPI for the core bot loop if it adds unnecessary complexity; health/readiness endpoints can be a small separate service or integrated cleanly.

## 5. Domain Model

### User
- id
- telegram_user_id
- username
- language
- status
- timestamps

### TelegramAccount
- id
- user_id
- encrypted session reference
- account identity
- status
- timestamps

### ForwardingTask
- id
- user_id
- source reference
- destination reference
- mode
- delay
- status
- timestamps

### TaskFilter
- task_id
- type
- configuration
- enabled

### TaskTransform
- task_id
- type
- configuration
- priority

### DeliveryRecord
- task_id
- source_chat_id
- source_message_id
- destination_chat_id
- destination_message_id
- fingerprint
- status
- error
- timestamps

Use foreign keys, indexes and unique constraints appropriate for multi-user isolation and idempotency.

## 6. Security Requirements

- Never store OTP codes or Telegram passwords as plaintext.
- Do not log OTP codes, passwords, phone verification secrets or session strings.
- Encrypt Telegram session material at rest using an application encryption key stored outside PostgreSQL.
- Keep encryption keys outside the database and outside source control.
- Use environment variables / secret management for bot token, API ID, API hash, database URL, Redis URL and encryption key.
- Never hard-code secrets in Python source files.
- Validate and sanitize chat identifiers and transformation inputs.
- Disconnect must remove/revoke stored session material from the application side.
- Scope every account/task query by authenticated application user.
- Never expose another user's task/account data through callback data or API parameters.

## 7. Telegram Authentication

Use Telethon's real authorization flow.

Required states:

`START → PHONE → CODE → optional 2FA → CONNECTED`

Requirements:

- Send verification code through Telegram/Telethon.
- Accept the verification code from the user.
- Handle `SessionPasswordNeededError` for 2FA.
- Persist the resulting Telethon session securely after successful authentication.
- Never persist OTP or 2FA password after the authentication attempt.
- Support cancellation and reconnect.
- Handle expired codes, invalid codes, flood waits and authorization errors.
- Do not implement fake OTPs, fixed passwords, fake session IDs or simulated authentication.

## 8. Task Processing Pipeline

`Receive event → resolve matching tasks → validate task/account → message-type filter → keyword filters → idempotency fingerprint → transformations → enqueue → delay/rate limit → deliver → persist result → important notification`

Prefer event-driven processing from Telethon's NewMessage events.

Workers must load active tasks from PostgreSQL and enqueue only valid delivery jobs.

## 9. Forward / Copy Delivery

Implement real delivery through the authorized Telethon session.

### Forward
Use Telegram's native forwarding behavior where supported.

### Copy
Send/copy the content without the normal forwarded header where technically supported.

Support V1:

- Text
- Photo
- Video
- Document
- Audio
- Voice

Preserve captions where supported.

Do not claim support for protected content or inaccessible chats. Respect Telegram permissions and platform restrictions.

Album/media-group support is deferred unless testing demonstrates it can be safely included without destabilizing the core pipeline.

## 10. Reliability

- Redis-backed queue.
- PostgreSQL is authoritative for task and delivery state.
- Idempotency keys/fingerprints prevent duplicate delivery.
- Make idempotency atomic with a database unique constraint/transaction; never rely only on a check-then-insert sequence.
- Persist job state so worker restarts do not silently lose work.
- Bounded retries with exponential backoff.
- Handle Telegram `FloodWait` by delaying/requeueing jobs for the requested wait duration.
- Use failed-job/dead-letter handling for permanent failures.
- Gracefully recover workers after crashes.
- Deleted tasks must not process newly received events.
- Paused tasks must not enqueue new delivery jobs.

## 11. Account and Task State Machines

### Account

`DISCONNECTED → AUTHENTICATING → CONNECTED → EXPIRED/ERROR → RECONNECTING`

### Task

`DRAFT → ACTIVE ↔ PAUSED`

`ACTIVE → ATTENTION_REQUIRED / ERROR`

Deleted tasks must no longer participate in active processing.

## 12. Internal Service Actions

- `start_user(user_telegram_id)`
- `set_language(user_id, language)`
- `begin_account_auth(user_id)`
- `submit_phone(user_id, phone)`
- `submit_otp(user_id, code)`
- `submit_two_factor(user_id, password)`
- `disconnect_account(user_id)`
- `create_task(user_id, task_config)`
- `test_task(user_id, task_id)`
- `pause_task(user_id, task_id)`
- `resume_task(user_id, task_id)`
- `edit_task(user_id, task_id, patch)`
- `delete_task(user_id, task_id)`
- `get_tasks(user_id)`
- `get_task_stats(user_id)`

## 13. Observability

Track structured events for:

- Authentication
- Task creation
- Queueing
- Delivery success/failure
- Retries
- Rate limits
- Worker health

Never include OTPs, passwords or raw session credentials in logs.

## 14. Performance Targets

- Normal text forwarding should enter the delivery queue quickly after source receipt.
- Workers may process jobs concurrently while respecting Telegram limits.
- Active-task lookup must use indexed PostgreSQL queries.
- Worker restarts must not require users to recreate tasks.
- A single slow/failing task must not block unrelated users/tasks.

## 15. Testing

### Unit
- Filters
- Transformations
- Fingerprints
- State transitions
- Access-control checks

### Integration
- PostgreSQL
- Redis queue
- Telethon adapter boundaries
- Repository transactions

### Authentication
- Valid OTP
- Invalid OTP
- Expired OTP
- 2FA
- Cancellation
- Session persistence/reconnect

### Delivery
- Text
- Photo
- Video
- Document
- Audio
- Voice

### Reliability
- Retry
- FloodWait/rate limit
- Duplicate event
- Worker restart
- Destination failure
- Paused/deleted task
- Concurrent duplicate processing

### End-to-End
Use controlled Telegram accounts and chats.

## 16. Deployment

Start with a small Dockerized deployment:

- Bot/API service
- Worker service
- PostgreSQL
- Redis

Use:

- environment variables/secrets management
- automated database backups
- health checks
- restart policies
- persistent PostgreSQL storage
- structured logs

Kubernetes is not required for V1.

## 17. Python Developer Experience

The project must be runnable with standard Python tooling.

Minimum commands:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

python -m app.main
```

Recommended quality commands:

```bash
pytest
ruff check .
ruff format --check .
```

If using `pyproject.toml`, keep configuration centralized there.

## 18. Future-Proofing

Keep Telegram-specific adapters behind interfaces so future Instagram, WhatsApp, Threads or other providers can be added later.

V1 must not implement those providers or expose them in the UI.
