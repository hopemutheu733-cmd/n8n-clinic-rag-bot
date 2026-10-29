# Clinic RAG Assistant

AI-powered clinic assistant for patients and clinic staff who need fast, reliable answers from a trusted knowledge base.

This project builds a clinic-facing AI assistant that answers patient questions using only the clinic's own documentation, routes emergencies to staff, and supports appointment booking in a safe and structured way.

## Who this is for

This project is designed for:

- healthcare clinics
- patient support teams
- AI builders exploring safe RAG systems
- teams building operational chat assistants with guardrails

## Problem it solves

Many chatbots answer from general internet knowledge, which is risky in healthcare. Clinics need a system that:

- answers from trusted sources only
- routes urgent issues to human staff
- avoids unsafe medical advice
- captures booking requests in a structured way

## What this project does

This workflow:

- retrieves clinic knowledge from a stored document set
- searches that knowledge using embeddings and vector similarity
- sends the most relevant context to an AI agent
- answers only from retrieved clinic information
- flags urgent or out-of-scope requests
- stores booking requests for staff review
- sends email alerts when escalation is needed

## Why this matters

Healthcare is a high-risk domain, so AI needs to be:

- grounded
- constrained
- safe
- transparent

This project shows how RAG can be used with operational guardrails to make an assistant more useful and safer.

## Architecture

```text
Patient message
   ↓
n8n chat trigger
   ↓
Search relevant clinic docs in Supabase (pgvector)
   ↓
LLM agent receives message + retrieved context
   ↓
Decision:
   - INFO response
   - EMERGENCY escalation
   - HANDOFF
   - Book appointment request
   ↓
Supabase storage / Gmail alert / final reply
```

## Core workflow

### 1. Knowledge ingestion

Clinic knowledge is loaded into Supabase as vectorized chunks.

- Google Docs source
- chunking
- embeddings with Gemini
- vector storage in Supabase

![Clinic Knowledge Ingestion](Clinic%20Knowledge%20Ingestion.png)

### 2. Patient chat assistant

The chatbot:

- receives a patient question
- finds the most relevant knowledge snippets
- answers only from that context
- triggers escalation when needed

### 3. Appointment booking

When the patient asks to book:

- the bot requests missing information
- stores the appointment request in Supabase
- prevents booking during emergency situations

![Appointment booking](appointment%20booking.png)

![Appointment booking details](appointment%20booking%20details%202.png)

![Appointment booking info](appointment%20booking%20info%203.png)

![Appointment saved in Supabase](APPOINTMENT%20LEAD.PNG)

### 4. Staff alert system

If the response is classified as EMERGENCY or HANDOFF, staff are notified by email.

![Emergency escalation](emergency%20.png)

## Safety design

- no diagnosis or medication dosing
- emergency symptoms trigger a strong escalation message
- emergency or handoff cases never proceed with booking
- the system does not guess when information is missing
- the assistant tells the patient to contact a person when needed

## Example conversations

| Patient message | Response type | Outcome |
| --- | --- | --- |
| How much is a malaria test? | INFO | Answer from clinic knowledge base |
| I have severe chest pain and difficulty breathing | EMERGENCY | Urgent escalation and staff alert |
| Can you prescribe antibiotics for a cough? | HANDOFF | Escalate to staff |
| I want to book an appointment | INFO | Booking details collected |

## Tech stack

- n8n
- Supabase (pgvector)
- Google Gemini (embeddings)
- OpenRouter
- Google Docs
- Gmail
- JavaScript / workflow logic

## Repository structure

```text
.
├── 8a-clinic-knowledge-ingestion.json
├── 8b-clinic-rag-chat.json
├── README.md
├── Clinic Knowledge Ingestion.png
├── appointment booking.png
├── appointment booking details 2.png
├── appointment booking info 3.png
├── APPOINTMENT LEAD.PNG
└── emergency .png
```

## Setup

1. Import both workflow JSON files into n8n.
2. Add credentials for Supabase, Google Gemini, OpenRouter, Google Docs, and Gmail.
3. Enable the vector extension in Supabase.
4. Create the `appointments` table and vector store.
5. Add the knowledge base into a Google Doc.
6. Run the ingestion workflow.
7. Test the chat flow in n8n.

## Skills demonstrated

- RAG system design
- vector search
- workflow automation
- AI safety design
- operational chatbot design
- healthcare domain reasoning
- structured data handling

## Notes

- This is a portfolio project and not medical advice.
- The clinic information is fictional.
- Credentials are not stored in the repo.
- Re-running ingestion can duplicate vectors unless the table is cleared.

## Future improvements

- stronger data access controls (row-level security on the appointments table, service role key)
- an evaluation set to measure triage and answer accuracy
- 24/7 deployment on an always-on server instead of a demo setup
- WhatsApp integration through Meta's Cloud API

## Author

**Hope Mutheu** · [GitHub](https://github.com/hopemutheu733-cmd)
