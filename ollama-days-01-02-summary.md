# Ollama Learning Journey --- Days 1--2 Summary

**Date:** September 1, 2026\
**Project:** Ollama Learning Journey\
**Learning format:** \~30-minute micro-lessons with hands-on
verification\
**Domain focus:** Technical and fundamental stock analysis

------------------------------------------------------------------------

## Project Objective

Learn Ollama professionally from fundamentals through production-style
implementation.

The learning journey will use finance and stock-analysis examples
wherever practical. The eventual capstone is a **Local AI Stock Research
Assistant** using Ollama, Python, REST APIs, Docker, structured data,
and eventually RAG.

The learning rule is:

> **Learn one concept → test it → verify it → move forward.**

------------------------------------------------------------------------

# Environment

## Workstation

-   CPU: Intel Core i7-14700HX
-   CPU cores: 20
-   Logical processors: 28
-   RAM: \~32 GB
-   GPU: NVIDIA RTX 1000 Ada Generation Laptop GPU
-   GPU VRAM: \~4 GB
-   OS: Windows
-   Shell: PowerShell

## Development Tools

-   Ollama
-   VS Code
-   Docker
-   Git
-   GitHub
-   Python

## Ollama Version

``` powershell
ollama --version
```

Observed:

``` text
ollama version is 0.33.2
```

------------------------------------------------------------------------

# Day 1 --- Ollama Fundamentals

## Core Mental Model

Ollama is the **local model runtime/server**, not the AI model itself.

``` text
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

Examples of models include Llama, Gemma, Qwen, etc.

A useful Docker analogy:

``` text
docker images ≈ ollama list
docker ps     ≈ ollama ps
```

This is only a mental model; the technologies are not technically
identical.

## Commands Learned

### Check Ollama version

``` powershell
ollama --version
```

### List downloaded models

``` powershell
ollama list
```

Initial models:

``` text
NAME               SIZE
gemma4:e2b         7.2 GB
llama3.2:latest    2.0 GB
```

### See currently loaded models

``` powershell
ollama ps
```

Initially no models were loaded.

### Run a model

``` powershell
ollama run llama3.2
```

Finance test prompt:

``` text
Explain RSI in one sentence.
```

Llama correctly explained RSI as a technical-analysis indicator used to
assess recent price momentum and potential overbought/oversold
conditions.

While the model was running:

``` powershell
ollama ps
```

Observed:

``` text
NAME               SIZE      PROCESSOR    CONTEXT
llama3.2:latest    2.6 GB    100% GPU     4096
```

This demonstrated that Ollama automatically used the NVIDIA GPU and that
this model fit entirely on the GPU for the tested configuration.

### Exit an interactive model

``` text
/bye
```

After exiting:

``` powershell
ollama ps
```

returned no active model.

## Day 1 Key Takeaways

-   `Ollama` = runtime/server
-   `llama3.2` = model
-   `ollama list` = models stored locally
-   `ollama ps` = models currently loaded
-   Ollama can automatically use GPU acceleration
-   A model can remain loaded temporarily for subsequent requests
-   `/bye` ended the tested interactive session and the model was no
    longer shown by `ollama ps`

------------------------------------------------------------------------

# Day 2 --- Models, Parameters and Quantization

## Parameters

Model sizes are commonly described as:

``` text
3B
7B
8B
14B
32B
70B
```

`B` means **billions of parameters**.

Parameters are numerical weights learned during model training.

More parameters can increase capability, but usually also increase
compute and memory requirements. Bigger is not automatically better for
every task.

The engineering goal should be:

> **Use the smallest model that performs the required task well
> enough.**

------------------------------------------------------------------------

## Quantization

A 3.2-billion-parameter model stored entirely as straightforward 32-bit
floating-point values would require roughly:

``` text
3.2 billion × 4 bytes ≈ 12.8 GB
```

Yet the local Llama model occupies only about 2 GB on disk.

The major reason is **quantization**.

Quantization reduces the numerical precision used to represent model
weights, substantially reducing model size and memory requirements, with
a possible quality trade-off.

Conceptually:

``` text
Higher precision
      ↓
More storage/memory/compute

Quantization
      ↓
Smaller model
Lower memory requirements
Often faster locally
Potential quality trade-off
```

------------------------------------------------------------------------

# Inspecting Models

## Llama 3.2

Command:

``` powershell
ollama show llama3.2
```

Observed:

``` text
architecture        llama
parameters          3.2B
context length      131072
embedding length    3072
quantization        Q4_K_M
```

Capabilities:

``` text
completion
tools
```

Important observation:

``` text
Model-supported context: 131072
Observed running context: 4096
```

The model's supported context length and the context configured/used by
a particular Ollama run are separate concepts.

------------------------------------------------------------------------

## Gemma 4 E2B

Command:

``` powershell
ollama show gemma4:e2b
```

Observed:

``` text
architecture        gemma4
parameters          5.1B
context length      131072
embedding length    1536
quantization        Q4_K_M
requires            0.20.0
```

Capabilities:

``` text
completion
vision
audio
tools
thinking
```

Downloaded size:

``` text
7.2 GB
```

------------------------------------------------------------------------

# Happy Accident --- gemma4:latest

While intending to use `gemma4:e2b`, the command below was accidentally
executed:

``` powershell
ollama run gemma4
```

This pulled another model:

``` text
gemma4:latest
```

Downloaded size:

``` text
9.6 GB
```

Rather than deleting it, the model became a useful experiment.

Command:

``` powershell
ollama show gemma4
```

Observed:

``` text
architecture        gemma4
parameters          8.0B
context length      131072
embedding length    2560
quantization        Q4_K_M
requires            0.20.0
```

Capabilities:

``` text
completion
vision
audio
tools
thinking
```

------------------------------------------------------------------------

# GPU Experiment

A finance prompt was sent to `gemma4:latest`:

``` text
RSI is 72, price is above the 50-day moving average,
and volume is 1.8x average.
Give me a one-sentence technical assessment.
```

Gemma identified:

-   bullish momentum
-   confirmation from elevated volume
-   price above the 50-day moving average
-   caution because RSI above 70 may indicate an overbought condition

While running:

``` powershell
ollama ps
```

Observed:

``` text
NAME             SIZE      PROCESSOR    CONTEXT
gemma4:latest    3.2 GB    100% GPU     4096
```

This was an important result.

The original expectation was that a 9.6 GB downloaded model might
require a CPU/GPU split because the workstation has roughly 4 GB VRAM.

Instead, Ollama reported:

``` text
100% GPU
```

## Important Lesson

These measurements are related but **not interchangeable**:

``` text
Parameter count
      ↓
Model scale/complexity

Quantization
      ↓
How compactly weights are represented

Disk/download size
      ↓
Storage occupied by the model artifacts

Runtime memory
      ↓
Resources actually used while running
```

Do not assume that a model's download size equals its VRAM requirement.

------------------------------------------------------------------------

# Model Tags

The accidental download introduced the concept of **tags**.

These commands are different:

``` powershell
ollama run gemma4:e2b
ollama run gemma4
```

Without an explicit tag:

``` text
gemma4
```

resolved to:

``` text
gemma4:latest
```

This is similar conceptually to Docker tags:

``` text
docker pull nginx:latest
docker pull nginx:1.27
```

For experimentation, `latest` is convenient.

For controlled/repeatable applications, prefer explicit model
versions/variants when available so behavior is less likely to change
unexpectedly.

Final local model list:

``` text
NAME               SIZE
gemma4:latest      9.6 GB
gemma4:e2b         7.2 GB
llama3.2:latest    2.0 GB
```

------------------------------------------------------------------------

# Finance-Focused Model Selection

The objective is **not** to always run the largest model.

Different tasks may justify different model capabilities:

``` text
Extract ticker from text
        ↓
Small / fast model

Classify a setup
        ↓
Small / fast model

Summarize indicators
        ↓
Small / medium model

Analyze technical + fundamental evidence
        ↓
More capable model

Create detailed research thesis
        ↓
More capable model
```

Future model comparisons should use repeatable benchmarks measuring:

-   response quality
-   accuracy
-   hallucination rate
-   latency
-   RAM/VRAM usage
-   consistency

------------------------------------------------------------------------

# Day 2 Bonus Exercise --- Quantitative Stock Data

A real quantitative scanner record for CEG was provided to `gemma4:e2b`.

The data contained fields such as:

``` text
Symbol
Live Price
Setup Type
Score
Grade
Checks Met
Entry
Stop
Target 1
Target 2
Risk Per Share
R:R T1
R:R T2
ML_Pred_Return
ML_Rank
Dist_EMA20_ATR
Dist_EMA50_ATR
Rel_Volume_5_20
BB_Width_ATR
RS_vs_SPY_21d
RS_vs_SPY_63d
Expected Value
ATR
RSI
Fib 78.6%
```

Gemma produced a polished and mostly coherent technical-analysis report.

However, the experiment revealed an extremely important LLM engineering
problem.

## Hallucination / Schema Interpretation Problem

Gemma made assumptions about fields whose definitions were not supplied.

Examples:

### `ML_Pred_Return`

Value:

``` text
0.60
```

Gemma suggested this might represent a predicted return percentage.

But the prompt did not define whether `0.60` means:

-   0.60%
-   60%
-   probability
-   normalized score
-   another internal model output

Therefore the interpretation was unsupported.

### `Expected Value`

Value:

``` text
231.00
```

Gemma interpreted this as:

``` text
$231.00
```

No currency/unit definition had been provided.

### `Rel_Volume_5_20`

Gemma described this as a type of volume correlation.

That interpretation was also not grounded in a supplied field
definition.

## Critical Lesson

> **An LLM can understand financial language while still
> misunderstanding an application's data contract.**

A polished response is not proof of a correct response.

The future Stock Research Assistant should therefore avoid feeding
unexplained raw numbers directly to an LLM.

Instead:

``` text
Quant / ML Engine
        ↓
Calculates deterministic facts
        ↓
Structured Fact Packet
        ↓
Ollama
        ↓
Interprets supplied facts
        ↓
Research Narrative
```

Avoid:

``` text
Raw unexplained metrics
        ↓
LLM guesses definitions
        ↓
Confident-looking analysis
```

A future structured packet could resemble:

``` text
ML_Pred_Return:
  value: 0.60
  definition: <exact model definition>

Expected_Value:
  value: 231
  definition: <exact calculation and units>

Rel_Volume_5_20:
  value: 0.849526
  definition: <exact calculation>

Instructions:
  - Do not infer missing definitions.
  - Do not invent units.
  - Explicitly identify unknown fields.
```

This CEG example should be preserved as a future benchmark. After
learning structured prompting, Modelfiles, APIs, and RAG, rerun the same
record and compare the quality and grounding of the response.

------------------------------------------------------------------------

# GitHub Repository

Repository:

``` text
https://github.com/jigar3730/ollama-learning
```

Local path:

``` text
C:\Apps\Projects\ollama-learning
```

Default branch:

``` text
main
```

Initial commit:

``` text
docs: initialize Ollama learning journey
```

The local repository is connected to:

``` text
origin/main
```

------------------------------------------------------------------------

# Repository Structure

Current/planned structure:

``` text
ollama-learning/
│
├── README.md
├── .gitignore
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
├── logs/
│   └── raw/
│
└── stock-research-assistant/
```

Empty directories use `.gitkeep` so Git can track them.

------------------------------------------------------------------------

# PowerShell Session Logging

PowerShell's built-in transcript functionality will be used to capture
future terminal sessions.

Start:

``` powershell
Start-Transcript -Path ".\logs\raw\day-03-powershell.txt"
```

Stop:

``` powershell
Stop-Transcript
```

Raw logs are intentionally excluded from Git.

`.gitignore` contains:

``` gitignore
logs/raw/
```

This provides the workflow:

``` text
PowerShell / Ollama
        ↓
Raw local transcript
        ↓
Review / sanitize
        ↓
Clean lesson notes
        ↓
GitHub
```

Raw logs may contain usernames, paths, API keys, tokens, prompts, or
sensitive financial data and should be reviewed before publishing.

A sanitized Day 2 transcript was placed at:

``` text
logs/day-02-powershell.txt
```

------------------------------------------------------------------------

# Concepts Learned Through Day 2

By the end of Day 2:

-   Ollama runtime vs LLM model
-   Local model lifecycle
-   `ollama list`
-   `ollama run`
-   `ollama ps`
-   `ollama show`
-   CPU vs GPU execution
-   RAM vs VRAM
-   model parameter counts
-   quantization
-   Q4_K_M
-   context length
-   model capabilities
-   model tags
-   `latest` vs explicit tags
-   disk size vs runtime memory
-   basic model-selection strategy
-   finance-focused prompting
-   hallucination risk
-   importance of data contracts
-   structured fact packets
-   Git/GitHub project tracking
-   PowerShell transcript logging

------------------------------------------------------------------------

# Starting Point for Day 3

**Next lesson:** Tokens and context windows.

Finance use case:

Understand what happens when supplying increasingly large inputs such
as:

-   technical indicator packets
-   historical observations
-   fundamental metrics
-   earnings reports
-   financial statements
-   research documents

Key future question:

> **How much information can we safely give a local model, and how do
> context size, memory usage, speed, and response quality interact?**

Continue from here in the Day 3 conversation.

------------------------------------------------------------------------

## Capstone Direction

The long-term project remains:

# Local AI Stock Research Assistant

Target architecture:

``` text
Market / Fundamental Data
          ↓
Quantitative Analysis
          ↓
Structured Fact Packet
          ↓
Python Application
          ↓
Ollama REST API
          ↓
Local LLM
          ↓
Grounded Research Brief
```

Later stages will add:

-   custom Modelfiles
-   REST APIs
-   Python integration
-   Docker
-   structured outputs
-   embeddings
-   RAG
-   model benchmarking
-   performance optimization
-   production engineering
