# Compass — Your University in Slack

> A Slack-native academic support agent that connects students to answers, resources, and each other.

---

## The Problem

University students deal with a constant, low-level friction that nobody talks about: not knowing who to ask, where to look, or whether it's even worth the effort. Financial aid deadlines get missed. Office hours go unattended because students didn't know they existed. A student struggles in a course alone when three classmates are in the same boat one floor below them.

Compass exists to close that gap — inside the tool many campuses already use every day.

---

## What Compass Does

Compass is a Slack agent with four focused features built for the Slack Agent Builder Challenge.

### Ask Compass
Type a question in Slack and get a grounded, sourced answer from the campus knowledge base. Compass will never make things up — if it doesn't know, it says so and points you to the right office.

### Hand Raise `/handraise`
A student can raise a support request in seconds. A form opens in Slack, the student picks a support type, writes a short description, and submits. The request is saved and posted to a private channel where staff can act on it.

### Study Buddies `/findbuddy COURSE_CODE`
Students opt into a study group for any course. Compass matches them with classmates who are already interested in the same course and posts a connection message. No complex algorithm — just a clean match on course code.

### Success Team `/myteam`
A personalized card showing the contacts every student should have: academic advisor, tutoring center, financial aid, wellness support, career services. Formatted in Slack Block Kit so it's readable at a glance.

---

## Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Slack framework | `@slack/bolt` (Node.js) | Official Slack SDK, well-documented, handles OAuth and events cleanly |
| Runtime | Node.js 20 | Stable LTS, large ecosystem, works well with Bolt |
| AI model | Groq API — Llama 3.3 70B | Free tier, fast inference, strong instruction-following |
| Knowledge base | Local `.md` / `.json` files | Simple, version-controlled, no external service needed for MVP |
| Database | Supabase (free tier) | Postgres with a REST API, easy to set up, no credit card required |
| Slack UI | Block Kit | Native Slack formatting — modals, cards, buttons |
| AI tool protocol | MCP (`@modelcontextprotocol/sdk`) | Connects the AI to the knowledge base as a structured tool call |

---

## Project Structure

```
compass-slack-agent/
├── src/
│   ├── app.js                  # Bolt app entry point
│   ├── handlers/
│   │   ├── askCompass.js       # Feature 1 — AI Q&A
│   │   ├── handRaise.js        # Feature 2 — Support requests
│   │   ├── studyBuddies.js     # Feature 3 — Course matching
│   │   └── successTeam.js      # Feature 4 — Campus contacts
│   ├── ai/
│   │   ├── groqClient.js       # Groq API wrapper
│   │   └── mcp/                # MCP server for knowledge base
│   ├── db/
│   │   └── supabase.js         # Supabase client
│   └── utils/
│       └── blocks.js           # Block Kit helpers
├── data/
│   ├── campus_resources.json   # Campus contacts (fake but realistic)
│   └── knowledge_base/         # Campus FAQ and policy files
├── prompts/
│   └── compass-system-prompt.md
├── docs/
│   ├── architecture.md
│   ├── mvp-scope.md
│   └── demo-script.md
├── .env.example
└── README.md
```

---

## Getting Started

> Full setup instructions will be added once the app is built. The sections below are placeholders.

### Prerequisites

- Node.js 20+
- A Slack workspace where you can install apps
- A Supabase project (free tier)
- A Groq API key (free at console.groq.com)

### Environment Variables

Copy `.env.example` to `.env` and fill in your keys:

```env
SLACK_BOT_TOKEN=xoxb-...
SLACK_SIGNING_SECRET=...
SLACK_APP_TOKEN=xapp-...
GROQ_API_KEY=gsk_...
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=...
SUPPORT_STAFF_CHANNEL_ID=...
STUDY_GROUPS_CHANNEL_ID=...
```

### Install and Run

```bash
npm install
npm run dev
```

---

## Hackathon

**Challenge:** Slack Agent Builder Challenge
**Track:** Slack Agent for Good — Education and Student Success

Compass addresses student wellbeing, academic accessibility, and peer connection. It is designed to work at any university that already uses Slack, with minimal setup and no integration with legacy systems required for MVP.

---

## Status

This project is under active development as part of a hackathon submission. MVP features are being built one at a time.

---

## Author

Built by Patrick Mulikuza.
