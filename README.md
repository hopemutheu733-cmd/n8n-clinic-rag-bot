# Clinic RAG Assistant (n8n + Supabase)

An AI agent for a fictional clinic, built in n8n. It answers patient questions using only the clinic's own knowledge base (retrieval-augmented generation), saves appointment requests to Supabase, and emails clinic staff when a message needs a human.

Every reply is tagged `INFO`, `EMERGENCY` or `HANDOFF`, so urgent or out-of-scope messages can be routed to a human.

## How it works

The project has two n8n workflows. The chat workflow has an agent with a booking tool, followed by a staff alert.

### 1. Knowledge ingestion (`8a-clinic-knowledge-ingestion.json`)

![Knowledge ingestion workflow](Clinic%20Knowledge%20Ingestion.png)

- Runs manually when the knowledge base changes.
- Reads the clinic knowledge base from a Google Doc.
- Splits the text into chunks with the Default Data Loader.
- Creates embeddings with Google Gemini.
- Stores the chunks and embeddings in Supabase (pgvector).
- Sample run: the clinic Google Doc was split into 5 chunks and stored in Supabase.

### 2. RAG chat agent (`8b-clinic-rag-chat.json`)

![RAG chat agent workflow with booking tool and staff alert](canvas-with-alert.png)

- Receives a patient message through the n8n chat trigger.
- Searches the Supabase vector store for the most relevant chunks.
- A JavaScript Code node processes the retrieved chunks and passes them, with the patient's message, to the AI Agent.
- The AI Agent (OpenRouter chat model, simple memory) answers using only the retrieved excerpts.
- When a patient wants to book, the agent calls the **Book Appointment** tool, which saves the request to Supabase.
- After the agent, an If node checks the reply type. `EMERGENCY` and `HANDOFF` replies send an alert email to the clinic through Gmail. `INFO` replies send nothing.
- A final Edit Fields node returns the agent's reply to the patient, so the chat shows the answer instead of the email result.

### 3. Appointment booking (agent tool)

When a patient asks to book, the agent collects four details: full name, phone number, preferred date or time, and the reason for the visit. Once it has all four, it calls a Supabase tool that saves the request to an `appointments` table with status `pending`. The clinic confirms by phone.

**The agent with the Book Appointment tool attached**

![Agent with the Book Appointment tool](appointment%20booking%20details%202.png)

**Step 1: the agent asks for any missing details**

![Agent asking for missing details](appointment%20booking.png)

**Step 2: the patient gives the details and the booking is confirmed**

![Completed booking conversation](appointment%20booking%20info%203.png)

**Step 3: the request is saved in Supabase with status pending**

![Saved appointment in Supabase](APPOINTMENT%20LEAD%20.PNG)

**Emergency messages never create a booking**

![Emergency conversation](emergency%20.png)

### 4. Staff alert (If node + Gmail)

When the agent's reply starts with `EMERGENCY` or `HANDOFF`, the workflow emails the clinic with the bot's reply and the patient's original message. This covers emergencies, out-of-scope requests and failed bookings. Ordinary `INFO` replies, including successful bookings, send no email.

The alert is a separate step after the agent, not a tool the model chooses to call. That means the email is sent every time the condition is met, even if the model would not have decided to send it.

## Safety design

- Never diagnoses and never gives medicine doses.
- Emergency symptoms get an `EMERGENCY` reply that tells the patient to call 999 or 112 or go to the nearest hospital.
- Emergency symptoms never create a booking, even if the patient asks to book.
- Emergency and handoff replies automatically alert clinic staff by email.
- Anything not in the knowledge base gets a `HANDOFF` reply instead of a guess.
- If the booking tool fails, the agent does not claim the appointment was booked. It replies with a `HANDOFF` asking the patient to call the clinic, and staff are alerted.

## Example conversations

| Patient message | Reply type | Bot reply | Staff email |
| --- | --- | --- | --- |
| How much is a malaria test? | INFO | INFO: A malaria test costs KES 300. | No |
| I have severe chest pain and I am struggling to breathe | EMERGENCY | EMERGENCY: Call 999 or 112 immediately, or go to the nearest hospital. Severe chest pain and difficulty breathing require urgent medical attention. | Yes |
| Can you prescribe me antibiotics for my cough? | HANDOFF | HANDOFF: I'll pass your request to the clinic team. Please call 0700 000 000 for a consultation. | Yes |
| I'd like to book an appointment | INFO | INFO: To book an appointment I need your full name, phone number, preferred date or time, and the reason for the visit. Please provide these details. | No |
| My name is Test Patient, phone 0700000000, I'd like Monday morning for a general check-up | INFO | INFO: Your appointment request for a general check-up on Monday morning has been received. Our clinic will confirm the exact time by phone. | No |
| I have severe chest pain and can't breathe, can I book for next week? | EMERGENCY | EMERGENCY: Call 999 or 112 immediately, or go to the nearest hospital. Do not wait. | Yes |

Prices, triage rules and contact details are stored in the knowledge base (Google Doc), retrieved from Supabase, and passed to the AI Agent as context. The bot answers only from that context.

## Tools

n8n Cloud, Supabase (pgvector), Google Gemini embeddings, OpenRouter, Google Docs, Gmail

## Setup

1. Import both workflow files into n8n.
2. Add your own credentials for Supabase, Google Gemini, OpenRouter, Google Docs and Gmail.
3. Enable the `vector` extension in Supabase and create the vector store table used by the workflow.
4. Create an `appointments` table in Supabase with these columns: `id`, `patient_name`, `phone`, `preferred_date`, `reason`, `status` (default `pending`) and `created_at`. Make sure the Supabase credential used in n8n has permission to write to it.
5. Put the clinic knowledge base in a Google Doc and link it in the ingestion workflow.
6. Set the recipient address on the Gmail node to the clinic's staff inbox.
7. Run the ingestion workflow once to load the chunks.
8. Open the chat workflow and click **Open chat** to test.

## Workflow files

- `8a-clinic-knowledge-ingestion.json`
- `8b-clinic-rag-chat.json`

## Notes

- The clinic and its information are fictional. This is a portfolio project, not medical advice.
- Credentials are not included in the exported workflows.
- Re-running ingestion adds the chunks again, so clear the Supabase table before re-loading an updated document.
- Test bookings use fictional patient details only.
- In this demo, staff alerts go to the project owner's own inbox.

## Next

- Lock down the `appointments` table with Row Level Security and a service role key
- Add a test set of messages to measure how often reply types are correct
- Deploy n8n on a server so the bot runs without a laptop
- WhatsApp integration through Meta's Cloud API
