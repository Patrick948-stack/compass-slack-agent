# MVP Scope — Compass

This document defines exactly what is and is not in the Compass MVP. Its purpose is to keep the project focused during development and to give hackathon judges a clear picture of what has been built deliberately versus what is planned for the future.

---

## What MVP Means Here

MVP (Minimum Viable Product) does not mean "quick and sloppy." It means the smallest version of the project that:

1. Demonstrates the core value proposition clearly
2. Works reliably enough to demo without crashing
3. Can be built by one person in a hackathon timeframe

The four MVP features were chosen because they each solve a real student problem, are technically independent (one can be built without the others), and together tell a coherent story about what Compass is.

---

## Feature 1 — Ask Compass

### What it does
Students ask Compass a question about university life in natural language. Compass responds with a short, grounded answer drawn from the campus knowledge base.

### Acceptance criteria
- A student can @mention Compass or DM it with a question
- Compass reads from the local knowledge base before answering
- If the answer is in the knowledge base, Compass gives a short, warm response (2–4 sentences max)
- If the answer is NOT in the knowledge base, Compass says it is not sure and recommends a specific office or contact
- Compass never makes up facts, policies, or names
- The response is formatted as a clean Slack message (not a raw wall of text)

### Example questions it should handle
- "When is the last day to withdraw from a course?"
- "Where is the tutoring center?"
- "How do I apply for financial aid?"
- "What's the counseling center's phone number?"
- "Can I take CS101 without taking MATH101 first?"

### What it will NOT do in MVP
- Answer questions outside the campus knowledge base
- Remember previous questions (no conversation history)
- Handle follow-up questions or clarifications
- Access real-time data (enrollment numbers, live schedules)

---

## Feature 2 — Hand Raise

### What it does
Students use `/handraise` to submit a support request. A form opens in Slack. The request is saved and posted to a private staff channel.

### Acceptance criteria
- `/handraise` opens a Block Kit modal in Slack
- The modal includes:
  - A dropdown to select support type (Academic, Advising, Financial, Wellness, Administrative, Other)
  - A text input for a short description (required, max ~500 characters)
- On submit, the request is saved to the `hand_raise_requests` Supabase table
- A formatted notification is posted to the designated private support-staff Slack channel
- The student receives a confirmation message: "Your request has been received."
- The saved record includes: student Slack ID, support type, description, timestamp, and status = "open"

### What it will NOT do in MVP
- Let students track the status of their request
- Send email notifications to staff
- Allow staff to reply directly through Compass
- Handle escalation, routing, or assignment
- Show a request history to students

---

## Feature 3 — Study Buddies

### What it does
Students use `/findbuddy COURSE_CODE` to opt into a study group for a specific course. Compass matches them with other students who have already opted in for the same course.

### Acceptance criteria
- `/findbuddy CS101` (or any course code in caps) is a valid command
- If no course code is provided, Compass responds with a clear usage message
- The student's opt-in is saved to the `study_buddy_opts` Supabase table
- If other students are already opted in for the same course, Compass returns their names (Slack display names) in a formatted message
- If no one else has opted in yet, Compass confirms the student is added and says it will notify them when a match is found
- A student can only be listed once per course (no duplicate entries)
- Optionally: Compass posts a message to a `#study-groups` channel when a new match is made

### What it will NOT do in MVP
- Send notifications to existing matches when a new student opts in
- Allow students to remove themselves from a course
- Support study group scheduling, channels, or coordination
- Use any algorithm beyond exact course code matching

---

## Feature 4 — Success Team

### What it does
Students use `/myteam` to see a formatted card of key campus contacts.

### Acceptance criteria
- `/myteam` returns an ephemeral Block Kit message (visible only to the requesting student)
- The card shows entries for at least:
  - Academic Advisor
  - Tutoring Center
  - Financial Aid Office
  - Wellness / Counseling Center
  - Career Services
  - Library
- Each entry shows: role/name, email, office hours, and appointment link (where available)
- Data is read from `data/campus_resources.json`
- The message is formatted clearly — not a JSON dump

### What it will NOT do in MVP
- Show personalized contacts based on the student's actual advisor
- Pull from a real student information system
- Let students update or save their own contacts
- Show real-time office hours or availability

---

## Out of Scope for MVP

The following were explicitly cut to keep the project buildable and focused. They are valid future features, not forgotten ideas.

| Feature | Why it's out of scope |
|---|---|
| University login / SSO | Requires integration with identity providers (complex, institution-specific) |
| Real student data | Privacy and FERPA compliance — fake data only for demo |
| Admin dashboard | A separate web UI — outside the Slack-native scope |
| Staff reply through Compass | Requires a full conversation thread model |
| AI memory / conversation history | Adds complexity; each Ask Compass query is stateless for MVP |
| Mobile app | Slack already works on mobile — no separate app needed |
| Notification system for Hand Raise updates | Real-time status tracking requires more database and Slack API work |
| Mental health emergency handling | Must not be handled by an AI — defer to trained staff |
| Multi-workspace deployment | Single workspace only for hackathon demo |
| Rate limiting / abuse prevention | Handled at Slack's level for now |
| Payment / premium features | Not applicable |
| Vector database / semantic search | Local file search is sufficient at MVP scale |

---

## Definition of Done

The MVP is complete when:

- [ ] All four slash commands and the @mention trigger return a response
- [ ] Hand Raise modal opens, submits, and posts to staff channel
- [ ] Hand Raise data is visible in Supabase dashboard
- [ ] Study Buddies matching works for at least two test users in the same course
- [ ] Success Team card renders correctly and is ephemeral
- [ ] Ask Compass answers at least 5 sample questions from the knowledge base correctly
- [ ] Ask Compass correctly says "I don't know" for a question outside the knowledge base
- [ ] The app runs locally in Socket Mode without crashing during a 5-minute demo
- [ ] All environment variables are documented in `.env.example`
- [ ] README explains what the project is and how to run it

---

## Implementation Order

Build in this order to maximize early demo value and minimize dependency issues:

1. **Success Team** — no AI, no database, just read a JSON file and render Block Kit. Fast win.
2. **Ask Compass** — adds the AI layer on top of the app skeleton. Core value.
3. **Hand Raise** — introduces Supabase and modals. Slightly more complex.
4. **Study Buddies** — uses Supabase already set up in step 3. Straightforward.
