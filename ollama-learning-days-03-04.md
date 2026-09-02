# Ollama Learning Journey --- Days 3 & 4 Reference

## Learning Goal

Build professional Ollama skills in short, hands-on sessions using
increasingly finance-focused examples. The capstone is a **Local AI
Stock Research Assistant**:

``` text
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

Financial examples are for learning and research purposes, not
investment advice.

## Session Workflow

Each learning day follows:

1.  Start a PowerShell transcript.
2.  Learn one small concept at a time.
3.  Run a hands-on command/test.
4.  Verify the result before moving forward.
5.  Stop the transcript at the end.

Start each day with:

``` powershell
Start-Transcript -Path ".\logs\day-XX-powershell.txt"
```

End with:

``` powershell
Stop-Transcript
```

**Important:** Remind me to start the transcript before beginning each
new learning day.

# Day 3 --- Ollama Model Lifecycle

## Objective

Understand how Ollama manages models locally:

``` text
Download → List → Inspect → Run → Stop / Timeout
```

## Models Observed

``` text
gemma4:latest       9.6 GB
gemma4:e2b          7.2 GB
llama3.2:latest     2.0 GB
```

## Core Commands

``` powershell
ollama list
```

Shows models installed locally on disk.

``` powershell
ollama ps
```

Shows models currently loaded for inference.

Key distinction:

``` text
ollama list = What models are installed?
ollama ps   = What models are loaded/running?
```

## Running and Unloading

``` powershell
ollama run llama3.2
```

Observed runtime:

``` text
SIZE        2.6 GB
PROCESSOR   100% GPU
CONTEXT     4096
UNTIL       ~5 minutes
```

Lifecycle observed:

``` text
Model on disk
    ↓
ollama run
    ↓
Loaded into GPU/RAM
    ↓
Inference
    ↓
Keep-alive
    ↓
Automatic unload
```

Manual unload:

``` powershell
ollama stop llama3.2
```

`ollama stop` releases runtime memory; it does **not** delete the
installed model.

## Inspecting a Model

``` powershell
ollama show llama3.2
```

Important metadata:

``` text
architecture        llama
parameters          3.2B
context length      131072
embedding length    3072
quantization        Q4_K_M
```

Key concepts:

-   **Architecture** --- underlying model family.
-   **Parameters** --- approximately 3.2 billion parameters.
-   **Context length** --- maximum supported context; runtime allocation
    can be smaller.
-   **Quantization Q4_K_M** --- 4-bit quantization variant that helps
    reduce memory/storage requirements.
-   **Capabilities: completion, tools** --- response generation plus
    tool/function-calling support.

# Day 4 --- Custom Models and Modelfiles

## Objective

Understand how Ollama customizes model behavior with a `Modelfile`.

Mental model:

``` text
Dockerfile                     Ollama Modelfile
---------                      ----------------
FROM python                    FROM llama3.2
Configuration                  SYSTEM / PARAMETER
      ↓                              ↓
Docker image                   Custom Ollama model
```

Inspect an existing definition:

``` powershell
ollama show --modelfile llama3.2
```

Important components observed:

``` text
FROM
TEMPLATE
PARAMETER
LICENSE
```

The `TEMPLATE` formats system/user/assistant/tool messages. We
intentionally did not modify it.

## First Custom Model Experiment

A temporary `devops-assistant` was created to learn the mechanics:

``` text
FROM llama3.2

SYSTEM """
You are a senior DevOps engineer.
Explain technical concepts clearly and concisely.
When possible, use practical examples.
"""
```

Build:

``` powershell
ollama create devops-assistant -f .\Modelfile
```

This demonstrated:

``` text
Base llama3.2
      +
SYSTEM instructions
      ↓
devops-assistant
```

This is **not training or fine-tuning**. It creates a customized
configuration around the existing base model.

The DevOps model was only a teaching experiment; the curriculum then
returned to the finance-focused goal.

# Finance-Focused Custom Model

Created:

``` text
Modelfile.finance
```

Built with:

``` powershell
ollama create market-researcher -f .\Modelfile.finance
```

Result:

``` text
market-researcher:latest
```

Concept:

``` text
llama3.2
   +
Finance SYSTEM prompt
   ↓
market-researcher
```

## Base vs Finance Model Test

Both models received the same synthetic input:

``` text
Ticker: SOFI
RSI: 72
Price: Above 50-day moving average
Volume: 1.8x average

Provide a concise technical assessment.
```

### Important Findings

The custom model produced more structured output, but both models still
made unsupported or incorrect interpretations.

Examples:

-   The base model incorrectly described `1.8x` average volume as being
    above `2x`.
-   The finance model interpreted elevated volume as selling pressure
    even though direction/order-flow information was not provided.
-   An earlier response characterized price above the 50-day moving
    average as evidence of a reversal without enough supporting
    information.

### Major Lesson

A SYSTEM prompt can improve role, formatting, and behavioral
constraints, but it does **not** guarantee correct financial reasoning.

``` text
Good SYSTEM prompt
        ↓
Better structure
        ↓
Still capable of bad inference
```

# Improved Finance Modelfile

Current `Modelfile.finance`:

``` text
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

Rebuild:

``` powershell
ollama create market-researcher -f .\Modelfile.finance
```

Verify:

``` powershell
ollama show market-researcher
```

# Capstone Architecture Lesson

Preferred architecture:

``` text
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
Interpret / synthesize
     ↓
Structured Research Brief
```

Example future fact packet:

``` json
{
  "ticker": "SOFI",
  "rsi": 72,
  "above_50dma": true,
  "relative_volume": 1.8,
  "volume_direction": "unknown"
}
```

Core design principle:

> **Use deterministic code for calculations and validation; use the LLM
> primarily for interpretation, synthesis, and explanation.**

Later lessons will build toward structured JSON input/output, REST API
integration, Python, Docker, RAG, and performance optimization.

# Commands Learned

``` powershell
ollama list
ollama ps
ollama run llama3.2
ollama stop llama3.2
ollama show llama3.2
ollama show --modelfile llama3.2
ollama create devops-assistant -f .\Modelfile
ollama create market-researcher -f .\Modelfile.finance
ollama show market-researcher
```

# Current Custom Models

``` text
devops-assistant:latest
market-researcher:latest
```

-   `devops-assistant` --- temporary learning experiment.
-   `market-researcher` --- finance-focused model aligned with the
    capstone.

# Next --- Day 5

Before starting Day 5:

``` powershell
Start-Transcript -Path ".\logs\day-05-powershell.txt"
```

Continue with finance-focused examples, one micro-concept at a time,
with hands-on verification before advancing.
