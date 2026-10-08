# Clinic RAG Assistant (n8n + Supabase)

A retrieval-augmented chatbot that answers patient questions about a fictional clinic using only its own knowledge base. Every reply is tagged `INFO`, `EMERGENCY` or `HANDOFF`, so urgent or out-of-scope messages can be routed to a human.

<img width="600" height="270" alt="CLINIC RAG CHAT B" src="https://github.com/user-attachments/assets/c9d0c5c4-5e57-458f-9707-a1ab53fa0d35" />


## How it works

The project has two n8n workflows.

**1. Knowledge ingestion** (`8a-clinic-knowledge-ingestion.json`)
- Reads the clinic knowledge base from a Google Doc.
- Splits the text into chunks.
- Creates embeddings with Google Gemini.
- Stores the chunks and embeddings in Supabase (pgvector).

**2. RAG chat** (`8b-clinic-rag-chat.json`)
- Receives a patient message through the n8n chat trigger.
- Searches the Supabase vector store for the most relevant chunks.
- Passes those chunks to an AI Agent (OpenRouter chat model, with simple memory).
- The agent answers using only the retrieved excerpts.
- A JavaScript Code node processes the reply before it is returned.

## Safety design

- Never diagnoses and never gives medicine doses.
- Emergency symptoms get an `EMERGENCY` reply.
- Anything not in the knowledge base gets a `HANDOFF` reply instead of a guess.

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

## Next

WhatsApp integration through Meta's Cloud API
