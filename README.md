Scalable Question Generation System (Gemini LLM)

I built an end-to-end, scalable pipeline that generates conceptual multiple-choice questions (MCQs) from long PDFs/TXT files using the Google Gemini API. The design follows the Savaal research pipeline (arXiv:2502.12477).

🔎 What it does

Chunk large documents with overlap for context continuity

Extract & rank main ideas with an LLM

Retrieve passages with Gemini embeddings + cosine similarity

Generate MCQs (1 correct + 3 plausible distractors) as strict JSON

Postprocess (validate, deduplicate, optional quality filtering)

Group outputs per document into a single all_questions.json

Repo structure
.
├── takehome_gemini_pipeline.ipynb     # Main notebook (end-to-end)
├── sample/                            # (optional) sample outputs/screens
└── README.md

Quick start (2 minutes)
1) Environment
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install google-generativeai pypdf tqdm numpy scikit-learn
# (Optional, more robust PDF parser)
# pip install pymupdf

2) Set your Gemini API key
import os
os.environ["GEMINI_API_KEY"] = "YOUR_KEY"

3) Open the notebook and run the last cell

Set:

CFG.input_files = ["Take Home Assignment.pdf", "2502.12477v2.pdf"]  # or your docs


Run:

grouped = run_all_grouped(CFG.input_files, out_path="all_questions.json")


Output: all_questions.json (one JSON, grouped per file)

Sample output (JSON)
{
  "Take Home Assignment": [
    {
      "question": "Which design choice in the pipeline most directly improves scalability on long documents?",
      "options": ["Chunking + retrieval", "Feeding full PDFs to the LLM", "Removing overlap", "Skipping embeddings"],
      "answer": "A"
    }
  ],
  "2502.12477v2": [
    {
      "question": "Why does the pipeline extract main ideas before question generation?",
      "options": ["Reduce cost and focus on concepts", "Increase token count", "Bypass retrieval", "Avoid using LLMs"],
      "answer": "A"
    }
  ]
}

How it works (high level)

Document ingestion & chunking

Split text into ~1800-char chunks with 200-char overlap to preserve context.

Main-idea extraction

Map (per chunk) → Combine/Reduce (deduplicate) → Rank (focus on core concepts).

Passage retrieval

Gemini embeddings → in-memory NearestNeighbors(metric="cosine", algorithm="brute").

MCQ generation

Strict JSON prompt; 1 correct + 3 plausible distractors.

Post-processing

Validate schema, deduplicate questions, optional LLM quality filter (score ≥3).

Output

Single grouped JSON across all input files.

Configuration

Key knobs (set in the notebook CFG):

Param	Default	Notes
max_chars_per_chunk	1800	Chunk size
chunk_overlap	200	Preserves continuity
gen_model	gemini-1.5-flash	LLM for ideas & MCQs
embed_model	models/text-embedding-004	Embeddings for retrieval
k_passages	3	Passages retrieved per idea
target_questions	20	Total questions per file
n_per_idea	2	MCQs generated per main idea
Why this design

Scalability: chunking + retrieval prevents feeding full PDFs to the LLM.

Quality: conceptual prompts, plausible distractors, schema validation, optional LLM judging.

Cost-efficiency: idea extraction first reduces downstream context size.

Simplicity: in-memory retriever is fast and submission-friendly; switchable to FAISS/Chroma if needed.

Evaluation criteria (how I meet them)

Functionality & Quality: End-to-end pipeline with JSON output; validates, deduplicates, and can filter for quality.

Scalability & Design: Fixed-size overlapping chunks; embeddings + cosine retrieval; grouped output for many files.

Code Quality: Modular functions, clear markdown explanations, debug logs, exception handling.

Communication: Notebook headings explain design choices; short demo flow; readable JSON preview.

Troubleshooting

“Ignoring wrong pointing object …” when reading PDF

These are harmless pypdf warnings. For better extraction, switch to PyMuPDF:

import fitz
with fitz.open(path) as doc:
    text = "\n".join([p.get_text("text") for p in doc])


Saved 0 questions

Check debug logs:

docs=… should be > 0 (PDF must yield text).

Parsed main ideas: … should be > 0.

Final emb matrix: (N, D) should show non-zero N and D≈768.

If QG fails JSON, the code attempts salvage; try re-running or reduce n_per_idea.

Rate limits / empty LLM output

Reduce target_questions, or run per file. Add small sleeps in retries if necessary.

Ethics & use

This generates practice questions from documents. Review for accuracy and bias before deploying in learning settings.

Acknowledgments

Savaal: Scalable Concept-Driven Question Generation to Enhance Human Learning (arXiv:2502.12477) — pipeline inspiration.
