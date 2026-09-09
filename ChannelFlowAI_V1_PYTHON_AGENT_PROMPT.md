# ChannelFlowAI V1 — Python Rebuild Master Prompt

You are rebuilding ChannelFlowAI V1 from scratch in Python.

## Source of truth
Use these three documents as the product/UX/technical source of truth:
- ChannelFlowAI_V1_PRD_PYTHON.md
- ChannelFlowAI_V1_UI_UX_PYTHON.md
- ChannelFlowAI_V1_TRD_PYTHON.md

Do not silently add V2 features.

## Important reset
The previous Node.js/Bun prototype must NOT be treated as a production implementation. Do not copy its simulated authentication, JSON database, fake forwarding, fake OTP, hardcoded secrets, or placeholder queue logic.

## Python stack
- Python 3.12+
- aiogram 3.x for bot UI
- Telethon for authorized Telegram user-session / MTProto operations
- PostgreSQL + SQLAlchemy 2.x + Alembic
- Redis + arq (or another clearly justified Redis-backed Python queue)
- Pydantic Settings
- Docker Compose
- pytest + pytest-asyncio
- Ruff
- Sentry-compatible error tracking

## Architecture
Keep bot control-plane and Telegram user-session data-plane separated:

Telegram User
→ aiogram Bot
→ application/service layer
→ PostgreSQL + Redis
→ worker
→ Telethon authorized session
→ source
→ destination

## Non-negotiable
1. Real Telethon authentication: phone → OTP → optional 2FA → encrypted session persistence.
2. Never store OTP or 2FA passwords.
3. Never hard-code BOT_TOKEN, API_ID, API_HASH, DATABASE_URL, REDIS_URL or ENCRYPTION_KEY.
4. Encrypt Telegram session material at rest.
5. PostgreSQL must be the real authoritative database. No JSON-file database.
6. Redis queue must be real. No fake process loop.
7. Forward and Copy must perform real Telegram delivery.
8. Use atomic idempotency/unique constraints to prevent duplicates.
9. Scope every user/account/task query to the correct application user.
10. Handle FloodWait, retries, worker restart and permanent failures.
11. Do not claim features that are not implemented.
12. Keep Telegram-specific code behind interfaces/adapters for future providers.

## UX
Implement the approved flow:
Start → Language → Main Menu → Connect Account → Phone → OTP → optional 2FA → Connected → Create Task → Source → Destination → Mode → Filters → Transform → Delay → Review → Test → Activate.

Keep Back / Cancel / Home behavior consistent with the UI/UX document.

## Deliverables
Create:
- production-quality Python package
- pyproject.toml and/or requirements.txt with pinned compatible versions
- .env.example
- Alembic migrations
- Dockerfile
- docker-compose.yml
- PostgreSQL repositories/models
- Redis queue + worker
- aiogram handlers/keyboards/FSM states
- Telethon account/auth/session service
- forwarding/copy delivery service
- filters/transforms/idempotency
- tests
- README with exact local setup and Docker setup

## Validation
Before declaring completion:
- run tests
- run Ruff
- run type checking if configured
- validate Alembic migrations
- start bot + worker against PostgreSQL + Redis
- verify the main onboarding flow
- verify controlled Telegram authentication
- verify a controlled source → destination text delivery
- verify at least the required media types
- verify duplicate protection
- verify pause/resume/delete
- verify FloodWait/retry behavior
- verify restart recovery

If something cannot be tested without real Telegram credentials, mark it clearly as requiring controlled credentials instead of faking success.

Work in small milestones and report what is actually implemented after each milestone.
