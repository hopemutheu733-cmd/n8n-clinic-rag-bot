LangChain ICD-10 Assistant README
md
# ICD-10 Assistant

AI-powered ICD-10 coding support for clinicians, coders, and healthcare teams who need faster, more structured diagnosis suggestions from plain-language descriptions.

This project turns everyday symptom descriptions into likely ICD-10 diagnosis suggestions using a grounded AI pipeline. Instead of letting the model guess freely, it searches a real ICD-10 database first and only returns codes that are supported by the data.

## Who this is for

This project is designed for:
- medical coders
- healthcare professionals reviewing diagnosis codes
- AI builders exploring grounded clinical LLM workflows
- teams building healthcare software with real-world data constraints

## Problem it solves

Medical coding is slow, repetitive, and depends on highly specific terminology. People often describe symptoms in plain language, but ICD-10 coding requires precise, standardized terms.

This project reduces that friction by:
- converting natural language into clinical terms
- searching a real diagnosis dataset
- narrowing suggestions to valid ICD-10 codes
- explaining the recommendation in a clear, human-readable format

## What this project does

Given a description like:

```json
{
  "description": "burning pain when passing urine"
}
the app returns likely diagnosis suggestions such as:

JSON
{
  "clinical_terms": "Dysuria",
  "suggestions": [
    {
      "code": "R30.0",
      "description": "Dysuria",
      "reason": "Burning pain when passing urine matches dysuria."
    }
  ]
}
Why this matters
Medical AI needs to be grounded. A model that simply invents codes is unsafe and unreliable.

This project demonstrates a more practical pattern:

interpret the user input
retrieve real data from a trusted code table
let the model choose from valid candidates
return a response with reasoning
How it works
The user enters a plain-English medical description.
An LLM rewrites the description into more clinical terminology.
The system searches a MySQL ICD-10 dataset for matching candidates.
The model selects the strongest matches from the real data.
The final answer is returned along with short explanations.
Text
User input
   ↓
Rewrite step (LLM)
   ↓
Search ICD-10 database
   ↓
Candidate selection
   ↓
Validated response
Tech stack
Python
FastAPI
LangChain
OpenRouter
MySQL
HTML / frontend
Docker
Render
Key features
plain-English symptom-to-code workflow
grounded results from a real ICD-10 dataset
candidate filtering before final output
REST API + UI
evaluation script for example cases
deployment-ready container setup
Evaluation
I tested the app on 20 common symptom descriptions and tracked whether an acceptable code appeared in the suggestions.

Measure	Result
Acceptable code appeared in the suggestions	16 of 20 (80%)
First suggestion was acceptable	16 of 20
Known limitations:

keyword-based search can still miss unusual phrasing
the project is not a certified billing tool
it covers diagnosis coding, not full coding workflows
Safety and limitations
This is a portfolio project, not a final clinical coding tool.
Suggestions should be reviewed by a qualified coder.
Do not enter real patient data into the demo.
The project covers ICD-10 diagnosis codes only.
Project structure
Text
.
├── api.py
├── pipeline.py
├── chain.py
├── db.py
├── schema.sql
├── icd10.sql
├── evaluate.py
├── results.csv
├── index.html
├── Dockerfile
├── requirements.txt
├── .env.example
├── README.md
├── demo.png
└── .gitignore
Run locally
bash
git clone https://github.com/hopemutheu733-cmd/langchain-icd10-assistant.git
cd langchain-icd10-assistant
python -m pip install -r requirements.txt
Create a MySQL database, then run schema.sql and load icd10.sql.

Copy .env.example to .env and fill in your values:

env
MYSQL_HOST=localhost
MYSQL_USER=your_mysql_user
MYSQL_PORT=3306
MYSQL_PASSWORD=your_mysql_password
MYSQL_DATABASE=your_database_name
OPENROUTER_API_KEY=your_openrouter_key
OPENROUTER_MODEL=your_model_name
Then run:

bash
python -m uvicorn api:app --reload
Open:

http://127.0.0.1:8000
http://127.0.0.1:8000/docs
Skills demonstrated
AI / LLM engineering
prompt design
retrieval-augmented workflows
backend API development
healthcare domain modeling
evaluation and failure analysis
deployment configuration
Author
Hope Mutheu
