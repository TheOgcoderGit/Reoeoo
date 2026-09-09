# ChannelFlowAI V1 — UI/UX Specification

**Version:** 1.1  
**Interface:** Telegram bot  
**Implementation:** Python

## 1. UX Principles

- Match the approved reference onboarding flow and hierarchy.
- Keep every step short and actionable.
- Use consistent Back, Cancel and Home controls where applicable.
- Hide implementation details unless needed for troubleshooting.
- Clearly show Connected, Active, Paused, Attention Required and Error states.
- Prefer guided wizard flows over large forms.

## 2. Start & Language

### `/start`

Introduce ChannelFlowAI and its core value:

> Automate your Telegram message flow.

Primary action:

`🌐 Choose Language`

### Language

Options:

- 🇬🇧 English
- 🇮🇳 Hinglish

Persist the selected language.

## 3. Main Menu — Not Connected

Show:

- Welcome message
- Account: Not connected
- `📱 Connect Account`
- `✨ Features`
- `🔐 Why connect?`
- `❓ How it works`
- `📞 Support`

Do not show task controls before an account is connected.

## 4. Main Menu — Connected

Show:

- Welcome back
- Connected account identity/status
- `🚀 Create Task`
- `📋 My Tasks`
- `📊 Statistics`
- `📱 Account`
- `✨ Features`
- `❓ How it works`
- `⚙️ Settings`
- `📞 Support`

## 5. Authentication Screens

### Connect Account
Explain why Telegram account access is required.

### Phone
Request international phone number and show an example.

### OTP
Explain that Telegram sent a verification code.

### 2FA
Request the Telegram two-step password only when required.

### Success
Show connected account and actions for Create Task / Main Menu.

### Errors
Handle invalid number, invalid/expired code, wrong password, cancelled authentication, session failure and rate limits.

Authentication must be implemented through the Python Telegram client layer; the bot UI must never expose raw session credentials.

## 6. Create Task Wizard

`Source → Destination → Mode → Filters → Transform → Delay → Review → Test → Activate`

Show progress such as `Step 2 of 7`.

## 7. Source & Destination

V1 accepts a Telegram username or chat ID.

Validate the input and confirm the resolved chat identity when possible. Do not allow task creation if the chat cannot be resolved or accessed.

## 8. Mode

Two choices:

- `📎 Forward` — Telegram forwarding behavior.
- `📋 Copy` — copied content without the normal forwarded header where supported.

The selected mode must be visually obvious.

## 9. Filters

Choices:

- No Filters
- Keyword Filter
- Message Type Filter

Keyword modes:

- Include
- Exclude

Empty filters mean no filtering.

## 10. Transform

Choices:

- Skip
- Add Header
- Add Footer
- Find & Replace

Show a compact preview where practical.

## 11. Review & Activation

Review:

- Source
- Destination
- Mode
- Filters
- Transformation
- Delay

Actions:

- Test
- Edit
- Create / Activate
- Cancel

## 12. My Tasks

List tasks with:

- Status
- Source → Destination
- Activity count

Task actions:

- Pause
- Resume
- Edit
- Duplicate
- Delete

Deletion requires confirmation.

## 13. Statistics

V1 uses lightweight text statistics:

- Total processed
- Forwarded
- Filtered
- Failed
- Last activity

Charts are future scope.

## 14. Empty / Loading / Error States

### No Tasks
Explain how to create the first task and show Create Task CTA.

### Connecting
Show progress without exposing OTP/password.

### No Access
Explain that the connected account cannot access the selected chat.

### Destination Failure
Provide permission/access guidance.

### Session Expired
Show Reconnect Account CTA.

### Worker Issue
Show Attention Required; never falsely report Active.

## 15. Navigation Contract

- Back returns to the previous logical step.
- Cancel abandons the current operation without creating partial tasks.
- Home returns to the correct main menu.
- Completed authentication does not restart unless the user explicitly reconnects.

## 16. Copy Style

Short, friendly and professional.

Use consistent terms:

`Account`, `Source`, `Destination`, `Task`, `Forward`, `Copy`, `Filter`, `Transform`, `Active`, `Paused`.
