# Clinic RAG Assistant (n8n + Supabase)

A RAG chatbot that answers patient questions about a fictional clinic using only its own knowledge base. Replies are tagged INFO, EMERGENCY or HANDOFF so urgent messages can be routed to a human.

## How it works
1. The ingestion workflow reads a Google Doc, splits it into chunks, creates embeddings (Google Gemini) and stores them in Supabase (pgvector).
2. The chat workflow searches Supabase for the most relevant chunks and passes them to an AI Agent, which answers using only those excerpts.

## Safety design
- Never diagnoses or gives medicine doses
- Emergency symptoms get an EMERGENCY reply
- Anything not in the knowledge base gets a HANDOFF reply

## Tools
n8n Cloud, Supabase (pgvector), Google Gemini embeddings, OpenRouter, Google Docs

## Workflows
- 8a-clinic-knowledge-ingestion.json
- 8b-clinic-rag-chat.json

## Next
WhatsApp integration through Meta's Cloud API
