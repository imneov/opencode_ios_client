# Working Notes - OpenCode iOS Client

## Recent Changes

### Question Feature (2026-03-07)

Implemented the Question feature so the iOS client can handle AI-initiated questions from the OpenCode server. Previously, when the server's AI asked questions via the MCP `question` tool, the iOS client had no handler and the session would stall. Now the client displays question cards with selectable options and custom text input, sends replies back to the server, and the session continues.

**What was added:**

- `Models/QuestionModels.swift` — `QuestionOption`, `QuestionInfo`, `QuestionRequest` (Codable, matching server's question API contract)
- `Controllers/QuestionController.swift` — SSE event parsing for `question.asked`, `question.replied`, `question.rejected`
- `Views/Chat/QuestionCardView.swift` — Blue-themed SwiftUI card with radio/checkbox options, multi-question pagination, custom text input, dismiss/submit actions
- `Services/APIClient.swift` — Added `pendingQuestions()`, `replyQuestion()`, `rejectQuestion()` methods
- `Support/L10n.swift` — 10 new localization keys (EN + ZH) for question UI
- `AppState.swift` — `pendingQuestions` state, SSE event handling, refresh on session select/bootstrap, respond/reject methods
- `Views/Chat/ChatTabView.swift` — Renders `QuestionCardView` alongside existing `PermissionCardView`, updates scroll anchor

**Server API contract:**

- `GET /question` — list pending questions
- `POST /question/{requestID}/reply` — send answers (`{ "answers": [["label1"], ["label2"]] }`)
- `POST /question/{requestID}/reject` — dismiss question
- SSE events: `question.asked`, `question.replied`, `question.rejected`

**Tests added:** 12 new tests covering model decoding, controller event parsing, and SSE event structure.
