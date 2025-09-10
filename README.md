# Scalable Question Generation System (Gemini LLM)

I built a **scalable question generation system** that ingests long documents (PDF/TXT) and produces **conceptual multiple-choice questions (MCQs)** with plausible distractors using the **Google Gemini API**.  
The design is inspired by the *Savaal* research pipeline (arXiv:2502.12477).

---

## 🔎 What it does
- Processes large documents by **chunking + retrieval**  
- Extracts and ranks **main ideas** using Gemini  
- Generates **MCQs in JSON format** (1 correct + 3 distractors)  
- Groups questions per document into a single clean output file  

---

## 🚀 Quick Start
```bash
pip install google-generativeai pypdf numpy scikit-learn tqdm
```
```python
import os
os.environ["GEMINI_API_KEY"] = "your-key"   # set API key

# In notebook:
CFG.input_files = ["Take Home Assignment.pdf", "2502.12477v2.pdf"]
grouped = run_all_grouped(CFG.input_files, out_path="all_questions.json")
```

Output example (`all_questions.json`):
```json
{
  "Take Home Assignment": [
    {"question": "Why is chunking used?", "options": ["Scalability", "Token inflation", "Skip embeddings", "Randomization"], "answer": "A"}
  ],
  "2502.12477v2": [
    {"question": "Why extract main ideas first?", "options": ["Reduce cost", "Increase tokens", "Bypass ranking", "Avoid retrieval"], "answer": "A"}
  ]
}
```

---

## ✅ Highlights
- **Scalable**: chunking + retrieval enables handling of long PDFs  
- **Quality-focused**: plausible distractors, JSON schema validation, optional LLM quality filter  
- **Clean design**: modular code, grouped JSON outputs, well-documented notebook  

---

## 📖 References
- *Savaal: Scalable Concept-Driven Question Generation* (Noorbakhsh et al., 2025)  
- Google Gemini API documentation  
