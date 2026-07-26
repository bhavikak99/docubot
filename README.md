# DocuBot

DocuBot is a lightweight Retrieval-Augmented Generation (RAG) assistant that answers developer questions using local project documentation. It supports three different modes to demonstrate the differences between naive generation, document retrieval, and Retrieval-Augmented Generation (RAG).

The project uses Markdown documentation stored in the `docs/` folder to simulate a software project's internal documentation.

---

## Features

### 1. Naive LLM Mode
- Sends the entire documentation corpus directly to Gemini.
- Does not perform retrieval.
- Demonstrates how an LLM can produce fluent but sometimes weakly grounded answers.

### 2. Retrieval Only Mode
- Searches the documentation using a simple keyword-based retrieval system.
- Builds a basic inverted index.
- Scores document paragraphs using keyword matching.
- Returns the most relevant snippets without using an LLM.

### 3. RAG Mode
- Retrieves the most relevant documentation snippets first.
- Sends only those snippets to Gemini.
- Generates answers grounded in the retrieved evidence.

---

## Retrieval System

The retrieval pipeline consists of three stages:

1. **Index Construction**
   - Loads all Markdown documents from the `docs/` folder.
   - Builds a simple inverted index mapping words to the documents in which they appear.

2. **Document Scoring**
   - Converts queries to lowercase.
   - Removes punctuation and common stop words.
   - Scores document paragraphs by counting meaningful keyword matches.

3. **Snippet Retrieval**
   - Splits documents into paragraphs.
   - Returns the highest-scoring paragraphs instead of entire documents.
   - Returns "I do not know based on these docs." when no relevant evidence is found.

---

## Setup

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Configure environment variables

Create a `.env` file and add your Gemini API key:

```text
GEMINI_API_KEY=your_api_key_here
```

Gemini is only required for:
- Mode 1 (Naive LLM)
- Mode 3 (RAG)

Retrieval Only mode works without a Gemini API key.

---

## Running DocuBot

Start the application:

```bash
python3 main.py
```

Choose one of the available modes:

| Mode | Description |
|------|-------------|
| 1 | Naive LLM over the full documentation corpus |
| 2 | Retrieval Only (no LLM) |
| 3 | Retrieval-Augmented Generation (RAG) |

You can either:
- Press **Enter** to run the built-in sample queries, or
- Enter your own custom question.

---

## Project Structure

| File | Purpose |
|------|---------|
| `docubot.py` | Retrieval system, indexing, scoring, and RAG pipeline |
| `llm_client.py` | Gemini client and prompting logic |
| `dataset.py` | Sample evaluation queries |
| `evaluation.py` | Retrieval evaluation script |
| `docs/` | Project documentation used for retrieval |

---

## Guardrails

DocuBot includes several simple guardrails to improve reliability:

- Ignores common stop words during scoring.
- Retrieves paragraph-level snippets instead of entire documents.
- Refuses to answer when no meaningful evidence is found.
- RAG mode instructs Gemini to answer only from retrieved documentation.

---

## Requirements

- Python 3.9+
- `python-dotenv`
- Gemini API key (Modes 1 and 3 only)

No database or backend services are required. All documentation is stored locally inside the `docs/` folder.