# IMF Kenya Report – LLM PDF-to-Dashboard Pipeline

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ParimiVenkatesh/imf-kenya-llm-pipeline/blob/main/IMF_Kenya_LLM_Pipeline.ipynb)

A Google Colab notebook that reads an IMF Country Report (PDF) and uses Large Language Models to turn it into summaries, structured JSON and a dashboard.

**Report used:** IMF Country Report No. 24/13 – *Kenya: 2023 Article IV Consultation, Sixth Reviews under the EFF and ECF Arrangements* (published January 2024). The 2025 Article IV consultation was postponed at the Kenyan authorities' request, so this is the latest Article IV report in the 2024–2025 period.

## How to run

1. Click **Open in Colab** above.
2. Click **Runtime → Run all**.

No files need to be uploaded and no API key is needed: the report PDF and the saved Gemini answers (`gemini_responses.json`) are downloaded automatically from this repository, so the notebook reproduces the published results.

*Optional:* to send new prompts, add a free Gemini key (<https://aistudio.google.com/apikey>) in Colab's 🔑 **Secrets** panel as `GEMINI_API_KEY`.

## What the notebook does

| Task | What happens |
|---|---|
| 1. PDF processing | `pypdf` extraction, header/footer removal, sentence-aware chunks of up to 320 MPNet tokens (at least one sentence of overlap), cleaning and chunking checks, executive summary, section summaries (<100 words), LLM-as-a-Judge scoring |
| 2. Structured output | 7 financial-stability theme counts (whole report and main chapters), growth drivers & vulnerabilities, core indicator JSON with `-1.0` fallbacks and logged reasons, schema validation, evidence-quote checks, cross-check against the report's Financial Soundness Indicators table, validated multi-year trends |
| 3. RAG | `all-mpnet-base-v2` embeddings + FAISS `IndexFlatL2`, BM25 keyword search and hybrid (reciprocal rank fusion); Hit@k / MRR on 14 gold questions; grounded Q&A scored by the judge and checked in Python |
| 4. Dashboard | `pd.melt` tidy data, 4-panel dashboard with an executive text overlay, saved at 300 dpi |
| Extension | `system_instruction` directive vs generic prompt: schema compliance, latency and judge accuracy |

## Design choices

- **Two models** – Gemini Flash extracts, Gemini Flash-Lite judges, so a model never marks its own work.
- **Local caching** – every Gemini reply is stored in one JSON file, so re-runs make no repeat API calls.
- **Chunk size** – 320 tokens, counted with the embedder's own tokenizer, keeps every chunk inside the 384 word-piece limit of `all-mpnet-base-v2`.
- **Judge checked, not trusted** – a planted-error test and plain-Python number checks run beside the LLM judge.
- **Schema enforcement** – JSON mode + Python type checks; missing values become `-1.0` with the reason logged.
- **Scale check** – a ratio such as `0.068` is converted to `6.8%` only when the report itself states the percent form.
- **Pinned versions** – `requirements.txt` pins `pypdf==7.0.0`; the cached answers are keyed on the exact extracted text.

## Outputs

- `outputs/*.json` – chunks, summaries, judge scores, theme counts, insights, indicators, fallback log, audits, trends, RAG results, directive benchmark
- `figures/*.png` – dashboard and extra charts (300 dpi)

## Licence

Code released under the [MIT License](LICENSE). The IMF report is © International Monetary Fund.

## References

- International Monetary Fund (2024). *Kenya: 2023 Article IV Consultation… Staff Report.* IMF Country Report No. 24/13. Washington, DC: IMF.
- Google (2026). Gemini API documentation. https://ai.google.dev
- Reimers, N. & Gurevych, I. (2019). Sentence-BERT. *EMNLP-IJCNLP*.
- Song, K. et al. (2020). MPNet: Masked and permuted pre-training for language understanding. *NeurIPS*.
- Johnson, J., Douze, M. & Jégou, H. (2021). Billion-scale similarity search with GPUs. *IEEE Transactions on Big Data*, 7(3), 535–547.
- Lewis, P. et al. (2020). Retrieval-augmented generation for knowledge-intensive NLP tasks. *NeurIPS*.
- Robertson, S. & Zaragoza, H. (2009). The probabilistic relevance framework: BM25 and beyond. *Foundations and Trends in Information Retrieval*, 3(4).
- Cormack, G.V., Clarke, C.L.A. & Büttcher, S. (2009). Reciprocal rank fusion outperforms Condorcet and individual rank learning methods. *SIGIR*.
- Zheng, L. et al. (2023). Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena. *NeurIPS*.
