# DocuBot Model Card

This model card is a short reflection on your DocuBot system.

---

## 1. System Overview

### What is DocuBot trying to do?

DocuBot is a lightweight Retrieval-Augmented Generation (RAG) assistant that answers questions about a project's documentation. It searches local documentation files for relevant information and, depending on the selected mode, either returns the retrieved evidence or uses Gemini to generate a grounded response.

### What inputs does DocuBot take?

- User question
- Documentation files stored in the `docs/` folder
- Gemini API key (for LLM modes)

### What outputs does DocuBot produce?

Depending on the selected mode, DocuBot returns either:
- A generated answer from Gemini,
- Retrieved documentation snippets,
- Or "I do not know based on these docs." when sufficient evidence cannot be found.

---

## 2. Retrieval Design

### How does your retrieval system work?

The retrieval system first loads all Markdown and text documents from the `docs/` folder. It builds a simple inverted index by converting each document into lowercase words and mapping each word to the documents that contain it.

For each query, the system:
- Converts the query to lowercase.
- Removes punctuation and common stop words.
- Scores each paragraph by counting how many meaningful query words appear in it.
- Returns the highest-scoring paragraphs as the retrieved snippets.

### What tradeoffs did you make?

The system favors simplicity over advanced search quality. Counting keyword matches is easy to understand and fast to implement, but it cannot recognize synonyms or understand semantic meaning. Retrieval quality depends heavily on exact word matches.

---

## 3. Use of the LLM (Gemini)

### When does DocuBot call the LLM and when does it not?

**Naive LLM mode**
- Sends the entire documentation corpus directly to Gemini without retrieval.

**Retrieval Only mode**
- Does not use Gemini.
- Returns only the retrieved documentation snippets.

**RAG mode**
- Uses the retrieval system to find relevant snippets.
- Sends only those snippets to Gemini to generate a grounded response.

### What instructions do you give the LLM to keep it grounded?

The LLM is instructed to:
- Use only the retrieved snippets when answering.
- Avoid making unsupported assumptions.
- Respond with "I do not know based on these docs." if the retrieved evidence is insufficient.
- Base answers only on the provided documentation.

---

## 4. Experiments and Comparisons

| Query | Naive LLM | Retrieval Only | RAG | Notes |
|------|-----------|----------------|-----|------|
| Where is the auth token generated? | Harmful | Helpful | Helpful | Naive mode failed to identify the correct location. Retrieval and RAG used documentation, although retrieval selected less relevant snippets. |
| How do I connect to the database? | Helpful | Helpful | Helpful | All modes found relevant information, but Retrieval and RAG relied directly on the documentation. |
| Which endpoint lists all users? | Helpful | Helpful | Helpful | Retrieval correctly found API documentation and RAG summarized it. |
| How does a client refresh an access token? | Helpful | Helpful | Helpful | Retrieval returned authentication-related documentation and RAG generated a grounded summary. |

### What patterns did you notice?

Naive LLM sometimes produced fluent responses that were weakly grounded or incomplete. Retrieval Only always showed evidence directly from the documentation but often lacked readability because it returned raw snippets. RAG combined retrieval with natural language generation, making answers easier to understand while remaining grounded in the retrieved documentation. However, RAG still depended on retrieval quality and could not recover information that was never retrieved.

---

## 5. Failure Cases and Guardrails

### Failure Case 1

**Question:** Where is the auth token generated?

**Observed behavior:** Retrieval selected general API token paragraphs instead of the paragraph describing `generate_access_token` in `AUTH.md`.

**Expected behavior:** Retrieval should have selected the most relevant authentication paragraph.

### Failure Case 2

**Question:** How do I process credit card payments?

**Observed behavior (before improvements):** Retrieval returned unrelated documentation because common words matched multiple documents.

**Expected behavior:** The system should refuse to answer because the documentation contains no payment information.

### When should DocuBot say "I do not know based on the docs I have"?

- When no retrieved paragraph contains meaningful evidence for the user's question.
- When the documentation does not discuss the requested topic.

### What guardrails did you implement?

- Ignored common stop words during scoring.
- Returned only paragraph-level snippets instead of entire documents.
- Refused to answer when no relevant evidence was found.
- Limited retrieval to the highest-scoring snippets.

---

## 6. Limitations and Future Improvements

### Current Limitations

1. Retrieval relies on exact keyword matching.
2. The system cannot understand synonyms or semantic similarity.
3. Retrieval sometimes selects less relevant paragraphs instead of the best evidence.

### Future Improvements

1. Use embeddings or semantic search instead of keyword matching.
2. Improve paragraph ranking using TF-IDF or BM25.
3. Highlight the exact matching sentences instead of entire paragraphs.

---

## 7. Responsible Use

### Where could this system cause real-world harm if used carelessly?

If used for medical, legal, financial, or security documentation, incorrect retrieval or incomplete evidence could cause users to make poor decisions. Users may also trust generated responses without verifying the original documentation.

### What instructions would you give real developers who want to use DocuBot safely?

- Always verify important answers against the original documentation.
- Treat generated responses as summaries, not absolute truth.
- Update the documentation regularly so retrieval stays accurate.
- Continue improving retrieval quality before deploying the system in production.

---

## Mode Comparison

### Naive LLM Mode

For the question "Where is the auth token generated?", the naive LLM incorrectly stated that the documentation did not provide enough information, even though the answer existed in `AUTH.md`. This demonstrated how an LLM can miss relevant details despite having access to the full corpus.

### Retrieval Only Mode

Retrieval Only returned documentation snippets rather than generating an answer. The information was grounded in the documentation, but the retrieved snippets were not always the most relevant and required the user to interpret them manually.

### RAG Mode

RAG generated a readable answer using only the retrieved snippets. It avoided unsupported claims, but because retrieval selected incomplete context, the generated answer was also incomplete.

### Refusal Behavior

For the question "How do I process credit card payments?", Retrieval Only returned "I do not know based on these docs." This demonstrated that the guardrail prevented unsupported answers when no meaningful evidence existed.

### Remaining Limitations

The quality of RAG depends directly on the quality of retrieval. If retrieval selects incomplete or less relevant context, the generated answer will also be incomplete. Improving retrieval would significantly improve overall system performance.