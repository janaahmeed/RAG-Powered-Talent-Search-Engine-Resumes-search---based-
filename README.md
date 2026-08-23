## Setup

```bash
python -m venv .venv
souce .venv/bin/activate # Windows: .venv\Scripts\activate

pip install -r requirements.txt
python -m spacy download en_core_web_sm

cp .env.example .env
# edit .env and set OPENAI_API_KEY
```
Run the app:

```bash
streamlit run streamlit_app.py
```
Run the tests:

```bash
pytest tests/unit_test.py -v
```


---

## Architecture

```
Resume PDFs / DOCX
        │
        ▼
Text extraction          (pypdf / python-docx)
        │
        ▼
Resume cleaning           (whitespace normalize)
        │
        ▼
Section parsing       (regex sections, skills, POS-based job titles)
        │
        ▼
NER enrichment              (spaCy → org / location / name)
        │
        ▼
CandidateProfile       (single Pydantic model, used everywhere below)
        │
        ▼
Embeddings + ChromaDB     (MiniLM sentence-transformer)
        │
        ▼
Natural-language query 
        │
        ▼
Top-10 semantic retrieval 
        │
        ▼
Hybrid rerank (semantic + BM25 keyword)  
        │
        ▼
Top 3
        │
        ▼
LLM evaluation            src/rag/generator.py + src/rag/prompts.py
        │
   ┌────┴────┐
   ▼         ▼
Match      Missing
explanation  skills
   │         │
   └────┬────┘
        ▼
  Streamlit UI            streamlit_app.py
        │
        ▼
Bias / fairness audit     bias_check

try on HuggingFace Dataset : yashpwr/resume-ner-training-data

embedding Model : `all-MiniLM-L6-v2`
spaCy (`en_core_web_sm`)
NER_TRANSFORMER_MODEL : yashpwr/resume-ner-bert-v2  finetuning on data  
LLM_MODEL=gpt-4o-mini
