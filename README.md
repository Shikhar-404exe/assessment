# Automated Answer Sheet Evaluation
End-to-end pipeline in `answer_sheet_evaluation.ipynb` for grading digitized handwritten student exam PDFs against an answer key.
1. **Dataset Discovery & Exploration**: Automatically locates student PDFs, `answerkey.txt`, `Question.txt`, and `Teacher_manual_marks_Anonymized.csv` (read-only).
2. **PDF Ingestion & Preprocessing**: Converts PDF pages to images via PyMuPDF and applies OpenCV grayscale conversion, bounded skew correction (±3°), denoising, and CLAHE contrast enhancement.
3. **Handwritten Answer Extraction**: Sends preprocessed pages to `nvidia/nemotron-3.5-super-vl:free` via OpenRouter to return structured JSON preserving question numbers and page references without fabricating unreadable text.
4. **Resumable Checkpointing**: Caches preprocessed pages and per-page extraction/scoring JSONs under `outputs/`, retrying HTTP 429/5xx, malformed JSON, and failed pages individually.
5. **Rubric-Based Semantic Scoring**: Evaluates MCQs deterministically and batches short answers via Nemotron for concept-level partial marks (0.5-point steps) with concise reasoning.
6. **Structural Confidence Assessment**: Assigns `high`, `medium`, or `low` confidence from readability, answer completeness, question mapping, and score bounds, flagging uncertain items for review.
7. **Validation Against Teacher Marks**: Aligns automated totals with `Teacher_manual_marks_Anonymized.csv` to compute MAE, exact agreement (±0.5), per-question error, and student-wise score differences.
8. **Structured Export**: Writes final student marks, metrics, and per-question explanations to `outputs/results.json`, `outputs/results.csv`, and `outputs/student_marks.csv`.
9. **Setup**: Install `pymupdf`, `opencv-python`, `pandas`, `numpy`, `requests`, and `python-dotenv`, then configure `OPENROUTER_API_KEY` and `MODEL` in `.env`.
10. **Execution**: Run `answer_sheet_evaluation.ipynb` cells sequentially (`MAX_STUDENTS = 5` by default; validated on 5 students with MAE = 7.40).
