# LLM API Call Flow & Sequence

This document provides a technical breakdown of every LLM API call made by PaperBanana. For each agent, we define the specific **Inputs** and **Outputs** to clarify the data flow.

---

## 1. Methodology Diagrams (`generate`)

### Phase 0: Pre-Processing
#### **Optimizer Agent** (Optional)
*   **Input**: Raw methodology text, vague figure caption.
*   **Output**: Structured methodology (bulleted/sectioned) and a sharpened communicative intent.

---

### Phase 1: Strategic Planning
#### **Retriever Agent**
*   **Input**: 
    *   *Target*: Methodology text + Figure caption.
    *   *Candidates*: A pool of ~30-300 candidates (each providing: ID, Caption, Truncated Snippet).
*   **Output**: List of **Top K Reference IDs** most visually and contextually relevant.

#### **Planner Agent**
*   **Input**: Methodology text, caption, and full data for selected Reference Examples (Captions + Visual Descriptions of those examples).
*   **Output**: A detailed textual **Layout Description** ($) and a recommended **Aspect Ratio**.

#### **Stylist Agent**
*   **Input**: Layout Description ($), academic style guidelines (e.g., NeurIPS, ACL).
*   **Output**: An **Aesthetic-Enhanced Description** (adds instructions for colors, line weights, and typography).

---

### Phase 2: Iterative Refinement
#### **Visualizer Agent** (Image Generation)
*   **Input**: Enhanced Description, Aspect Ratio, Seed.
*   **Output**: A rendered **Image file** (PNG/JPG/WebP).

#### **Critic Agent** (Vision-VLM)
*   **Input**: 
    *   *Visual*: The generated image.
    *   *Context*: The original methodology text + layout description.
*   **Output**: 
    *   `needs_revision`: Boolean (True/False).
    *   `critic_suggestions`: List of visual/logical errors.
    *   `revised_description`: An updated prompt for the next generation attempt.

---

### Phase 3: Finalization
#### **Structurer / IR Planner Agent** (Optional Vector Export)
*   **Input**: Final Layout Description, Methodology text.
*   **Output**: **Diagram IR** (JSON) — a structured graph of nodes, edges, and labels for Graphviz.

#### **Captioner Agent** (Optional)
*   **Input**: Final Image, original methodology text.
*   **Output**: A formal, publication-ready **Figure Caption**.

---

## 2. Statistical Plots (`plot`)

#### **Plotter Agent**
*   **Input**: Raw data (CSV/JSON strings), communicative intent.
*   **Output**: **Executable Python Code** (Matplotlib/Seaborn script).
*   *Note: This code is executed locally; the LLM does not generate the image directly.*

---

## 3. Paper Orchestration (`orchestrate`)

#### **Orchestration Planner**
*   **Input**: Full paper text/PDF.
*   **Output**: A **Generation Plan** (List of figure tasks, each with a text snippet, caption, and inferred type: Diagram vs. Plot).

---

## 4. Evaluation (`evaluate`)

#### **VLM Judge**
*   **Input**: Generated Image, Human Reference Image (Optional), Source Context, Caption.
*   **Output**: **Comparative Metrics** (Scores 1-100 for Faithfulness, Aesthetics, Conciseness, and Readability).

---

## Summary of LLM Usage Patterns

| Agent | Complexity | Quota Intensity | Model Recommendation |
| :--- | :--- | :--- | :--- |
| **Planner** | High | Low | Gemini 3 Pro (High Reasoning) |
| **Critic** | High | Medium | Gemini 2.5 Pro (High Vision) |
| **Visualizer** | Low | **High** | Gemini 3.1 Flash Image (Optimized Output) |
| **Retriever** | Medium | Low | Gemini 2.0 Flash (Speed) |
