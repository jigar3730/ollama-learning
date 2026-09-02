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
| 1 | Ollama Fundamentals | 🟢 In Progress |
| 2 | Ollama CLI & Model Management | ⬜ |
| 3 | Model Selection & Quantization | ⬜ |
| 4 | CPU, GPU, RAM & VRAM | ⬜ |
| 5 | Custom Modelfiles | ⬜ |
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
