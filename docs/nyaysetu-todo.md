# NyaySetu Delhi Police Voice Co‑Pilot – Technical TODO

This is a detailed, implementation-ready TODO list that an AI coder (Copilot, Codeium, etc.) can follow step-by-step to build the MVP. Adjust technologies (Node/Go/Python, cloud provider) as per your preference.

---

## 1. Project Planning & Repo Setup

1. Create a new monorepo (e.g., `nyaysetu-delhi-police-voice-copilot`) with `frontend/`, `backend/`, `infrastructure/`, and `docs/` folders.
2. Initialize Git and add a clear `README.md` describing the vision, core features, and tech stack.
3. Decide primary backend language and framework (e.g., Node.js + NestJS / Express, or Go + Gin).
4. Decide primary frontend stack (e.g., React + Next.js + TypeScript, or simple React SPA).
5. Create an initial `backend` project with package manager config (`package.json` / `go.mod` / `pyproject.toml`).
6. Create an initial `frontend` project (Next.js / React) with TypeScript enabled.
7. Set up shared coding standards: Prettier, ESLint, EditorConfig across repo.
8. Configure basic CI (GitHub Actions) to run lint and tests for frontend and backend on each push.
9. Create `.env.example` listing all environment variables required (telephony API keys, LLM keys, DB URL, etc.).
10. Add basic `CONTRIBUTING.md` and issue templates for future collaborators.

---

## 2. Domain & Requirements Detailing

11. Write a `docs/domain.md` describing key entities: Citizen, Call, Case, FIRDraft, Officer, Station, Jurisdiction, KnowledgeDocument.
12. Capture the 5 core problem goals and map them to features in `docs/product-spec.md`.
13. Define target languages and dialects for MVP (Hindi, English, Hinglish; optional Punjabi/Bengali).
14. List call types: emergency, non-emergency complaint, info query, follow-up, feedback.
15. Define escalation rules: when to connect to human, when to alert women’s cell, cyber cell, etc.
16. Draft sample conversation flows (scripts) for at least 5 use-cases (theft, domestic violence, cyber fraud, harassment, information query).
17. Define privacy constraints: what PII is collected, retention policy, and masking requirements.
18. Define success KPIs to measure in analytics (e.g., FIR conversion rate, call resolution time, sentiment score).

---

## 3. Core Backend Infrastructure Setup

19. Choose database (e.g., PostgreSQL) and ORM (Prisma/TypeORM/GORM) and add to backend.
20. Design DB schema (in `docs/schema.sql` or ORM models) for:
    - `citizens` (phone, optional demographics),
    - `calls` (timestamp, channel, language, type),
    - `call_transcripts`,
    - `cases`,
    - `fir_drafts`,
    - `officers`,
    - `stations`,
    - `jurisdictions`,
    - `events` (audit log).
21. Implement DB migration system (Prisma migrate, Flyway, or similar).
22. Set up basic backend API server with health-check endpoint (`/health`).
23. Integrate a logger (pino/winston/logrus) with structured logs and correlation IDs.
24. Implement standard error-handling middleware with meaningful error codes and messages.
25. Add configuration management (e.g., using `dotenv` and config module) for environment separation (dev, staging, prod).

---

## 4. Telephony & Channel Integration

26. Choose a telephony provider (Exotel/Twilio/Ozonetel) suitable for India; document chosen APIs.
27. Create a `telephony` service in backend that exposes webhooks to receive inbound call events.
28. Implement endpoint `/webhook/voice/inbound` to handle telephony provider callbacks.
29. Parse call metadata (caller number, call SID, timestamp, direction, IVR options) and store a `call` record.
30. Implement logic to answer the call with a greeting and connect audio stream to ASR pipeline.
31. For providers supporting media streaming (e.g., Twilio Media Streams), set up WebSocket / gRPC endpoints for streaming audio.
32. For providers without streaming, design DTMF/IVR fallbacks and simpler record-then-process flows.
33. Configure IVR menu for language selection (press 1 for Hindi, 2 for English, etc.) and route to appropriate ASR config.
34. Set up outbound call capability for follow-ups and surveys; implement `/api/outbound-call` endpoint that triggers telephony API.
35. Plan WhatsApp Business API integration for notifications; create stub `/webhook/whatsapp` for incoming messages.
36. Implement SMS gateway integration (e.g., Textlocal, MSG91) for basic text updates.

---

## 5. Speech Stack (ASR & TTS)

37. Integrate Sarvam AI Speech-to-Text API (or similar Indic ASR) in a dedicated `asr` module.
38. Implement a function `transcribeStream(callId, audioChunks, languageHint)` to handle real-time streaming.
39. Implement fallback batch transcription for recorded audio if streaming is not supported.
40. Integrate Bhashini ASR APIs as an alternative / backup for Hindi and regional languages.
41. Implement logic to auto-select ASR model based on IVR language choice and call metadata.
42. Wrap ASR APIs with retry logic and timeout handling.
43. Implement a `tts` module integrating Sarvam/Bhashini TTS for voice responses.
44. Predefine at least 2–3 TTS voices (gender, tone) for different scenarios (soothing, neutral, authoritative).
45. Create helper `speak(callId, text, voiceProfile)` that sends synthesized audio back to telephony provider.
46. Add unit tests for ASR and TTS wrappers using mocked responses.

---

## 6. Language Detection, Translation & NLU

47. Implement automatic language detection using Bhashini or heuristic based on IVR choice + ASR confidence.
48. Integrate translation API (Bhashini NMT/IndicTrans) to normalize text into a canonical internal language (e.g., English for LLM, or Hindi if using Indic LLM).
49. Define NLU intents and entities: emergency flag, crime type, relationship between parties, location, time, weapon, vulnerability flags.
50. Implement basic rule-based NLU (regex / keyword based) as a fallback for critical emergency detection (e.g., “maar dala”, “help”, “rape”).
51. Integrate LLM-based NLU (via an LLM API) to extract structured slots from utterances.
52. Define JSON schema for `CallNLUState` including slots for case type, parties involved, location, and urgency level.
53. Implement persistence of `CallNLUState` to DB per call, updating after each user turn.

---

## 7. Conversation Orchestration Engine

54. Design a conversation state machine with states (Greeting, IntentDetection, Clarification, StatementCapture, LegalCheck, Summary, Escalation, Closure).
55. Implement a `ConversationManager` class/service that:
    - Receives ASR text,
    - Updates NLU state,
    - Decides next action (ask more, summarize, escalate),
    - Generates response text.
56. Encode conversation flows for at least 5 key case types as JSON/YAML configuration to allow quick tweaking.
57. Add logic for emotion-aware responses (e.g., detect profanity/crying/raised voice via keywords and adjust system prompts/voice selection).
58. Integrate LLM to generate natural, culturally appropriate responses based on conversation state and RAG outputs.
59. Implement timeout and fallback behaviour if LLM or ASR fails (e.g., “I’m facing some technical issues, connecting you to a human officer now”).
60. Implement escalation rules: immediate escalation for emergency keywords, domestic violence signals, self-harm indicators.
61. Implement a `handover` mechanism where AI summarizes call and passes context blob to human officer via dashboard or telephony warm transfer.

---

## 8. Legal Knowledge Base & RAG

62. Collect legal and policy documents: Bharat Nyay Sanhita text, Delhi Police Standing Orders, relevant MHA circulars.
63. Convert documents into machine-readable formats (Markdown/JSON) and store in `docs/knowledge-base/`.
64. Implement a document ingestion script to chunk text into semantically meaningful sections.
65. Use an embedding model (e.g., multilingual MiniLM, or Indic-specific embeddings) to embed chunks.
66. Store embeddings in a vector store (e.g., pgvector, Qdrant, Pinecone) along with metadata (section, law, topic).
67. Implement a `KnowledgeService` with method `retrieveRelevantSections(query, topK)` for RAG.
68. Integrate RAG into the LLM prompt so that legal suggestions come only from retrieved sections.
69. Implement specific helpers: `suggestSectionsForCase(CallNLUState)`, `generateCitizenRightsSummary(CallNLUState)`, `generateOfficerGuidance(CallNLUState)`.
70. Add caching layer for common queries (e.g., lost phone, theft, cyber fraud) to reduce RAG overhead.
71. Write tests to ensure retrieval returns expected sections for canned queries.

---

## 9. Case Management, E-Statements & FIR Drafts

72. Define DB models for `cases` and `fir_drafts` with fields for status, assigned station/officer, legal sections, and history.
73. Implement an API endpoint `/api/cases` to create and fetch case records.
74. In ConversationManager, after enough information is collected, auto-create a `case` entry and link it to the `call`.
75. Implement logic to generate a structured e-statement as a timeline of facts (who/what/when/where/how) based on NLU state and transcript.
76. Implement `generateFIRDraft(caseId)` that:
    - Calls RAG to find relevant sections,
    - Uses LLM to draft an FIR-like narrative,
    - Stores result in `fir_drafts` table.
77. Create API `/api/cases/:id/fir-draft` to fetch FIR draft for human review.
78. Add status transitions: `NEW` → `UNDER_REVIEW` → `FIR_REGISTERED` or `CLOSED`.
79. Implement audit logging of all changes to `cases` and `fir_drafts`.
80. Add optional stub integration with CCTNS/e-FIR API (mock if real API not accessible) to demonstrate how FIR would be lodged.

---

## 10. Officer & Station Tools (Backend)

81. Design DB models for `stations` (name, address, jurisdiction polygon or pincode list).
82. Implement `JurisdictionService` that maps caller location (area, pincode, or rough description) to nearest station.
83. Create `officers` table with role, station assignment, contact info.
84. Implement basic authentication (JWT/session-based) for officer logins.
85. Build REST endpoints for officers to:
    - View assigned cases,
    - See call summaries,
    - Approve/edit FIR drafts,
    - Change case status and add notes.
86. Add role-based access control (e.g., SHOs vs constables vs HQ users).

---

## 11. Frontend – Officer Dashboard

87. Scaffold Next.js/React app with routing for `/login`, `/dashboard`, `/cases`, `/calls`, `/analytics`.
88. Implement authentication flow on frontend that stores JWT in httpOnly cookies.
89. Build a "Cases" list view showing case ID, type, language, status, priority, and last update time.
90. Build a "Case Detail" page showing:
    - Call summary,
    - Transcript snippets,
    - Extracted NLU slots,
    - Suggested legal sections,
    - FIR draft with edit capability.
91. Implement "Approve FIR draft" action that updates backend and triggers next workflow (e.g., e-FIR stub call).
92. Build "Calls" view summarizing recent calls, their classification, and outcomes.
93. Implement basic search/filter by station, case type, date range.
94. Implement responsive UI so dashboard works on laptops and tablets officers may use.

---

## 12. Frontend – Admin & Analytics Views

95. Create an "Analytics" dashboard route for supervisors.
96. Display key metrics: number of calls handled, auto-resolved vs escalated, cases opened, FIR drafts generated.
97. Add charts for language distribution, issue categories, and average response time.
98. Implement geographic view (by thana/area) showing complaint density heatmap (even rudimentary, e.g., choropleth by station).
99. Implement sentiment trend line based on LLM sentiment analysis of call transcripts.
100. Add a "Configuration" page to tweak thresholds (e.g., when to escalate, what keywords mark emergencies).

---

## 13. Notifications & Citizen Updates

101. Implement notification service that can send SMS/WhatsApp messages given a `caseId` and template.
102. Define message templates: `CASE_CREATED`, `FIR_DRAFTED`, `CASE_UPDATED`, `CASE_CLOSED`.
103. Connect conversation flow so that, on case creation, citizen receives case ID and brief summary via SMS/WhatsApp.
104. Implement endpoint `/public/case-status` where citizens can check status using case ID + OTP.
105. Implement event handlers so that every status change triggers an audit log entry and optional notification.

---

## 14. Security, Privacy & Compliance

106. Implement PII masking in logs (mask phone numbers, names in logs but not in DB where needed).
107. Encrypt sensitive fields at rest where possible (e.g., using field-level encryption or DB-native features).
108. Ensure all APIs use HTTPS; configure secure headers (CORS, CSP where applicable).
109. Implement rate limiting on public endpoints to prevent abuse.
110. Add role-based access checks to all officer/admin endpoints.
111. Draft a minimal `docs/privacy.md` describing data handling, retention, and anonymization for analytics.

---

## 15. Testing & QA

112. Write unit tests for key services: ASR wrapper, TTS wrapper, NLU extraction, KnowledgeService, ConversationManager.
113. Write integration tests to simulate a full call: mock ASR output, run conversation loop, ensure case + FIR draft created.
114. Add API tests for officer actions (view/edit case, approve FIR draft).
115. Add snapshot tests for core frontend components (case list, case detail, analytics cards).
116. Create a small synthetic dataset of example calls and expected outputs for regression testing.
117. Configure GitHub Actions to run backend + frontend test suites on each push.

---

## 16. Deployment & DevOps

118. Create Dockerfiles for backend and frontend.
119. Write a `docker-compose.yml` for local dev including Postgres, vector DB, and both services.
120. Set up simple cloud deployment (e.g., on Render/Fly.io/AWS ECS/GCP Cloud Run) with environment variables configured.
121. Configure a managed Postgres instance and migrate schema on deploy.
122. Set up logging aggregation (CloudWatch/Stackdriver/ELK) and basic alerting on critical failures.
123. Optionally, create Terraform or Pulumi scripts in `infrastructure/` to define infra as code.

---

## 17. Demo & Hackathon Readiness

124. Implement a script to simulate inbound calls (play pre-recorded audio to telephony API) for predictable demos.
125. Prepare 3–5 demo scenarios with pre-filled legal KB and station data.
126. Seed DB with a few stations, officers, and example cases.
127. Add a "demo mode" toggle that displays helpful overlays on frontend for judges (e.g., highlighting how AI decided next step).
128. Record a short screen capture of end-to-end flow: citizen call → AI conversation → dashboard view → FIR draft.
129. Update `README.md` with clear run instructions (`docker-compose up`, .env configuration, sample test numbers).
130. Create a short `docs/architecture.md` with architecture diagram (can be generated from this TODO and your PPT).

This TODO list is intentionally verbose so a human or AI assistant can implement each part step by step. Adjust exact tools (e.g., telephony or cloud provider) as per availability.