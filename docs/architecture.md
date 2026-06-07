# Architecture — Compass Slack Agent

This document explains how Compass is built: the components involved, how they talk to each other, and why each technology was chosen. It is written for a beginner who is implementing this project one feature at a time.

---

## High-Level Diagram

```
Student (Slack)
      │
      │  slash command / message
      ▼
┌─────────────────────────────────┐
│         Slack Platform          │
│  (events, slash commands,       │
│   modals, Block Kit)            │
└────────────┬────────────────────┘
             │  HTTPS + WebSocket (Socket Mode)
             ▼
┌─────────────────────────────────┐
│       Compass Node.js App       │
│       (@slack/bolt)             │
│                                 │
│  ┌──────────┐  ┌─────────────┐  │
│  │ handlers │  │   utils/    │  │
│  │ ask      │  │   blocks.js │  │
│  │ handraise│  └─────────────┘  │
│  │ buddies  │                   │
│  │ myteam   │                   │
│  └────┬─────┘                   │
└───────┼─────────────────────────┘
        │
        ├──────────────────────────────────────┐
        │                                      │
        ▼                                      ▼
┌───────────────────┐              ┌───────────────────────┐
│   Groq API        │              │   Supabase (Postgres) │
│   Llama 3.3 70B   │              │                       │
│                   │              │  hand_raise_requests  │
│   + MCP Server    │              │  study_buddy_opts     │
│   (knowledge base │              └───────────────────────┘
│    tool calls)    │
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│  Knowledge Base   │
│  (local files)    │
│  .md / .json      │
└───────────────────┘
```

---

## Components

### 1. Slack Platform

Slack is the interface. Students interact entirely through Slack: slash commands, modals, and formatted messages. There is no web UI, mobile app, or separate frontend.

Compass uses three Slack primitives:

- **Slash commands** — `/handraise`, `/findbuddy`, `/myteam` trigger specific features
- **App mentions** — `@Compass what is the add/drop deadline?` triggers the Ask Compass Q&A
- **Modals** — Block Kit modals collect structured input (e.g., the Hand Raise form)
- **Block Kit messages** — Formatted cards display answers, contacts, and match results

### 2. Compass Node.js App (`@slack/bolt`)

This is the core of the project. It runs as a Node.js server and uses Socket Mode, which means it connects to Slack over a WebSocket rather than needing a public HTTPS URL. This is important for local development and hackathon demos — no tunneling tool needed.

Bolt handles:
- Receiving events from Slack
- Verifying the request signature
- Routing events to the right handler function
- Sending responses back to Slack

Each feature lives in its own handler file under `src/handlers/`. This keeps the code organized and makes it easy to build and test one feature at a time.

### 3. AI Layer — Groq + Llama 3.3 70B

The Ask Compass feature uses an AI model to answer student questions. The model is accessed through the Groq API.

**Why Groq?**
- Free tier with no credit card required
- Very fast inference (sub-second on most queries)
- Supports Llama 3.3 70B, which is strong at instruction-following and grounded Q&A
- Simple REST API compatible with the OpenAI SDK format

**Why Llama 3.3 70B?**
- Open weights model — well-documented behavior
- Strong at following precise instructions (like "only answer from the provided context")
- Good balance of capability and cost at the free tier

**Grounding strategy:**
The AI is given the campus knowledge base as context in the system prompt or as a tool call result. It is explicitly instructed not to invent information. If the knowledge base does not contain the answer, it must say so and suggest the right office. This prevents hallucination.

### 4. MCP Server (Optional but Recommended)

MCP stands for Model Context Protocol. It is a standard for giving AI models structured access to external tools and data sources — similar to how a web browser has plugins.

For Compass, the MCP server exposes the local knowledge base as a tool the AI can call:

```
AI model → calls tool: search_knowledge_base("add/drop deadline")
MCP server → reads local .md files → returns relevant excerpts
AI model → generates answer grounded in those excerpts
```

**Why use MCP?**
Without MCP, you would need to load the entire knowledge base into the system prompt on every request. This wastes tokens and is slow. MCP lets the AI search only for what it needs.

**Why is it optional for MVP?**
It adds complexity. For a small knowledge base (under ~20 pages), loading the full context works fine. MCP becomes important when the knowledge base grows.

### 5. Knowledge Base (Local Files)

The knowledge base is a folder of plain text and Markdown files containing campus information:
- Course registration deadlines
- Financial aid policies
- Tutoring hours
- Wellness resources
- Campus contacts

**Why local files instead of a vector database?**
For MVP with a small knowledge base, a vector database (like Pinecone or Weaviate) is overkill. Local files are:
- Free
- Simple to update (just edit the file)
- Version-controlled in Git
- Fast enough at small scale

If the knowledge base grows beyond ~50 pages, switching to a vector database for semantic search would be the right next step.

### 6. Supabase

Two features need to persist data between requests:

- **Hand Raise** — support requests need to be saved so staff can review them later
- **Study Buddies** — student opt-ins need to be stored so Compass can match them

Supabase provides a Postgres database with a simple REST API (via `@supabase/supabase-js`).

**Why Supabase over SQLite or a JSON file?**
- Free hosted tier — no local database setup
- Real-time subscription support (future feature: staff get notified when a new request comes in)
- Web dashboard for reviewing data during a demo
- Easy to scale if the project grows

**Tables needed for MVP:**

```sql
-- Hand Raise requests
create table hand_raise_requests (
  id          uuid primary key default gen_random_uuid(),
  student_id  text not null,       -- Slack user ID
  type        text not null,       -- Academic, Financial, etc.
  description text not null,
  status      text default 'open', -- open | resolved
  created_at  timestamptz default now()
);

-- Study Buddy opt-ins
create table study_buddy_opts (
  id          uuid primary key default gen_random_uuid(),
  student_id  text not null,       -- Slack user ID
  course_code text not null,       -- e.g. CS101
  created_at  timestamptz default now(),
  unique(student_id, course_code)  -- prevent duplicate signups
);
```

---

## Data Flow Per Feature

### Ask Compass

```
1. Student mentions @Compass or sends a DM with a question
2. Bolt routes the event to askCompass.js handler
3. Handler loads relevant knowledge base content (directly or via MCP)
4. Handler sends a request to Groq API with:
   - System prompt (tone + grounding rules)
   - Knowledge base context
   - Student's question
5. Groq returns a response
6. Handler formats the response as a Block Kit message
7. Bolt sends the message back to Slack
```

### Hand Raise

```
1. Student types /handraise
2. Bolt routes the slash command to handRaise.js handler
3. Handler opens a Block Kit modal in Slack (form with support type + description)
4. Student fills out and submits the form
5. Bolt receives the view_submission event
6. Handler saves the request to Supabase hand_raise_requests table
7. Handler posts a formatted notification to the private support-staff Slack channel
8. Bolt sends a confirmation message to the student
```

### Study Buddies

```
1. Student types /findbuddy CS101
2. Bolt routes the slash command to studyBuddies.js handler
3. Handler parses the course code from the command text
4. Handler queries Supabase: "are there other students opted in for CS101?"
5a. If yes: returns a formatted list of matches to the student
5b. If no existing matches: saves the student's opt-in and sends a "we'll notify you" message
6. (Optional) Handler posts to a #study-groups channel to make the match visible
```

### Success Team

```
1. Student types /myteam
2. Bolt routes the slash command to successTeam.js handler
3. Handler reads campus_resources.json from the data/ folder
4. Handler builds a Block Kit card with roles, names, emails, office hours
5. Bolt sends the card as an ephemeral message (visible only to the requesting student)
```

---

## Key Technical Decisions

| Decision | What was chosen | Why |
|---|---|---|
| Socket Mode vs HTTP | Socket Mode | No public URL needed — works locally without ngrok |
| AI provider | Groq | Free tier, fast, no credit card |
| Database | Supabase | Free hosted Postgres, easy SDK |
| Knowledge base storage | Local files | Simple, version-controlled, free |
| UI framework | Block Kit only | Native Slack — no frontend to build or host |
| MCP | Optional | Adds structure when KB grows; skip for smallest MVP |

---

## What Is Not in This Architecture

The following were deliberately left out of MVP to keep the project buildable in a hackathon:

- Authentication / SSO (students are identified by their Slack user ID)
- Admin dashboard
- Email notifications
- Real student record system (SIS integration)
- Vector database / semantic search
- Multi-tenant support (one workspace only)
- Rate limiting (trust Slack's built-in rate limits for now)
