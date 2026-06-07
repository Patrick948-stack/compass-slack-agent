# Demo Script — Compass

This is the step-by-step script for demoing Compass to hackathon judges, in a live presentation or in a recorded video submission.

Total demo time target: **3–4 minutes**

---

## Before the Demo

Make sure these are ready before you start:

- [ ] Compass app is running locally (`npm run dev`)
- [ ] Socket Mode is connected (you should see "⚡️ Bolt app is running!" in the terminal)
- [ ] You are logged into the Slack workspace where Compass is installed
- [ ] Supabase dashboard is open in a browser tab (to show live data)
- [ ] The `#support-staff` Slack channel is visible in a second window or tab
- [ ] Terminal is visible but not showing sensitive keys
- [ ] Screen resolution is high enough to be readable on a recording

---

## Opening (30 seconds)

**Say:**

> "Every university student has had this moment — you need to know a deadline, or you need help, but you don't know who to ask or where to look. You close the browser and do nothing. Compass fixes that by bringing your university's support system directly into Slack."

> "I'm going to show you four features in the next three minutes."

---

## Feature 1 — Success Team (30 seconds)

**Action:** In any Slack channel or DM, type:
```
/myteam
```

**What happens:** A formatted card appears showing campus contacts — advisor, tutoring center, financial aid, wellness, career services.

**Say:**

> "The first thing any student needs is to know who their support team is. `/myteam` shows them instantly. It's ephemeral — only they can see it — so it doesn't spam the channel."

**Talking point for judges:**
> "This pulls from a `campus_resources.json` file. In a real deployment, this could be connected to a student information system to show personalized contacts."

---

## Feature 2 — Ask Compass (60 seconds)

**Action 1 — Question in the knowledge base:**

In the Compass DM or a channel where Compass is present, type:
```
@Compass what is the deadline to withdraw from a course this semester?
```

**What happens:** Compass responds with a short, grounded answer from the knowledge base. The answer cites the source or says "based on the academic calendar."

**Say:**

> "Students ask questions like this constantly. Compass checks its campus knowledge base first before answering — it will not make things up."

**Action 2 — Question outside the knowledge base:**

```
@Compass what is the parking policy for the east lot?
```

**What happens:** Compass says it doesn't have that information and suggests contacting the Facilities or Campus Safety office.

**Say:**

> "This is the honest behavior. If Compass doesn't know, it says so. It tells the student exactly who to contact instead of guessing. This is the most important design decision in the whole project."

---

## Feature 3 — Hand Raise (60 seconds)

**Action:** Type:
```
/handraise
```

**What happens:** A modal opens with a support type dropdown and a description field.

**Fill out the form:**
- Support type: Financial
- Description: "I haven't received my financial aid disbursement yet and rent is due Friday."

**Click Submit.**

**What happens next:**
- The student sees: "Your request has been received. A member of the support team will follow up with you."
- In the `#support-staff` channel, a formatted card appears with the request details.

**Switch to the Supabase dashboard tab.**

**Say:**

> "The request is saved in the database. Staff can review it, track it, and mark it resolved. This is a simple but important workflow — it replaces the email chain, the 'can you forward this to so-and-so', the request that falls through the cracks."

---

## Feature 4 — Study Buddies (45 seconds)

**Action 1 — Opt in:**
```
/findbuddy CS101
```

**What happens:** Compass saves the opt-in. If no one else is in CS101 yet, it says: "You're the first to sign up for CS101! We'll let you know when someone else joins."

**Action 2 — Match (switch to a second Slack account or ask a helper to do this):**
```
/findbuddy CS101
```

**What happens:** Compass detects the match and returns a message to both students showing who else is interested in CS101.

**Say:**

> "Studying alone is one of the top predictors of struggling academically. `/findbuddy` takes fifteen seconds and removes the awkwardness of finding classmates. No algorithm, no scheduling app — just a match on course code."

---

## Closing (30 seconds)

**Say:**

> "Compass is a Slack-native academic support agent. It answers questions, routes support requests, connects students with each other, and puts every key contact in one card. All four features work without leaving Slack."

> "The stack is Bolt on Node.js, Groq for the AI layer, Supabase for persistence, and plain files for the knowledge base. It's a beginner-friendly stack that can scale."

> "The most important thing Compass does is reduce the friction between a student who needs help and the person or resource that can give it to them."

---

## If Something Goes Wrong

| Problem | What to do |
|---|---|
| Modal doesn't open for `/handraise` | Check that the slash command is registered in the Slack app manifest |
| Ask Compass doesn't respond | Check Socket Mode connection in terminal — may need a restart |
| Supabase data not showing | Check env variables — `SUPABASE_URL` and `SUPABASE_ANON_KEY` |
| Study buddy match not working | Confirm both test accounts are in the same workspace and used the same course code |
| App crashes | Have `npm run dev` ready in the terminal — restart takes under 5 seconds |

---

## Judge Q&A Prep

**Q: How does it avoid hallucinating?**
> The AI is given explicit instructions in the system prompt: only answer from the provided context. If the context doesn't contain the answer, say "I'm not sure" and name the right office. This is enforced at the prompt level, not just hoped for.

**Q: What would it take to deploy this at a real university?**
> The main changes would be: connect the knowledge base to real institutional documents, integrate with a student information system for personalized contacts, and set up proper Slack workspace permissions with IT. The architecture supports this — the data sources are pluggable.

**Q: Why Groq instead of OpenAI?**
> Free tier, no credit card required for the hackathon, and fast enough for synchronous Slack responses. The prompt engineering approach works with any LLM provider — switching to OpenAI or Anthropic is a one-line change.

**Q: How is student privacy handled?**
> Students are identified only by their Slack user ID — no names are stored in the database. The app only stores the data it needs: support type, description, and course code. No student records, grades, or personal information are accessed or stored.
