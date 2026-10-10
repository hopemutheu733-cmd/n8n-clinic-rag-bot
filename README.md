# Clinic RAG Assistant (n8n + Supabase)

An AI agent for a fictional clinic, built in n8n. It answers patient questions using only the clinic's own knowledge base (retrieval-augmented generation) and can book appointment requests by saving them to Supabase.

Every reply is tagged `INFO`, `EMERGENCY` or `HANDOFF`, so urgent or out-of-scope messages can be routed to a human.

## How it works

The project has two n8n workflows. The chat workflow has an agent with a booking tool.

### 1. Knowledge ingestion (`8a-clinic-knowledge-ingestion.json`)

![Knowledge ingestion workflow](Clinic%20Knowledge%20Ingestion.png)

- Runs manually when the knowledge base changes.
- Reads the clinic knowledge base from a Google Doc.
- Splits the text into chunks with the Default Data Loader.
- Creates embeddings with Google Gemini.
- Stores the chunks and embeddings in Supabase (pgvector).
- Sample run: the clinic Google Doc was split into 5 chunks and stored in Supabase.

### 2. RAG chat agent (`8b-clinic-rag-chat.json`)

![RAG chat agent workflow with the Book Appointment tool](appointment_booking_info_3.png)

- Receives a patient message through the n8n chat trigger.
- Searches the Supabase vector store for the most relevant chunks.
- A JavaScript Code node processes the retrieved chunks and passes them, with the patient's message, to the AI Agent.
- The AI Agent (OpenRouter chat model, simple memory) answers using only the retrieved excerpts.
- When a patient wants to book, the agent calls the **Book Appointment** tool, which saves the request to Supabase.

### 3. Appointment booking (agent tool)

When a patient asks to book, the agent collects four details: full name, phone number, preferred date or time, and the reason for the visit. Once it has all four, it calls a Supabase tool that saves the request to an `appointments` table with status `pending`. The clinic confirms by phone.

**The agent asks for any missing details**

![Agent asking for missing details](appointment_booking.png)

**The saved row in Supabase**

![Saved appointment in Supabase](APPOINTMENT_LEAD_.PNG)

**An emergency message: no booking is created**

![Emergency conversation](emergency_.png)

## Safety design

- Never diagnoses and never gives medicine doses.
- Emergency symptoms get an `EMERGENCY` reply that tells the patient to call 999 or 112 or go to the nearest hospital.
- Emergency symptoms never create a booking, even if the patient asks to book.
- Anything not in the knowledge base gets a `HANDOFF` reply instead of a guess.
- If the booking tool fails, the agent does not claim the appointment was booked. It replies with a `HANDOFF` asking the patient to call the clinic.

## Example conversations

| Patient message | Reply type | Bot reply |
| --- | --- | --- |
| How much is a malaria test? | INFO | INFO: A malaria test costs KES 300. |
| I have severe chest pain and I am struggling to breathe | EMERGENCY | EMERGENCY: Call 999 or 112 immediately, or go to the nearest hospital. Severe chest pain and difficulty breathing require urgent medical attention. |
| Can you prescribe me antibiotics for my cough? | HANDOFF | HANDOFF: I'll pass your request to the clinic team. Please call 0700 000 000 for a consultation. |
| My name is Test Patient, phone 0700000000, I'd like Monday morning for a general check-up | INFO | INFO: Your appointment for a general check-up on Monday morning is booked. The clinic will confirm by phone. |

Prices, triage rules and contact details are stored in the knowledge base (Google Doc), retrieved from Supabase, and passed to the AI Agent as context. The bot answers only from that context.

## Tools

n8n Cloud, Supabase (pgvector), Google Gemini embeddings, OpenRouter, Google Docs

## Setup

1. Import both workflow files into n8n.
2. Add your own credentials for Supabase, Google Gemini, OpenRouter and Google Docs.
3. Enable the `vector` extension in Supabase and create the vector store table used by the workflow.
4. Create an `appointments` table in Supabase with these columns: `id`, `patient_name`, `phone`, `preferred_date`, `reason`, `status` (default `pending`) and `created_at`. Make sure the Supabase credential used in n8n has permission to write to it.
5. Put the clinic knowledge base in a Google Doc and link it in the ingestion workflow.
6. Run the ingestion workflow once to load the chunks.
7. Open the chat workflow and click **Open chat** to test.

## Workflow files

- `8a-clinic-knowledge-ingestion.json`
- `8b-clinic-rag-chat.json`

## Notes

- The clinic and its information are fictional. This is a portfolio project, not medical advice.
- Credentials are not included in the exported workflows.
- Re-running ingestion adds the chunks again, so clear the Supabase table before re-loading an updated document.
- Test bookings use fictional patient details only.

## Next

- Staff alert for emergencies and failed bookings
- Deploy n8n on a server so the bot runs without a laptop
- WhatsApp integration through Meta's Cloud API
