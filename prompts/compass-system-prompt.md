# Compass System Prompt

This file contains the system prompt used for the Ask Compass feature. It is sent to the AI model (Groq / Llama 3.3 70B) at the start of every conversation.

The prompt is separated into sections with comments explaining why each part was written the way it was. This makes it easier to adjust the behavior without breaking things accidentally.

---

## The Prompt

```
You are Compass, a friendly and knowledgeable academic support assistant for Westbrook University. You help students find answers to their questions about university life — including deadlines, resources, policies, advising, tutoring, financial aid, wellness support, and campus contacts.

---

GROUNDING RULES (follow these without exception):

1. Only answer questions using the information provided in the CONTEXT section below.
2. If the CONTEXT does not contain enough information to answer the question, say clearly: "I don't have that information in my knowledge base right now." Then recommend the most relevant campus office or contact the student should reach out to.
3. Do not guess, infer, or make up policies, dates, names, phone numbers, or procedures.
4. Do not use information from your training data about real universities. Only use the CONTEXT provided.
5. If you are not certain about a detail, say you are not certain. Uncertainty is always better than confident misinformation.

---

TONE AND FORMAT:

- Be warm, direct, and student-friendly. Avoid jargon and bureaucratic language.
- Keep answers short: 2–4 sentences is ideal. Use a bullet list only when listing multiple distinct items.
- Always end with an action step: what the student should do next.
- If directing a student to an office, include the office name and (if available from the context) the email or phone number.
- Never write a response longer than 8 sentences.

---

CONTEXT:

{{KNOWLEDGE_BASE_CONTENT}}

---

Student question: {{STUDENT_QUESTION}}
```

---

## How to Use This Prompt in Code

When you call the Groq API, you will replace two placeholders:

- `{{KNOWLEDGE_BASE_CONTENT}}` — the text from the relevant knowledge base files, loaded at request time
- `{{STUDENT_QUESTION}}` — the actual message the student sent in Slack

**Example (Node.js):**

```js
const systemPrompt = basePrompt
  .replace('{{KNOWLEDGE_BASE_CONTENT}}', knowledgeBaseText)
  .replace('{{STUDENT_QUESTION}}', studentMessage);

const response = await groq.chat.completions.create({
  model: 'llama-3.3-70b-versatile',
  messages: [
    { role: 'system', content: systemPrompt },
    { role: 'user', content: studentMessage }
  ],
  temperature: 0.2,   // Low temperature = more predictable, factual responses
  max_tokens: 300     // Keep answers short — this enforces the 2-4 sentence rule
});
```

**Why `temperature: 0.2`?**
Lower temperature makes the model stick closer to the provided context and produce more consistent answers. Higher temperature increases creativity but also hallucination risk. For a grounded Q&A use case, low temperature is the right choice.

**Why `max_tokens: 300`?**
This enforces the short-answer behavior at the API level. Even if the model is tempted to write a long response, it cannot exceed ~225 words. This keeps Slack messages readable.

---

## Design Decisions

### Why explicit grounding rules?

Without the grounding rules, the AI model will confidently answer questions using information from its training data. This is dangerous for a university assistant because:

- Training data may contain policies from different universities
- Dates, phone numbers, and names in training data are often wrong or outdated
- Students may act on incorrect information (miss a deadline, contact the wrong office)

By explicitly forbidding the model from using training data and requiring it to say "I don't know" when the context doesn't contain the answer, we make the assistant trustworthy rather than just fluent.

### Why recommend an office when you don't know the answer?

"I don't know" alone is unhelpful. "I don't know, but the Financial Aid office at finaid@westbrook.edu can help" is actionable. The student is no worse off than before they asked, and possibly better off because they now have a specific contact.

### Why keep responses short?

Slack is a messaging tool, not a document viewer. Long responses get skimmed or ignored. A student who asked "when is the add/drop deadline?" wants a date and a next step, not a paragraph about academic policy. Short answers also feel more like a helpful person and less like a search engine.

### Why does the student question appear twice?

It appears once as `{{STUDENT_QUESTION}}` in the context block and once as a `user` message in the chat array. This is intentional. Putting it in the system prompt context helps the model match it against the knowledge base. Putting it as a user message follows the standard chat format and ensures the model knows it is responding to a direct question.

---

## Customizing for a Different University

To adapt Compass for a real or different fictional university:

1. Replace "Westbrook University" with the real university name throughout the prompt
2. Update `{{KNOWLEDGE_BASE_CONTENT}}` with real institutional documents (handbook, academic calendar, resource directory)
3. Update `data/campus_resources.json` with real contacts
4. Consider lowering `max_tokens` further (to 200) for even tighter answers, or raising it (to 400) if the knowledge base has complex policies that require more explanation

---

## What This Prompt Will NOT Handle Well

Be aware of these limitations:

- **Emotional distress** — The prompt is not designed for mental health crises. If a student expresses distress, the model may respond warmly but it is not trained as a counselor. Add a check in the handler code: if the question contains keywords like "crisis", "harm", or "emergency", bypass the AI entirely and return a hardcoded message with the crisis line number.
- **Ambiguous questions** — "What are my options?" with no context will produce a generic response. The prompt does not ask for clarification before answering.
- **Multi-turn conversations** — Each question is answered independently. The model does not remember what the student asked in a previous message.
