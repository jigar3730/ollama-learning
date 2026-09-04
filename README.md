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
| 2 | Models, Parameters & Quantization | ✅ |
| 3 | Tokens & Context Windows | ✅ |
| 4 | Model / Runtime Configuration | ✅ |
| 5 | Ollama REST API via PowerShell | ✅ |
| 6 | Python → Ollama REST API | ⏭️ Next |
| 7 | Structured JSON Output & Validation | ⬜ |
| 8 | Model Selection & Benchmarking | ⬜ |
| 9 | Hugging Face Ecosystem | ⬜ |
| 10 | GGUF, Quantization & Hugging Face → Ollama | ⬜ |
| 11 | Custom Ollama Modelfiles | ⬜ |
| 12 | Docker + Ollama | ⬜ |
| 13 | Embeddings | ⬜ |
| 14 | RAG Fundamentals | ⬜ |
| 15 | Finance-Document RAG | ⬜ |
| 16 | Performance Benchmarking & Optimization | ⬜ |
| 17 | Production Architecture, Logging & Security | ⬜ |
| 18 | Capstone — Local AI Stock Research Assistant | ⬜ |

> **Current checkpoint:** Day 5 complete. Next milestone: **Python → Ollama REST API**.

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

## Day 5 — PowerShell & Ollama REST API ✅

Moved beyond the Ollama CLI and successfully invoked the local Ollama server through its HTTP REST API.

Verified Ollama and the installed models with:

```powershell
curl.exe http://localhost:11434/api/tags
```

The main generation endpoint used was:

```text
POST http://localhost:11434/api/generate
```

### PowerShell REST call

```powershell
$request = @{
    model  = "market-researcher"
    prompt = "Analyze RSI 72"
    stream = $false
}

$body = $request | ConvertTo-Json

$result = Invoke-RestMethod `
    -Uri "http://localhost:11434/api/generate" `
    -Method Post `
    -ContentType "application/json" `
    -Body $body

$result.response
```

### PowerShell concepts practiced

- Variables, hashtables, pipelines, and object properties
- `ConvertTo-Json` and `ConvertFrom-Csv`
- `Invoke-RestMethod`
- Here-strings
- Numeric casting with `[double]`
- Rounding with `[math]::Round()`
- `if / elseif / else`
- Comparison operators such as `-gt`, `-lt`, `-ge`, and `-le`
- Logical operators such as `-and` and `-or`
- Nested PowerShell objects and JSON using `ConvertTo-Json -Depth`

### Finance exercise

Coiled Cobra scanner CSV data for APH and MNDY was parsed into PowerShell objects:

```powershell
$stocks = $csv | ConvertFrom-Csv
$stock = $stocks[0]
```

Instead of asking the LLM to calculate deterministic metrics, PowerShell calculated them:

```powershell
$entry = [double]$stock.'Stock Entry'
$stop = [double]$stock.'Stock Stop'
$target1 = [double]$stock.'Target 1'

$risk = [math]::Round($entry - $stop, 2)
$reward = [math]::Round($target1 - $entry, 2)
$rewardRisk = [math]::Round($reward / $risk, 2)
```

APH results:

```text
Risk/share:       4.10
Reward/Target 1:  8.20
Reward/Risk:      2.00
```

PowerShell also derived authoritative classifications:

```text
RSIState:     Neutral
VolumeState:  Below average
TrendState:   Price above both EMA20 and EMA50
```

### Structured fact packet

A nested fact packet was built and serialized before calling Ollama:

```text
Scanner CSV
     ↓
PowerShell parses data
     ↓
PowerShell calculates metrics
     ↓
PowerShell applies deterministic rules
     ↓
Structured JSON fact packet
     ↓
Ollama REST API
     ↓
market-researcher
     ↓
Grounded research narrative
```

Example:

```json
{
  "Symbol": "APH",
  "Setup": "SETUP_LONG",
  "Derived": {
    "TrendState": "Price above both EMA20 and EMA50",
    "RSIState": "Neutral",
    "VolumeState": "Below average",
    "RiskPerShare": 4.1,
    "RewardToTarget1": 8.2,
    "RewardRisk": 2
  }
}
```

Serialized with:

```powershell
$factJson = $factPacket | ConvertTo-Json -Depth 5
```

### Day 5 architecture lesson

```text
Raw Market Data
      ↓
Deterministic Code
  calculate + validate + classify
      ↓
Structured Fact Packet
      ↓
Ollama REST API
      ↓
Local LLM
      ↓
Interpret + summarize + explain
```

> **Code computes and validates facts. The LLM explains and synthesizes those facts.**

This pattern will carry directly into Python integration and eventually the Local AI Stock Research Assistant.

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

Preferred finance architecture:

```text
Market / Fundamental Data
        ↓
Quantitative & Technical Analysis
        ↓
Structured Fact Packet
        ↓
Python
        ↓
Ollama REST API
        ↓
Local LLM
        ↓
Grounded Stock Research Brief
```

Explicit grounding is required: the LLM should not invent meanings, units, thresholds, or definitions for proprietary financial metrics.

---

# 🤗 Hugging Face Learning Track

Hugging Face is now a formal part of the Ollama roadmap, focused on understanding, selecting, evaluating, converting, running, and eventually publishing models used with Ollama.

## Hugging Face Topics

Topics include model cards, Base vs Instruct models, architectures, parameter sizes, context lengths, licensing, evaluation and benchmarks, GGUF, quantization variants, local-hardware model selection, the Hugging Face → GGUF → Ollama workflow, and evaluation for financial-analysis workloads.

## Publishing Milestone

Eventually publish an **original project artifact** under `jigar3730`, such as a fine-tuned model, LoRA/adapter, permitted GGUF quantization, model card, evaluation results, usage examples, and intended-use/limitation documentation.

Before publishing, verify upstream licensing, redistribution rights, provenance, model-card requirements, versioning, evaluation methodology, and attribution. Existing models should not simply be republished and described as original project models.

---

# 🚀 Capstone Project

The final project will be a:

## Local AI Stock Research Assistant

Target architecture:

```text
Stock / Market Data
        ↓
Technical Analysis
        ↓
Fundamental Analysis
        ↓
Research Context
        ↓
Structured Fact Packet
        ↓
Python Application
        ↓
Ollama REST API
        ↓
Selected Local LLM
        ↓
Structured Stock Research Brief
```

The capstone will combine Ollama, local LLMs, Hugging Face, model selection, GGUF/quantization, Modelfiles, REST APIs, Python, Docker, structured output, embeddings, RAG, grounding, benchmarking, performance optimization, and production engineering.

Where useful, the same finance test cases will be run across multiple models so **quality, hallucination rate, latency, consistency, and resource utilization** can be compared objectively.

The LLM should operate primarily as an interpretation/research layer over deterministic financial calculations rather than inventing calculations, thresholds, units, or definitions for proprietary metrics.

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
