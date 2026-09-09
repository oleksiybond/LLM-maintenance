# Technical Analysis: Weak Points and Caveats

This document outlines the primary technical vulnerabilities and limitations of the Vessel Monitoring System.

## 1. Algorithmic & Model Weaknesses
- **Static Anomaly Thresholding**: The system uses a fixed threshold based on the 95th percentile of validation data. This does not account for varying operational regimes (e.g., transit vs. DP mode), which may lead to high False Positive Rates (FPR).
- **Simplistic Fault Injection**: CBM failure scenarios (e.g., "slow drift") are modeled as deterministic linear offsets. Real-world failures are typically stochastic and non-linear, meaning CBM benchmarks may overstate system effectiveness.
- **Lack of Online Adaptation**: There is no mechanism for the model to adapt to sensor drift or equipment aging without a full retraining cycle.

## 2. Data & Preprocessing Caveats
- **Fragile Data Cleaning**: Relying on a short forward-fill (`ffill`) for missing values means larger gaps are simply dropped. This can create temporal discontinuities that the Transformer may falsely flag as anomalies.
- **Normalization Drift**: The `StandardScaler` is static. Significant shifts in vessel power load over time could push features outside the original training distribution, degrading reconstruction accuracy.

## 3. LLM & Integration Risks
- **Causal Hallucination**: The LLM interprets anomalies based on contributing variables and internal knowledge. Without a formal causal model, the assistant may provide plausible but incorrect explanations for technical failures.
- **Local Service Dependency**: The system is strictly coupled to a local Ollama instance. Any failure in the Ollama service or the local hardware renders the primary user interface non-functional.

## 4. Architectural & Software Bottlenecks
- **Single-User State**: The use of global variables for the `detector`, `agent`, and `selected_time_index` in `src/app.py` makes the system non-thread-safe and unsuitable for multi-user deployment.
- **Blocking Computations**: Heavy GPU tasks (e.g., CBM Live Compute) are executed synchronously within the Gradio event loop, which can freeze the UI during processing.
- **In-Memory Data Loading**: Loading the entire dataset into memory via Pandas is not scalable for multi-year, high-frequency telemetry data.
