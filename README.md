# Assignment 5: Augmented RAG Learner Guide Assistant (EDAB6808)

A Retrieval-Augmented Generation (RAG) chatbot, built in Google Colab, that answers questions about the **EDAB6808 Introduction to Data Analytics for Business Learner Guide**. The guide is the only knowledge source. Answers are copied from the guide's own wording and cited by page and section. Questions the guide cannot answer get a fixed refusal.

## Summary

The notebook indexes the 16-page learner guide and answers questions with these steps:

1. Extract and lightly clean the PDF text.
2. Split it into token-based chunks with page, section and unit metadata.
3. Embed the chunks and store them in a local vector database.
4. Retrieve evidence by combining dense search and keyword search, then rerank it.
5. Have a local LLM decide whether the evidence answers the question.
6. If it does, select the supporting source lines, so the answer is verbatim guide text with metadata-built citations.

It ends with an 18-question evaluation, a retrieval-method comparison, and a Gradio chatbot with evidence and review panels.

## Data

| Item | Detail |
|---|---|
| Knowledge source | EDAB6808 Learner Guide (16 PDF pages, 38 chunks) |
| Not indexed | The assignment brief and method documentation |
| Evaluation set | 18 questions: 10 direct, 4 multi-section, 4 unsupported, covering six guide sections |
| Persistence | Google Drive (`EDAB6808_RAG_Augmented`) for the PDF, extraction, chunks, results and a Qdrant backup |

## Methods

| Stage | Method |
|---|---|
| PDF extraction | PyMuPDF, with the original text retained; repeated page footers removed |
| Chunking | 220 BGE tokens with 40 overlap, bounded by page; headings and `Unit n:` captured as metadata |
| Embeddings | `BAAI/bge-small-en-v1.5` (384 dimensions, normalised, cosine similarity, BGE query prefix) |
| Vector store | Qdrant Local, rebuilt on each run, archived to Drive |
| Retrieval modes | `dense` (top 4); `dense_rerank` (top 15, same-page expansion, rerank); `hybrid` (dense 15 + BM25 15, Reciprocal Rank Fusion, expansion, rerank) |
| Reranker | `cross-encoder/ms-marco-MiniLM-L6-v2` |
| Generation model | `Qwen/Qwen2.5-1.5B-Instruct` (local, no fine-tuning) |
| Evidence gate | Qwen judges whether the retrieved excerpts answer every part of the question |
| Answering | Qwen selects labelled source lines; Python verifies exact wording and builds citations |
| Interface | Gradio app with a chatbot, a source-evidence panel, a human review tab and a retrieval comparison tab |
| Optional | RAGAS non-LLM context recall and precision (skipped by default) |

**Environment:** Tesla T4 GPU, Python 3.13, PyTorch 2.11, transformers 5.18, sentence-transformers 5.7, Qdrant client 1.19.

## Results

### Retrieval comparison (18 questions, k = 4)

| Method | Hit@4 | All-required-evidence@4 | Mean retrieval time |
|---|---|---|---|
| Dense | 85.7% | 85.7% | 0.023 s |
| Dense + rerank | 100% | 92.9% | 0.110 s |
| **Hybrid** (default) | 100% | 92.9% | 0.107 s |

Hybrid beats dense by **+7.1 percentage points** on all-required-evidence. It shows **no gain** over dense + rerank (+0.0 points). Rerank and expansion, not lexical fusion, produced the improvement on this test set.

### End-to-end evaluation

| Measure | Result |
|---|---|
| Abstention on unsupported questions (Q15–Q18) | 100% (4/4) |
| Pipeline errors | 2 (Q11 and Q12, both multi-section) |
| Mean end-to-end response time | 2.70 s on GPU |
| Persistence check (restored Drive backup, fresh process) | Passed |
| Source-copying and citation guard checks | Passed |
| Answer and citation accuracy | Pending the human quality review |

### Answerability gate (diagnostic)

| Question | Expected | Model decision |
|---|---|---|
| How much does Assignment 5 contribute to the module mark? | YES | YES |
| What is the exact submission deadline for Assignment 5? | NO | NO |
| What is the tuition fee for EDAB6808? | NO | NO |

## Limitations

- **Small evaluation:** 18 questions, with only four unsupported cases. The results do not establish refusal reliability or accuracy in general.
- **Answer accuracy not reported:** the human quality review of answers, evidence and citations was not completed. Retrieval and refusal figures are automated diagnostics.
- **Two pipeline errors:** both multi-section questions (Q11, Q12) failed, so multi-part answers are the weakest area.
- **Verbatim copying is not correctness:** exact wording and citations are verified, but the copied text can be irrelevant, incomplete or fragmented. For example, the Q02 answer returned a truncated passage ("hours: Student number as subject title…").
- **Small model:** Qwen2.5-1.5B can judge answerability wrongly and may fail to follow the output format. Reranker scores are ranking signals, not answerability probabilities.
- **PDF extraction:** tables, such as the assessment and schedule tables, can be fragmented. Heading detection relies on this guide's explicit structure and must be rechecked if the PDF changes.
- **Hybrid adds no measured benefit** over dense + rerank on this test set, and costs extra complexity.
- **Single-document scope:** the system cannot answer anything outside the learner guide by design. Deadlines, fees and pass marks are not in it.
- **Environment:** needs a Colab runtime with internet for model downloads. The Gradio share link is temporary (about one week) and exposes the review controls.
- **No fine-tuning:** no LoRA or other task-specific training was applied.

## Contributors

The notebook names no author or contributors. Add your name here before publishing.

| Contributor | Role |
|---|---|
| *Mamello Malape* | Author |
| *Fumane Malangabe* |  Author  |
| *Reatile Seekoei* |  Author  |

The learner guide is the course material of the University of the Free State EDAB6808 module (lecturers: Dr H. MvE Combrink, V. Oelofse, J. Trip).

## References

BGE-small-en-v1.5; Qwen2.5-1.5B-Instruct; rank_bm25; cross-encoder/ms-marco-MiniLM-L6-v2; Qdrant Python client; PyMuPDF; Gradio; RAGAS context recall and precision documentation.
