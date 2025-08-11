Hannoworks — Legal RAG Agent

Deliverables included

Jupyter Notebook / Colab script

System Design Document: this file (architecture diagram + module descriptions).

Sample queries & outputs: samples.json (includes the 2025 biometric case example and outputs generated).

PDF Legal Brief: legal_brief.pdf (formatted sample output from the notebook).

README.md: instructions to run locally / in Colab and how to push to GitHub.

1. High-level architecture (ASCII diagram)

              <img width="1772" height="346" alt="Screenshot 2025-08-11 025431" src="https://github.com/user-attachments/assets/31305595-8fc2-46d8-85ad-e669c43f776f" />
          

2. Module descriptions

2.1 Retriever

Role: Finds the most relevant precedents for a given legal query.

Tech: sentence-transformers to compute embedding vectors; faiss-cpu for vector index and nearest-neighbor search.

Input: Query string.

Output: Top-K precedent documents (id, title, summary, metadata).

Notes: Pre-compute embeddings for static corpora; store index file to disk for faster loading in Colab.

2.2 Embedder

Role: Convert text (cases and queries) into dense vectors.

Tech: all-MiniLM-L6-v2 (small and fast) or a larger Law-specific model if available.

Notes: Normalize vectors before adding to FAISS to use cosine similarity via inner-product index.

2.3 Vector DB (FAISS)

Role: Efficient similarity search over precedent embeddings.

Tech: faiss-cpu (Colab-friendly).

Notes: For large corpora consider Milvus/Weaviate/Pinecone for managed scaling.

2.4 Reasoner (LLM + Prompting)

Role: Perform legal reasoning using retrieved precedents.

Tech: API-based LLM (OpenAI / Google Gemini) or local open models (Mistral/LLaMA) when API unavailable.

Prompt design: Include case description + retrieved precedents + explicit tasks (list arguments, run proportionality test, suggest safeguards).

Fallback: A simulated template reasoner for demo runs when API keys or billing is unavailable.

2.5 Agent Orchestrator

Role: Implements agentic workflow — multi-step process: evidence retrieval, credibility checks, LLM reasoning, evidence re-check, final formatting.

Tech: Simple orchestrator in Python; optionally LangChain / LlamaIndex for richer tooling.

Behavior: If LLM claims a precedent says X, agent re-checks the retrieved text for that claim (tool-use verification).

2.6 Responder / Formatter

Role: Convert raw LLM output into submission-ready artifacts: (1) structured JSON, (2) human-readable brief, (3) PDF.

Tech: reportlab for PDF or python-docx for DOCX.

2.7 Feedback Loop (Optional)

Role: Collect user ratings or corrections to re-weight retrieval or improve prompt templates.

Data: Store tuples (query, retrieved_docs, llm_output, user_rating, corrections) in a small DB (SQLite / JSONL).

Use: Re-run embedding weighting (e.g., upweight docs that got high ratings) or refine prompts.

3. Data Flow (step-by-step)

User submits a case description (the 2025 biometric policy example).

Retriever encodes the query → FAISS returns top-K precedents.

Agent builds a reasoning prompt containing the query and the retrieved summaries.

Reasoner (LLM) is called with the prompt and returns: summaries, arguments for both sides, likely verdict, and suggested safeguards.

Responder formats the output (JSON + PDF) and returns to the user.

(Optional) User rates the answer; feedback stored and used later to refine system.

4. Sample Queries & Outputs

Sample Query (primary example):

In 2025, the government issues a policy mandating that all citizens must submit biometric data (fingerprints and iris scans) to access public services such as welfare disbursement and government IDs. The petitioner claims this violates their fundamental right to privacy under Article 21.

Sample Output (abbreviated):

1) Precedent summary:
- SC-2017-RightToPrivacy: Privacy is fundamental under Article 21; proportionality required.
- HC-2019-DataRetention: retention limited by scope/time/purpose.
- SC-2022-BiometricGuidelines: biometric collection may be permissible with consent & safeguards, but universal mandatory databases are risky.

2) Arguments for petitioner:
- Mandatory biometric submission is an intrusion into informational privacy (cite SC-2017-RightToPrivacy).
- Policy fails proportionality: it's overbroad and not the least restrictive means.

3) Arguments for government:
- Biometric data improves efficient delivery of services and reduces fraud.

4) Likely verdict:
- Court may strike down or read-down the policy as applied universally and mandatorily.

5) Suggested safeguards:
- Purpose limitation, data minimization, retention schedules, independent oversight.

Full JSON sample is in samples.json.

5. How to run (Colab instructions)

Open Colab → new notebook.

Upload hannoworks_legal_rag.py or copy notebook cells.

Install dependencies at top of notebook:

!pip install -q sentence-transformers faiss-cpu reportlab tqdm google-generativeai

(If using OpenAI) set environment variable:

import os
os.environ['OPENAI_API_KEY'] = 'sk-XXX'  # optional

(If using Google Gemini) configure:

import google.generativeai as genai
genai.configure(api_key='YOUR_GOOGLE_KEY')

Run cells in order. Output JSON and PDF will be saved in /content or /mnt/data.


requirements.txt (suggested):

sentence-transformers
faiss-cpu
reportlab
tqdm
google-generativeai
openai

6. Optional: Quick feedback-loop implementation (simple)

Add a small web form (Flask) or a notebook cell that asks: Was this helpful? (1-5) and Corrections:

Store responses in feedback.jsonl as newline JSON. Example entry:

{"query":"...","retrieved":["SC-2017..."],"rating":4,"correction":"Add case X"}

Periodically re-run an offline process that reads feedback.jsonl and:

Re-ranks documents in FAISS by adding synthetic embeddings of high-rated outputs.

Adds new case summaries to the corpus and reindexes.

7. What I will prepare for you next (if you want)

A ready-to-upload GitHub repo zip with all files above.

A polished system_design.md and a README.md tailored for submission.

A final PDF brief derived from your notebook output (styled).

Tell me which of these you want me to generate now and I will create the files (notebook + md + samples + pdf) ready for you to download and push to GitHub.

