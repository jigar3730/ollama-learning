# Ollama Learning Journey --- Day 5

## PowerShell, REST API, Structured Market Data, and Grounded LLM Analysis

**Date:** 2026-09-03\
**Focus:** Calling Ollama through its REST API with PowerShell and
preparing structured financial-market data for a local LLM.

------------------------------------------------------------------------

## 1. Day 5 Objective

The original goal was to understand how applications communicate with
Ollama through its local REST API.

By the end of the session, the workflow expanded into a realistic
market-research pipeline:

``` text
Raw scanner data
      ↓
PowerShell
  parse + calculate + classify
      ↓
Structured fact packet
      ↓
Ollama REST API
      ↓
Local LLM
      ↓
Grounded research narrative
```

A key engineering principle emerged:

> **Code computes deterministic facts; the LLM explains those facts.**

------------------------------------------------------------------------

## 2. Verify the Ollama REST API

Ollama was confirmed running locally at:

``` text
http://localhost:11434
```

Installed models were retrieved with:

``` powershell
curl.exe http://localhost:11434/api/tags
```

Models included:

-   `market-researcher:latest`
-   `devops-assistant:latest`
-   `gemma4:latest`
-   `gemma4:e2b`
-   `llama3.2:latest`

The finance exercises used:

``` text
market-researcher
```

------------------------------------------------------------------------

## 3. Calling `/api/generate`

A direct `curl.exe` POST initially failed because of PowerShell/JSON
quoting:

``` text
invalid character 'm' looking for beginning of object key string
```

PowerShell's native `Invoke-RestMethod` proved cleaner.

### Build the request body

``` powershell
$body = @{
    model  = "market-researcher"
    prompt = "RSI is 72. Price is above the 50-day moving average. Provide a one-sentence technical assessment."
    stream = $false
} | ConvertTo-Json
```

### Send the POST request

``` powershell
$result = Invoke-RestMethod `
    -Uri "http://localhost:11434/api/generate" `
    -Method Post `
    -ContentType "application/json" `
    -Body $body
```

### Extract only the LLM answer

``` powershell
$result.response
```

Conceptually:

``` text
PowerShell object
      ↓
ConvertTo-Json
      ↓
JSON request
      ↓
HTTP POST /api/generate
      ↓
Ollama
      ↓
JSON response
      ↓
PowerShell object
      ↓
$result.response
```

------------------------------------------------------------------------

## 4. PowerShell Fundamentals Practiced

### Variables

``` powershell
$name = "Ollama"
$model = "market-researcher"
```

Variables are referenced using `$`.

### Hashtables

``` powershell
$request = @{
    model  = "market-researcher"
    prompt = "Analyze RSI 72"
    stream = $false
}
```

Access individual values:

``` powershell
$request.model
$request.prompt
```

### Pipeline

``` powershell
$request | ConvertTo-Json
```

The `|` passes an object to the next command.

### Cmdlet naming

PowerShell commonly follows:

``` text
Verb-Noun
```

Examples:

``` text
ConvertTo-Json
ConvertFrom-Csv
Invoke-RestMethod
```

A typo such as `ConverTo-Json` produces a `CommandNotFoundException`.

------------------------------------------------------------------------

## 5. Working with the Ollama Response Object

Instead of printing the API response immediately:

``` powershell
$result = Invoke-RestMethod ...
```

The returned JSON becomes a PowerShell object.

Useful properties:

``` powershell
$result.model
$result.response
$result.done
$result.eval_count
```

This demonstrates an important distinction:

``` text
$body   = outgoing JSON request
$result = parsed incoming API response
```

------------------------------------------------------------------------

## 6. Importing Real Scanner Data

The session used APH and MNDY Coiled Cobra scanner rows.

CSV text can be stored using a PowerShell here-string:

``` powershell
$csv = @"
Symbol,Setup Type,...
APH,SETUP_LONG,...
MNDY,SETUP_LONG,...
"@
```

Convert the CSV into PowerShell objects:

``` powershell
$stocks = $csv | ConvertFrom-Csv
```

Select the first row:

``` powershell
$stock = $stocks[0]
```

Access normal properties:

``` powershell
$stock.Symbol
$stock.RSI
$stock.Score
```

Properties containing spaces can be accessed with quotes:

``` powershell
$stock.'Stock Entry'
$stock.'Target 1'
$stock.'Checks Met'
```

------------------------------------------------------------------------

## 7. String Interpolation

A PowerShell here-string can dynamically insert object properties:

``` powershell
$prompt = @"
Symbol: $($stock.Symbol)
Entry: $($stock.'Stock Entry')
RSI: $($stock.RSI)
"@
```

The syntax:

``` powershell
$($stock.Symbol)
```

evaluates the expression and inserts its value into the string.

------------------------------------------------------------------------

## 8. Why Raw Numbers Should Not Be Left Entirely to the LLM

The first Ollama analysis made several interpretation errors, including:

-   treating RSI 54.01 as overbought,
-   claiming APH was below EMA20 when the close was actually above
    EMA20,
-   interpreting relative volume 0.89 as buying pressure,
-   confusing Target 1 with a historical high,
-   inventing thresholds that were never supplied.

Tighter prompting improved the answer but did not completely eliminate
unsupported interpretations.

This led to the key design decision:

``` text
Avoid:

Raw numbers
    ↓
LLM calculates + classifies + explains

Prefer:

Raw numbers
    ↓
PowerShell/Python calculates
    ↓
PowerShell/Python classifies
    ↓
LLM explains
```

------------------------------------------------------------------------

## 9. Numeric Type Conversion

CSV values should be explicitly converted before arithmetic:

``` powershell
$entry = [double]$stock.'Stock Entry'
$stop = [double]$stock.'Stock Stop'
$target1 = [double]$stock.'Target 1'
```

The `[double]` cast converts the CSV string into a numeric
floating-point value.

------------------------------------------------------------------------

## 10. Risk/Reward Calculations

For APH:

``` text
Entry    = 81.33
Stop     = 77.23
Target 1 = 89.53
```

PowerShell calculations:

``` powershell
$risk = $entry - $stop
$reward = $target1 - $entry
$rewardRisk = $reward / $risk
```

Results:

``` text
Risk/share       = 4.10
Reward/share     = 8.20
Reward/Risk      = 2.00
```

Floating-point arithmetic initially produced:

``` text
4.09999999999999
```

For human-facing output, round values:

``` powershell
$risk = [math]::Round($entry - $stop, 2)
$reward = [math]::Round($target1 - $entry, 2)
$rewardRisk = [math]::Round($reward / $risk, 2)
```

------------------------------------------------------------------------

## 11. PowerShell Conditional Logic

### Comparison operators

``` text
-gt   greater than
-lt   less than
-eq   equal
-ge   greater than or equal
-le   less than or equal
-ne   not equal
```

### RSI classification

``` powershell
if ($rsi -ge 70) {
    $rsiState = "Overbought"
} elseif ($rsi -le 30) {
    $rsiState = "Oversold"
} else {
    $rsiState = "Neutral"
}
```

APH result:

``` text
Neutral
```

Important interactive-shell lesson: `if`, `elseif`, and `else` form one
compound statement and should be entered/pasted as one block.

### Volume classification

``` powershell
if ($relVolume -gt 1) {
    $volumeState = "Above average"
} elseif ($relVolume -lt 1) {
    $volumeState = "Below average"
} else {
    $volumeState = "Average"
}
```

APH result:

``` text
Below average
```

------------------------------------------------------------------------

## 12. Logical Operators

Price relationships were calculated first:

``` powershell
$aboveEMA20 = $close -gt $ema20
$aboveEMA50 = $close -gt $ema50
```

APH:

``` text
Above EMA20 = True
Above EMA50 = True
```

PowerShell logical operators:

``` text
-and   both conditions must be true
-or    at least one condition must be true
```

Trend classification:

``` powershell
if ($aboveEMA20 -and $aboveEMA50) {
    $trendState = "Price above both EMA20 and EMA50"
} elseif ($aboveEMA20 -or $aboveEMA50) {
    $trendState = "Price above one moving average"
} else {
    $trendState = "Price below both EMA20 and EMA50"
}
```

APH result:

``` text
Price above both EMA20 and EMA50
```

------------------------------------------------------------------------

## 13. Building a Structured Fact Packet

Instead of giving Ollama raw scanner data, a nested PowerShell object
was created:

``` powershell
$factPacket = @{
    Symbol = $stock.Symbol
    Setup = $stock.'Setup Type'

    Price = @{
        Close = [double]$stock.Close
        Entry = $entry
        Stop = $stop
        Target1 = $target1
    }

    Derived = @{
        AboveEMA20 = $aboveEMA20
        AboveEMA50 = $aboveEMA50
        RSIState = $rsiState
        VolumeState = $volumeState
        RiskPerShare = $risk
        RewardToTarget1 = $reward
        RewardRisk = $rewardRisk
    }

    Scanner = @{
        Score = [double]$stock.Score
        Grade = $stock.Grade
        ChecksMet = $stock.'Checks Met'
    }
}
```

Trend was later added without rebuilding the object:

``` powershell
$factPacket.Derived.TrendState = $trendState
```

This demonstrated modifying a nested object property.

------------------------------------------------------------------------

## 14. Nested JSON

The fact packet was serialized with:

``` powershell
$factJson = $factPacket | ConvertTo-Json -Depth 5
```

`-Depth 5` allows nested objects such as `Price`, `Derived`, and
`Scanner` to be serialized correctly.

Example structure:

``` json
{
  "Price": {
    "Close": 82.07,
    "Stop": 77.23,
    "Target1": 89.53,
    "Entry": 81.33
  },
  "Scanner": {
    "Score": 71,
    "Grade": "B - Valid Coil",
    "ChecksMet": "5/8"
  },
  "Setup": "SETUP_LONG",
  "Symbol": "APH",
  "Derived": {
    "RewardRisk": 2,
    "RSIState": "Neutral",
    "VolumeState": "Below average",
    "AboveEMA50": true,
    "RiskPerShare": 4.1,
    "RewardToTarget1": 8.2,
    "AboveEMA20": true,
    "TrendState": "Price above both EMA20 and EMA50"
  }
}
```

------------------------------------------------------------------------

## 15. Grounded Ollama Prompt

The final prompt told the model to treat derived fields as
authoritative:

``` powershell
$prompt = @"
You are a stock research assistant.

Use ONLY the structured facts below.
Do not invent thresholds, calculations, or interpretations.
Treat all fields under Derived as authoritative classifications.
If a fact is unavailable, say unavailable.

FACT PACKET:
$factJson

Write a concise research summary covering:
1. Trend
2. Momentum
3. Volume
4. Risk/reward
5. Overall setup

Do not provide investment advice.
"@
```

Then:

``` powershell
$request.prompt = $prompt
$body = $request | ConvertTo-Json

$result = Invoke-RestMethod `
    -Uri "http://localhost:11434/api/generate" `
    -Method Post `
    -ContentType "application/json" `
    -Body $body

$result.response
```

The final Ollama output stayed substantially closer to the supplied
facts:

``` text
Trend: Price above both EMA20 and EMA50
Momentum: RSI neutral
Volume: Below average
Risk/reward: 2:1
Setup: SETUP_LONG
```

------------------------------------------------------------------------

## 16. Day 5 Key Engineering Lesson

The most important takeaway is not merely how to call Ollama.

It is how responsibilities should be divided.

### Deterministic layer --- PowerShell/Python

Use normal code for:

-   parsing data,
-   arithmetic,
-   indicator comparisons,
-   thresholds,
-   classifications,
-   validation,
-   risk/reward calculations,
-   structured fact creation.

### Generative layer --- Ollama/LLM

Use the LLM for:

-   summarization,
-   explanation,
-   narrative generation,
-   organizing observations,
-   turning structured facts into readable research briefs.

### Target architecture

``` text
Market / Scanner Data
        ↓
Technical Indicators
        ↓
PowerShell / Python
        ↓
Deterministic calculations
        ↓
Deterministic classifications
        ↓
Structured JSON Fact Packet
        ↓
Ollama REST API
        ↓
Local LLM
        ↓
Structured Research Brief
```

This architecture reduces hallucination risk and forms the foundation of
the planned **Local AI Stock Research Assistant**.

------------------------------------------------------------------------

## 17. PowerShell Commands Learned

``` powershell
# Variables
$name = "Ollama"

# Hashtable
$request = @{
    model = "market-researcher"
}

# Pipeline
$request | ConvertTo-Json

# CSV parsing
$stocks = $csv | ConvertFrom-Csv

# Property access
$stock.Symbol
$stock.'Stock Entry'

# Numeric conversion
[double]$stock.RSI

# Arithmetic
$risk = $entry - $stop

# Rounding
[math]::Round($risk, 2)

# Comparison
$rsi -ge 70

# Logical operators
$aboveEMA20 -and $aboveEMA50

# Conditional logic
if (...) {
} elseif (...) {
} else {
}

# Nested property
$factPacket.Derived.TrendState

# JSON serialization
$factPacket | ConvertTo-Json -Depth 5

# REST POST
Invoke-RestMethod `
    -Uri "http://localhost:11434/api/generate" `
    -Method Post `
    -ContentType "application/json" `
    -Body $body

# Ollama response
$result.response
```

------------------------------------------------------------------------

## 18. Day 6 Preview

Day 6 will move the same core workflow from interactive PowerShell into
a small Python program:

``` text
Python
   ↓
Ollama REST API
   ↓
market-researcher
   ↓
Python parses response
```

The goal is not to abandon what was learned in PowerShell. Python will
automate the same concepts:

-   objects/data structures,
-   JSON,
-   HTTP requests,
-   deterministic calculations,
-   structured fact packets,
-   Ollama responses.

------------------------------------------------------------------------

## Status

**Day 5: COMPLETE**

Skills demonstrated:

-   Ollama REST API connectivity
-   PowerShell objects and variables
-   CSV parsing
-   JSON serialization
-   REST POST requests
-   API response handling
-   numeric calculations
-   conditional logic
-   nested objects
-   structured fact packets
-   grounded LLM prompting
-   separation of deterministic computation from generative analysis

> Examples are for learning and research purposes and are not investment
> advice.
