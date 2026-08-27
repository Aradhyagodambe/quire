# quire

A personal RAG (Retrieval-Augmented Generation) assistant that answers questions about your own PDFs — papers, notes, documentation — with citations back to the source document and page, using a fully local, open-source model.

![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)
![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)
![LlamaIndex](https://img.shields.io/badge/built%20with-LlamaIndex-6E56CF.svg)
![Model](https://img.shields.io/badge/model-Zephyr--7B--beta-orange.svg)

## Table of contents

- [What it does](#what-it-does)
- [Example](#example)
- [How it works](#how-it-works)
- [Why these choices](#why-these-choices)
- [Setup](#setup)
- [Status](#status)
- [License](#license)

## What it does

- Upload one or more PDFs and ask questions about their contents
- Answers are grounded in retrieved text, not the model's general knowledge
- Every answer comes with source citations (filename + page number)
- Search across *all* uploaded documents, or filter to one specific file
- Remembers conversation context, so a follow-up like "how does *this* paper address that?" resolves correctly

## Example

> **Q:** Summarize the abstracts and conclusions found in these documents.
>
> **A:** Document 1: "Research PPR 4" — Abstract: The paper proposes a model for anomaly detection in financial transactions using machine learning techniques, using a confusion matrix to evaluate effectiveness and addressing overfitting in the context of fraud detection...
>
> *Sources: research ppr 4.pdf (p.1) · research ppr 2.pdf (p.1) · research ppr 3.pdf (p.1)*

A follow-up restricted to one paper:

> **Q:** How does this specific paper address those themes?
>
> **A:** The paper "Research PPR 2" addresses the themes of machine learning techniques for fraud detection and data imbalance... the dataset is highly imbalanced, with only a small fraction of transactions flagged as fraudulent...
>
> *Sources: research ppr 2.pdf (p.1) — filtered to this document only*

## How it works

<img width="1672" height="475" alt="architecture" src="https://github.com/user-attachments/assets/5b825188-d5da-4ed6-9a88-868d09fec9e9" />


1. **Extraction** — PDFs are parsed with `pymupdf4llm`, which preserves structure (headings, lists) better than raw text extraction, at the page level
2. **Indexing** — pages are chunked and embedded with `sentence-transformers/all-MiniLM-L6-v2`, then indexed with LlamaIndex
3. **Retrieval** — a question is embedded the same way; the index returns the closest chunks, optionally restricted to specific source files via metadata filtering
4. **Reranking** — a cross-encoder (`cross-encoder/ms-marco-MiniLM-L-6-v2`) re-scores retrieved chunks for relevance, since embedding similarity alone is a coarse signal
5. **Generation** — [HuggingFaceH4/zephyr-7b-beta](https://huggingface.co/HuggingFaceH4/zephyr-7b-beta), 4-bit quantized, generates the answer — entirely locally, no external API calls

## Why these choices

- **Fully local inference** — no document content leaves the runtime it's running on
- **4-bit quantization** — makes a 7B model workable on a single T4 GPU (Colab's free tier, ~16GB VRAM)
- **Reranking as a second pass** — "close in embedding space" and "actually answers this question" are related but different things

## Setup

Built and tested on Google Colab with a T4 GPU.

1. Open `notebooks/quire.ipynb` in Colab and select a T4 (or better) GPU runtime
2. `pip install -r requirements.txt`
3. Run the cells in order — you'll be prompted to upload PDFs partway through
4. Ask questions in the cells at the bottom

## Status

Working prototype, actively being extended. Roadmap:

- [ ] Hallucination / "I don't know" detection
- [ ] A real evaluation set — retrieval precision@k, recall@k, answer correctness
- [ ] Quiz and flashcard generation from ingested documents
- [ ] A proper UI (currently notebook-only)

## License

[MIT](LICENSE) — see the `LICENSE` file for details.

Built with [LlamaIndex](https://www.llamaindex.ai/), [Zephyr-7B-beta](https://huggingface.co/HuggingFaceH4/zephyr-7b-beta), and [Sentence Transformers](https://www.sbert.net/).
