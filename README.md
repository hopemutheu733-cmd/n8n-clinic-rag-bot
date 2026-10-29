# Clinic RAG Assistant

AI-powered clinic assistant built with n8n, Supabase, and LLMs. The workflow answers patient questions using only the clinic's own knowledge base, flags urgent or out-of-scope requests, and can collect appointment details for staff review.

This project demonstrates a practical AI workflow pattern: grounded answers from trusted knowledge, human escalation for risk, and automated operations around the bot.

## Why this project matters

Many AI assistants fail because they answer from broad internet knowledge instead of trusted source material. In healthcare, that is not acceptable.

This project solves that by combining:

- a retrieval layer over a clinic knowledge base
- grounded answers from only stored excerpts
- triage rules for emergency and handoff cases
- appointment booking for routine service requests
- automatic alerts to clinic staff

## What it does

The assistant:

- answers patient questions using a clinic knowledge base stored in Supabase
- classifies each response as `INFO`, `EMERGENCY`, or `HANDOFF`
- blocks unsafe or out-of-scope advice
- collects appointment information when a patient wants to book
- saves booking requests in Supabase
- sends alerts to clinic staff when urgent or escalated cases need attention

## Architecture

```text
Patient message
   ↓
Chat trigger (n8n)
   ↓
Search relevant knowledge chunks in Supabase (pgvector)
   ↓
LLM agent receives context + patient message
   ↓
Decision:
   - INFO answer
   - EMERGENCY response
   - HANDOFF response
   - Book appointment request
   ↓
Supabase storage / Gmail alert / final patient reply
```

## Core workflow

### 1. Knowledge ingestion

Loads clinic documents into Supabase as vectorized chunks for retrieval.

- Google Docs source
- chunking and embeddings
- pgvector storage in Supabase
- refreshes when the knowledge base changes

![Knowledge ingestion workflow](Clinic%20Knowledge%20Ingestion.png)

### 2. RAG chat workflow

- receives patient messages through the n8n chat trigger
- searches the most relevant excerpts from Supabase
- passes those excerpts and the patient message to the AI agent
- answers only from the retrieved context
- triggers booking or escalation logic when needed

<img width="600" height="270" alt="RAG chat workflow in n8n" src="https://github.com/user-attachments/assets/4a22907b-2ede-4ac9-afdd-cda61ebaf612" />

### 3. Appointment booking tool

When a patient asks to book:

- the agent asks for required details if missing
- it collects name, phone, preferred date/time, and reason
- it stores the request in Supabase as pending
- it does not proceed with a booking if the case is urgent

**The agent with the Book Appointment tool attached**

![Agent with the Book Appointment tool](appointment%20booking%20details%202.png)

**Step 1: the agent asks for any missing details**

![Agent asking for missing details](appointment%20booking.png)

**Step 2: the patient gives the details and the booking is confirmed**

![Completed booking conversation](appointment%20booking%20info%203.png)

**Step 3: the request is saved in Supabase with status pending**

![Saved appointment in Supabase](APPOINTMENT%20LEAD%20.PNG)

### 4. Staff alert workflow

If the response is labeled `EMERGENCY` or `HANDOFF`, staff receive an email with the patient's message and the bot's reply.

<img width="600" height="270" alt="Full workflow canvas with the staff alert" src="https://github.com/user-attachments/assets/ba7e5a42-2b4c-4326-a3f2-6b4447edbff9" />

## Safety design

- no diagnosis or medication dosing
- emergency symptoms trigger a strong escalation message
- emergency or handoff cases never proceed with booking
- if the booking tool fails, the assistant does not claim the appointment was made
- the system responds with a handoff instead of guessing

**Emergency messages never create a booking**

![Emergency conversation](emergency%20.png)

## Example conversations

| Patient message | Reply type | Outcome |
| --- | --- | --- |
| How much is a malaria test? | `INFO` | Answer from clinic knowledge base |
| I have severe chest pain and I am struggling to breathe | `EMERGENCY` | Urgent escalation and staff alert |
| Can you prescribe me antibiotics for my cough? | `HANDOFF` | Escalate to staff |
| I'd like to book an appointment | `INFO` | Collect details and save booking request |

## Tech stack

n8n · Supabase · pgvector · Gemini embeddings · OpenRouter · Google Docs · Gmail · JavaScript / workflow logic

## Repository structure

```text
.
├── 8a-clinic-knowledge-ingestion.json
├── 8b-clinic-rag-chat.json
├── README.md
├── docs/
│   └── screenshots/
├── Clinic Knowledge Ingestion.png
├── appointment booking.png
├── appointment booking details 2.png
├── appointment booking info 3.png
├── APPOINTMENT LEAD .PNG
├── emergency .png
└── ...
```

## Setup

1. Import both workflow files into n8n.
2. Add credentials for Supabase, Google Gemini, OpenRouter, Google Docs, and Gmail.
3. Enable the vector extension in Supabase.
4. Create the appointments table and the vector store table.
5. Put the clinic knowledge base in a Google Doc.
6. Run the ingestion workflow once to load the knowledge chunks.
7. Open the chat workflow and test the assistant.

## Notes

- the clinic and its information are fictional
- this is a portfolio project and not medical advice
- credentials are not stored in the repo
- re-running ingestion adds duplicate chunks, so clear the vector table before reloading updated content

## Skills demonstrated

- LLM workflow orchestration
- RAG system design
- vector search and embeddings
- decision-making and triage logic
- AI product safety patterns
- workflow automation with n8n
- structured storage for appointments and service data

## Future improvements

- stronger row-level security for appointment data
- better evaluation of triage accuracy
- production deployment on a dedicated server
- WhatsApp or SMS integration
- more robust monitoring and retry logic

## Author

**Hope Mutheu** · [GitHub](https://github.com/hopemutheu733-cmd)
