# ChannelFlowAI V1 — Product Requirements Document

**Version:** 1.1  
**Product:** ChannelFlowAI  
**Scope:** Telegram → Telegram automation  
**Implementation:** Python

## 1. Product Vision

ChannelFlowAI is a reliable Telegram message automation product. V1 focuses only on Telegram-to-Telegram forwarding. The brand and architecture remain extensible for future social platforms, but those platforms are explicitly out of V1 scope.

## 2. Goals

- Deliver the approved onboarding and Telegram-account connection flow.
- Allow users to create, manage, pause, resume and delete Telegram forwarding tasks.
- Support reliable forwarding/copying of core message types.
- Provide basic filtering, transformation, delay and duplicate protection.
- Handle authentication, rate limits, retries, failures and session recovery safely.
- Make V1 suitable for controlled beta testing before adding monetization or additional platforms.

## 3. Non-Goals

- Instagram, WhatsApp, Threads, Discord, X or other platforms.
- AI rewriting, translation, affiliate-link processing, advanced regex and advanced scheduling.
- Teams, white-label, public API, webhooks and advanced analytics.
- Paid plans/billing in the initial V1 beta.

## 4. Core User Flow

`Start → Language → Main Menu → Connect Telegram Account → Phone → OTP → optional 2FA → Connected → Create Task → Source → Destination → Mode → Filters → Transform → Delay → Review → Test → Activate`

## 5. Functional Requirements

### Onboarding
- `/start`
- English and Hinglish
- Persistent language selection
- Back / Cancel / Home navigation

### Telegram Account
- Connect account
- OTP
- Optional 2FA
- Connected status
- Reconnect
- Disconnect
- Session-expiry handling

### Forwarding Tasks
- One source → one destination per task
- Multiple tasks per user
- Active / Paused / Attention Required / Error states
- Forward and Copy modes

### Message Types
- Text
- Photo
- Video
- Document
- Audio
- Voice

### Filters
- Message type
- Include keywords
- Exclude keywords
- Case-insensitive matching
- Multiple keywords

### Transformations
- Add header
- Add footer
- Find & replace
- Remove specific text

### Delay
- Immediate
- Configurable delay

### Reliability
- Redis-backed queue
- Retry
- Rate-limit handling
- Idempotency
- Duplicate protection
- Failed-job handling

### Task Management
- List
- View
- Edit
- Pause
- Resume
- Duplicate
- Delete
- Statistics

### Notifications
Important system/task events only. Do not notify for every forwarded message.

### Help / Settings
- How it works
- Troubleshooting
- Language
- Notifications
- Account settings

## 6. Test Message

Before activation, users should be able to run a controlled test delivery when technically possible. The UI must clearly report success or the exact reason the test cannot run.

## 7. Acceptance Criteria

- A new user can reach the main menu without errors.
- A valid Telegram account can complete OTP/2FA authentication and remain connected across worker restarts.
- A valid source-to-destination task can be created and activated.
- Supported messages are delivered according to the selected mode.
- Filters and transformations are deterministic.
- A source message is not duplicated for the same task after reconnect/retry.
- Telegram rate limits are handled without crashing the worker.
- Pause/resume/delete operations behave predictably.
- Authentication secrets are never exposed in logs or user-facing errors.

## 8. Future Compatibility

Use provider-neutral concepts such as Connection, Channel, Source, Destination, Task, Filter, Transform and Delivery. Do not hard-code Telegram terminology into core domain models where a generic abstraction is practical.

## 9. V1 Release Gate

Release only after authentication, task creation, message delivery, media handling, retries, duplicate protection, pause/resume, error handling and recovery have been tested in a controlled beta environment.
