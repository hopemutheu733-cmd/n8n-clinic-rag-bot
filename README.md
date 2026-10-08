# Clinic RAG Assistant (n8n + Supabase)

A retrieval-augmented chatbot that answers patient questions about a fictional clinic using only its own knowledge base. Every reply is tagged `INFO`, `EMERGENCY` or `HANDOFF`, so urgent or out-of-scope messages can be routed to a human.

## How it works

The project has two n8n workflows.

### 1. Knowledge ingestion (`8a-clinic-knowledge-ingestion.json`)

![Knowledge ingestion workflow](Clinic%20Knowledge%20Ingestion.png)

- Runs manually when the knowledge base changes.
- Reads the clinic knowledge base from a Google Doc.
- Splits the text into chunks with the Default Data Loader.
- Creates embeddings with Google Gemini.
- Stores the chunks and embeddings in Supabase (pgvector).
- Sample run: the clinic Google Doc was split into 5 chunks and stored in Supabase.

### 2. RAG chat (`8b-clinic-rag-chat.json`)

![Clinic RAG chat workflow](CLINIC_RAG_CHAT_B.png)

- Receives a patient message through the n8n chat trigger.
- Searches the Supabase vector store for the most relevant chunks.
- A JavaScript Code node processes the retrieved chunks before they reach the agent.
- Passes the chunks to an AI Agent (OpenRouter chat model, with simple memory).
- The agent answers using only the retrieved excerpts.

## Safety design

- Never diagnoses and never gives medicine doses.
- Emergency symptoms get an `EMERGENCY` reply that tells the patient to call 999 or 112 or go to the nearest hospital.
- Anything not in the knowledge base gets a `HANDOFF` reply instead of a guess.

## Example conversations

| Patient message | Reply type | Bot reply |
|---|---|---|
| How much is a malaria test? | INFO | INFO: A malaria test costs KES 300. |
| I have severe chest pain and I am struggling to breathe | EMERGENCY | EMERGENCY: Call 999 or 112 immediately, or go to the nearest hospital. Severe chest pain and difficulty breathing require urgent medical attention. |
| Can you prescribe me antibiotics for my cough? | HANDOFF | HANDOFF: I'll pass your request to the clinic team. Please call 0700 000 000 for a consultation. |

Prices, triage rules and contact details are stored in the knowledge base (Google Doc), retrieved from Supabase, and passed to the AI Agent as context. The bot answers only from that context.

## Tools

n8n Cloud, Supabase (pgvector), Google Gemini embeddings, OpenRouter, Google Docs

## Setup

1. Import both workflow files into n8n.
2. Add your own credentials for Supabase, Google Gemini, OpenRouter and Google Docs.
3. Enable the `vector` extension in Supabase and create the vector store table used by the workflow.
4. Put the clinic knowledge base in a Google Doc and link it in the ingestion workflow.
5. Run the ingestion workflow once to load the chunks.
6. Open the chat workflow and click **Open chat** to test.

## Workflow files

- `8a-clinic-knowledge-ingestion.json`
- `8b-clinic-rag-chat.json`

## Notes

- The clinic and its information are fictional. This is a portfolio project, not medical advice.
- Credentials are not included in the exported workflows.
- Re-running ingestion adds the chunks again, so clear the Supabase table before re-loading an updated document.

## Next

WhatsApp integration through Meta's Cloud API
