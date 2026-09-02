# Ollama Learning Journey

A hands-on journey to learn **Ollama and local Large Language Models (LLMs)** from fundamentals through production-style implementation.

The project uses practical examples from **technical and fundamental stock analysis** and will eventually evolve into a local AI-powered **Stock Research Assistant**.

## 🎯 Goal

Learn how to professionally use Ollama for:

- Running LLMs locally
- Selecting appropriate models
- Understanding model parameters and quantization
- Managing CPU, GPU, RAM, and VRAM usage
- Creating custom models with Modelfiles
- Calling Ollama through REST APIs
- Integrating Ollama with Python
- Running Ollama with Docker
- Using embeddings and RAG
- Optimizing local LLM performance
- Building a practical AI Stock Research Assistant

## 🖥️ Development Environment

Primary tools:

- Ollama
- VS Code
- Docker
- Python
- Git / GitHub
- Windows / PowerShell

### Hardware

Development workstation:

- CPU: Intel Core i7-14700HX
- Cores: 20
- Logical processors: 28
- RAM: ~32 GB
- GPU: NVIDIA RTX 1000 Ada Generation Laptop GPU
- GPU VRAM: ~4 GB

One goal of this project is to understand how model size, quantization, context length, RAM, and VRAM affect real-world Ollama performance.

---

# 📚 Learning Roadmap

| Module | Topic | Status |
| --- | --- | --- |
| 1 | Ollama Fundamentals | ✅ |
| 2 | Ollama CLI & Model Management | ✅ |
| 3 | Model Selection & Quantization | 🟢 In Progress |
| 4 | CPU, GPU, RAM & VRAM | 🟢 In Progress |
| 5 | Custom Modelfiles | 🟢 In Progress |
| 6 | Ollama REST API | ⬜ |
| 7 | Python Integration | ⬜ |
| 8 | Ollama + Docker | ⬜ |
| 9 | Embeddings & RAG | ⬜ |
| 10 | Production Engineering | ⬜ |
| 11 | Performance Optimization | ⬜ |
| 12 | Stock Research Assistant | ⬜ |

---

# 🧪 Progress

## Day 1 — Ollama Fundamentals ✅

Learned the basic Ollama architecture:

```text
User
  ↓
Ollama CLI
  ↓
Ollama Server
  ↓
Model Runtime
  ↓
Local LLM
  ↓
CPU / GPU
```

Commands practiced:

```powershell
ollama --version
ollama list
ollama run llama3.2
ollama ps
```

Key concepts:

- Ollama is the runtime/server, not the LLM itself.
- Models are downloaded and managed locally.
- `ollama list` shows locally available models.
- `ollama ps` shows models currently loaded.
- Ollama can use GPU acceleration automatically.
- `llama3.2` was verified running at `100% GPU`.

---

## Day 2 — Models & Quantization 🟢

Topics explored:

- Model parameter counts
- Billions of parameters (`B`)
- Quantization
- `Q4_K_M`
- Context length
- Disk size vs runtime memory
- GPU model loading
- Model capabilities

Models inspected:

### Llama 3.2

```text
Parameters:      3.2B
Quantization:    Q4_K_M
Context Length:  131072
Disk Size:       ~2 GB
Runtime:         100% GPU
```

### Gemma 4 E2B

```text
Parameters:      5.1B
Quantization:    Q4_K_M
Context Length:  131072
Disk Size:       ~7.2 GB
```

### Gemma 4 Latest

```text
Parameters:      8.0B
Quantization:    Q4_K_M
Context Length:  131072
Downloaded Size: ~9.6 GB
Runtime:         100% GPU
```

An accidental pull of the larger Gemma model became a useful experiment demonstrating that:

> **Model download size, parameter count, and runtime GPU memory are related but are not the same measurement.**

---

## Day 3 — Model Lifecycle & Runtime Management ✅

Learned how Ollama manages the lifecycle of locally installed models.

Commands practiced:

```powershell
ollama list
ollama ps
ollama run llama3.2
ollama stop llama3.2
ollama show llama3.2
```

Observed lifecycle:

```text
Model stored on disk
        ↓
ollama run
        ↓
Loaded into GPU / RAM
        ↓
Inference
        ↓
Keep-alive period
        ↓
Automatic unload or ollama stop
```

Key observations:

- `ollama list` shows models installed on disk.
- `ollama ps` shows models currently loaded for inference.
- `ollama stop` unloads a model from runtime memory without deleting it.
- Llama 3.2 used approximately `2.6 GB` at runtime and ran at `100% GPU`.
- Ollama automatically unloaded the model after its keep-alive period.
- Model disk size and runtime memory footprint are different measurements.
- `ollama show` exposes architecture, parameters, context length, quantization, and capabilities.

---

## Day 4 — Modelfiles & Finance-Focused Model Customization ✅

Learned how Ollama uses a `Modelfile` to customize the behavior of an existing model.

Inspected the base definition with:

```powershell
ollama show --modelfile llama3.2
```

Important Modelfile concepts introduced:

```text
FROM
SYSTEM
TEMPLATE
PARAMETER
LICENSE
```

A temporary `devops-assistant` model was first created to learn the mechanics of `ollama create`.

The exercise then moved to the project's finance focus by creating:

```text
market-researcher:latest
```

using:

```powershell
ollama create market-researcher -f .\Modelfile.finance
```

Current finance Modelfile:

```text
FROM llama3.2

SYSTEM """
You are a financial market research assistant.

Analyze only the facts provided by the user.

Rules:
- Do not invent missing market data.
- Do not calculate indicators unless explicitly requested.
- Separate bullish, bearish, and neutral observations.
- Highlight conflicting signals.
- Keep the assessment concise.
- End with a one-sentence overall assessment.
- Do not provide personalized investment advice.
"""
```

### Finance experiment

Both vanilla `llama3.2` and `market-researcher` were given:

```text
Ticker: SOFI
RSI: 72
Price: Above 50-day moving average
Volume: 1.8x average

Provide a concise technical assessment.
```

The custom model produced more structured output, but the experiment exposed an important limitation: **prompt engineering does not guarantee correct financial reasoning**.

This led to an important architecture decision:

```text
Market Data
     ↓
Python
     ↓
Calculate + validate indicators
     ↓
Structured facts
     ↓
Ollama
     ↓
Interpret + synthesize
     ↓
Structured Research Brief
```

Core principle:

> **Use deterministic code for financial calculations and validation. Use the LLM primarily for interpretation, synthesis, and explanation.**

This principle will guide later work with structured JSON, the Ollama REST API, Python, RAG, and the Stock Research Assistant.

---

# 📈 Finance-Focused Learning

Instead of generic AI examples, exercises will increasingly use financial-market scenarios.

Example:

```text
Market Data
     ↓
Technical Indicators
     ↓
Fundamental Data
     ↓
Python
     ↓
Ollama REST API
     ↓
Local LLM
     ↓
Structured Research Brief
```

Example prompt:

```text
RSI: 72
Price: Above 50-day moving average
Volume: 1.8x average

Provide a concise technical assessment.
```

These examples are for learning and research purposes and are not investment advice.

---

# 🚀 Capstone Project

The final project will be a:

## Local AI Stock Research Assistant

Potential architecture:

```text
Stock / Market Data
        ↓
Technical Analysis
        ↓
Fundamental Analysis
        ↓
Research Context
        ↓
Python Application
        ↓
Ollama REST API
        ↓
Local LLM
        ↓
Structured Stock Research Brief
```

The goal is to combine everything learned throughout the project:

- Ollama
- Local LLMs
- Model selection
- Modelfiles
- REST APIs
- Python
- Docker
- RAG
- Structured output
- Performance optimization

---

# 🗂️ Planned Repository Structure

```text
ollama-learning/
│
├── README.md
│
├── lessons/
│
├── examples/
│   ├── powershell/
│   └── python/
│
├── modelfiles/
│
├── docker/
│
└── stock-research-assistant/
```

---

# 🧠 Learning Philosophy

The project follows a simple rule:

> **Learn one concept → test it → verify it → move forward.**

Sessions are intentionally kept small and practical, targeting approximately **30 minutes per learning session**.

The objective is not simply to learn Ollama commands.

The objective is to understand enough about local AI infrastructure to **design, build, troubleshoot, optimize, and operate useful Ollama-based applications professionally.**
