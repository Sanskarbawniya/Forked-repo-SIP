
# Master Architecture Document: Intelligent Talent Navigator (ITN)
## 1. Project Objective
The Intelligent Talent Navigator (ITN) is a retrieval-based candidate screening tool. It transitions candidate evaluation from strict keyword matching to semantic analysis. The system provides recruiters with a ranked candidate shortlist, stratified by confidence intervals, alongside explicit match justifications. It structurally isolates demographic variables to maintain evaluation neutrality and features automated feedback generation to improve the candidate experience.
## 2. Operational Flow
The system operates sequentially through six primary stages:
 1. **Document Intake:** The system ingests candidate resumes (PDF format), extracts raw text, and drops corrupted files.
 2. **Sanitization & Defense:** The text passes through the Bias Blindfold (redacting identifiable demographic data) and Adversarial Defense (flagging white-text manipulation).
 3. **Vectorization:** Clean text is chunked, processed through the embedding model, and indexed in the vector database.
 4. **Query & Evaluation:** The system queries the vector database against a provided job description. High-semantic matches are routed to the LLM for logical evaluation.
 5. **Skill Delta & Audit:** The reasoning engine identifies matched criteria and extracts the explicit "Skill Delta" (missing requirements).
 6. **Rendering & Output:** The user interface displays the ranked candidates. For Tier 3 candidates (rejected), the system triggers the Constructive Rejection Pipeline to generate an automated feedback report.
## 3. System Architecture Overview
The system utilizes a decoupled architecture, isolating the data processing engine from the evaluation layer. This separation guarantees low-latency processing, allows independent scaling of the background search index, and enables accurate telemetry tracking.
## 4. Core Modules
 * **Ingestion, Defense & Pre-processing:** Utilizes PyMuPDF for text extraction. Implements an **Adversarial Resume Defense** mechanism to detect hidden text manipulation. Integrates a **Bias Blindfold** to redact names, locations, and university identifiers.
 * **Embedding & Semantic Search:** Implements FAISS for in-memory similarity search and OpenAI text-embedding-3-small for vector generation.
 * **Reasoning & Audit Engine:** Calls the Groq API (Llama-3-70b) for evaluation. Generates an Audit Matrix detailing specific matched criteria and the Skill Delta.
 * **Constructive Rejection Pipeline:** An asynchronous function that formats the Skill Delta of Tier 3 candidates into professional, objective feedback drafts, mitigating the "ATS black hole" problem.
 * **Telemetry Monitor:** A background logging utility that tracks API token consumption and request latency, calculating the operational cost-per-screen.
 * **User Interface:** Developed with Streamlit. Utilizes @st.cache_data and streaming outputs to manage state and render data efficiently.
## 5. UI/UX Specifications (Presentation Differentiation)
To differentiate the project during demonstration, the user interface must implement the following technical features:
 * **Asynchronous Loading States:** Implement skeleton loaders during API calls. The interface must remain interactive while background processing occurs.
 * **Cost & Performance Dashboard:** Include a dedicated header metric showing "Current Run Cost" (derived from token usage) and "Average Processing Time." This demonstrates production readiness.
 * **Data Portability:** Include a "Generate Report" function allowing data export to CSV.
 * **Audit Log View:** Provide a dedicated tab showing the raw, sanitized text that the LLM evaluated to prove the efficacy of the Bias Blindfold.
## 6. Confidence-Based Ranking Logic
Candidates are ranked utilizing a weighted algorithm.
 * **Tier 1 (High Confidence):** S_{total} \ge 0.8
 * **Tier 2 (Needs Review):** 0.5 \le S_{total} < 0.8
 * **Tier 3 (Secondary):** S_{total} < 0.5
## 7. Project Structure
```text
intelligent-talent-navigator/
├── .env                    # Environment variables (excluded from version control)
├── requirements.txt        # Dependencies (streamlit, fastapi, langchain, faiss-cpu, pypdf, openai, groq)
├── main.py                 # Application entry point
├── core/                   # Processing logic: ingestion.py, defense.py, embeddings.py, ranking.py
├── api/                    # External integrations: groq_service.py
├── utils/                  # Helpers: telemetry.py, feedback_generator.py
├── ui/                     # Presentation layer: app.py, components/
├── data/                   # Local storage for documents and vector index
└── tests/                  # Unit and integration tests

```
## 8. Implementation Guidelines
 * **Execution Order:** The ingestion pipeline must execute the Bias Blindfold and Adversarial Defense algorithms prior to vector embedding.
 * **Exception Management:** Wrap all external API requests in try-except blocks. Surface specific error codes and retry mechanisms in the UI upon failure.
 * **State Management:** Utilize Streamlit's native threading to ensure the primary UI thread is not blocked during embedding or LLM inference operations.
 * **Telemetry Integration:** All functions within the api directory must log token usage to the telemetry.py module before returning responses.
<!-- Optional if we want to generate the whole project from AI Agent -->
## 9. Instructions for AI Builder Agent
> *"Act as a Senior Software Architect. Build the ITN project following the structure and protocols defined above. Write code and documentation using strictly formal, technical language. Prioritize architectural modularity—ensure the core logic is fully independent of the api service. Implement the Bias Blindfold (regex/NLP redaction), Adversarial Defense (hidden character detection), and Telemetry Monitor (token tracking). Implement the UI utilizing Streamlit, ensuring the incorporation of skeleton loaders, a token cost dashboard, and CSV export functionality. The UI must explicitly render the Audit Matrix generated by the reasoning engine and provide access to the feedback drafts generated by the Constructive Rejection Pipeline."*
> 
