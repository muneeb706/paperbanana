# PaperBanana Architecture

PaperBanana is an agentic framework for generating publication-quality academic illustrations. This document outlines the system architecture, the multi-agent pipeline, and the data flow between components.

## 1. High-Level Overview

PaperBanana transforms raw methodology text into illustrative diagrams using a multi-agent orchestration pattern. The system is designed to be provider-agnostic, supporting multiple VLM and Image Generation models through a unified interface.

### Key Architectural Layers:
- **CLI/Entry Point (`paperbanana/cli.py`)**: Handles command parsing, configuration loading (`Settings`), and triggers the orchestration pipeline.
- **Orchestration Core (`paperbanana/core/pipeline.py`)**: Manages the `PaperBananaPipeline`, which handles state transitions, retries, and sequential agent execution.
- **Multi-Agent Layer (`paperbanana/agents/`)**: Specialized agents that encapsulate specific roles (Retriever, Planner, Stylist, etc.).
- **Provider Layer (`paperbanana/providers/`)**: Abstracted interfaces for VLMs and Image Generators.
- **Data & Prompts (`prompts/`, `data/`)**: Centralized prompt templates and curated reference sets for in-context learning.

---

## 2. The Multi-Agent Pipeline

The generation process is divided into two distinct phases:

### Phase 0: Input Optimization (Optional)
**Agent**: `InputOptimizerAgent`
**Flow**: Processes raw input in parallel using two sub-tasks:
1. **Context Enricher**: Structures raw methodology text into a diagram-ready format.
2. **Caption Sharpener**: Refines vague communicative intents into precise visual specifications.

### Phase 1: Linear Planning
**Flow**: `Retriever` → `Planner` → `Stylist`
1. **RetrieverAgent**: Selects relevant reference examples from the curated set.
2. **PlannerAgent**: Produces a detailed textual description ($P$) and recommends an aspect ratio.
3. **StylistAgent**: Enhances the description for visual aesthetics based on conference style guidelines.

### Phase 2: Iterative Refinement
**Flow**: `Visualizer` ↔ `Critic` (Loop up to $N$ iterations)
1. **VisualizerAgent**: Renders the description into an image.
2. **CriticAgent**: A Vision-LLM that loads the generated image, evaluates it against the source context, and provides:
    - **Critique**: List of visual or logical errors.
    - **Revised Description**: An updated prompt for the next generation attempt.

#### Smart Termination Logic:
To optimize for both time and API costs, the pipeline implements an early-exit strategy:
- **Condition**: After each refinement, the `CriticAgent` evaluates if the current image faithfully represents the source methodology.
- **Action**: If the `Critic` is satisfied (`needs_revision=False`), the pipeline terminates immediately, even if the user-specified iteration limit ($N$) hasn't been reached.
- **Terminal State**: Reached when the Critic is satisfied or the maximum iteration limit ($N$) is hit.


---

## 3. Data Flow & State Management

PaperBanana uses Pydantic models for type-safe state transfer across the pipeline:

| Model | Source | Usage |
| :--- | :--- | :--- |
| `GenerationInput` | CLI | Initial text, caption, and config overrides. |
| `CritiqueResult` | Critic Agent | JSON-parsed feedback and revised descriptions. |
| `IterationRecord` | Pipeline | Tracking history of images and critiques for a single run. |
| `GenerationOutput` | Pipeline | Final image path, optimized description, and metadata. |

### Intermediate Representation (IR)
For methodology diagrams, the system can optionally use a **`StructurerAgent`** or **`IRPlannerAgent`** to produce a `DiagramIR` (JSON). This IR is then converted to **`DOT` (Graphviz)** for rendering high-quality vector exports (SVG/PDF).

---

## 4. Directory Structure Summary

```text
paperbanana/
├── agents/      # Specific agent implementations (BaseAgent wrappers)
├── core/        # Orchestration (pipeline.py), types, and utilities
├── evaluation/  # VLM-as-Judge benchmarking and evaluation logic
├── providers/   # VLM and Image Generation provider adapters
├── reference/   # Reference set storage and retrieval logic
└── vector/      # Graphviz/DOT rendering logic for vector exports
prompts/         # Text templates for all agent tasks and evaluations
```
