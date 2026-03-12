---
name: context-budget-planner
description: >
  Plan and optimize token budget allocation for LLM context windows.
  Use this skill when designing system prompts, RAG pipelines, or
  multi-turn agents that must fit within a model's context limit.
  Produces a token budget breakdown, identifies waste, and suggests
  compression strategies to maximize the useful information density
  within the available context window.
license: MIT
metadata:
  author: ai-engineering-toolkit
  version: "1.0.0"
  category: ai-engineering
---

# Context Budget Planner

Design optimal token allocation strategies for LLM context windows.
Analyze current usage, identify waste, and produce a balanced budget
that maximizes output quality within the model's limits.

## When to Use

- Designing a new system prompt and need to plan token allocation
- Your agent is hitting context window limits and needs optimization
- Migrating between models with different context sizes (e.g., 8K → 128K → 200K)
- Analyzing cost vs. quality tradeoffs for production deployments
- Building multi-turn agents that must maintain coherent long conversations
- Designing RAG systems where retrieval results compete for context space

## Context Window Reference

| Model | Context Window | Effective Limit* | Cost Tier |
|-------|---------------|-------------------|-----------|
| Claude 3.5 Sonnet | 200K tokens | ~160K tokens | $$ |
| Claude 3.5 Haiku | 200K tokens | ~160K tokens | $ |
| Claude 3 Opus | 200K tokens | ~150K tokens | $$$ |
| GPT-4o | 128K tokens | ~100K tokens | $$ |
| GPT-4o-mini | 128K tokens | ~100K tokens | $ |
| Gemini 1.5 Pro | 2M tokens | ~1.5M tokens | $$ |
| Gemini 2.0 Flash | 1M tokens | ~800K tokens | $ |
| DeepSeek V3 | 128K tokens | ~100K tokens | $ |
| Qwen 2.5 | 128K tokens | ~100K tokens | $ |

*\*Effective limit accounts for output tokens and performance degradation
in the "lost in the middle" zone.*

## Budget Allocation Framework

### The 5-Zone Model

Divide the context window into five functional zones:

```
┌─────────────────────────────────────────────────┐
│  Zone 1: System Instructions        (10-20%)    │
│  ├── Role definition                            │
│  ├── Behavioral rules & constraints             │
│  ├── Output format specification                │
│  └── Safety guardrails                          │
├─────────────────────────────────────────────────┤
│  Zone 2: Persistent Knowledge        (5-15%)    │
│  ├── Domain knowledge / few-shot examples       │
│  ├── Long-term memory summaries                 │
│  └── User/session preferences                   │
├─────────────────────────────────────────────────┤
│  Zone 3: Retrieved Context (RAG)    (20-40%)    │
│  ├── Document chunks                            │
│  ├── Search results                             │
│  ├── API responses                              │
│  └── Tool outputs                               │
├─────────────────────────────────────────────────┤
│  Zone 4: Conversation History       (15-30%)    │
│  ├── Recent turns (full detail)                 │
│  ├── Older turns (summarized)                   │
│  └── Key decisions / anchors                    │
├─────────────────────────────────────────────────┤
│  Zone 5: Output Reserve             (10-20%)    │
│  └── Reserved for model's response generation   │
└─────────────────────────────────────────────────┘
```

### Allocation Profiles

Different use cases demand different zone distributions:

| Profile | Zone 1 | Zone 2 | Zone 3 | Zone 4 | Zone 5 |
|---------|--------|--------|--------|--------|--------|
| **Chatbot** | 15% | 5% | 10% | 55% | 15% |
| **RAG Q&A** | 10% | 5% | 50% | 15% | 20% |
| **Code Agent** | 20% | 10% | 35% | 20% | 15% |
| **Document Analysis** | 10% | 5% | 60% | 10% | 15% |
| **Multi-turn Planner** | 20% | 15% | 15% | 35% | 15% |
| **Creative Writer** | 15% | 10% | 10% | 30% | 35% |

## Workflow

### Step 1: Inventory Current Usage

Analyze the current context composition:

1. **Measure** each component's token count (use `tiktoken` for OpenAI,
   estimate 1 token ≈ 4 characters for Claude).
2. **Classify** each component into one of the 5 zones.
3. **Calculate** the percentage allocation.
4. **Produce** a usage breakdown table.

### Step 2: Identify Waste & Inefficiency

Check for these common issues:

| Issue | Detection Method | Typical Savings |
|-------|-----------------|-----------------|
| **Redundant instructions** | Duplicate or near-duplicate sentences | 10-30% of Zone 1 |
| **Verbose examples** | Examples longer than necessary to convey the pattern | 20-50% of Zone 2 |
| **Unranked RAG chunks** | Low-relevance chunks included without filtering | 30-60% of Zone 3 |
| **Full conversation replay** | Entire history without summarization | 40-70% of Zone 4 |
| **Unused tool definitions** | Tools defined but never called | 5-15% of Zone 1 |
| **Over-reserved output** | Max tokens set much higher than actual output length | 5-20% of Zone 5 |

### Step 3: Apply Compression Strategies

For each zone, apply the appropriate compression technique:

**Zone 1 (System Instructions)**:
- Merge overlapping rules
- Convert prose to structured lists
- Use shorthand for repeated patterns
- Move rarely-triggered rules to conditional loading

**Zone 2 (Persistent Knowledge)**:
- Replace verbose examples with minimal, high-signal demonstrations
- Use structured formats (tables, YAML) instead of prose
- Implement tiered loading: core knowledge always, extended on-demand

**Zone 3 (RAG Context)**:
- Increase retrieval precision (fewer, better chunks)
- Apply extractive summarization to long chunks
- Implement relevance scoring with a minimum threshold
- Use metadata-enriched chunks instead of raw text

**Zone 4 (Conversation History)**:
- Implement sliding window: keep last N turns in full
- Summarize older turns into bullet-point recaps
- Extract and persist key decisions as "anchor points"
- Drop redundant turns (e.g., greetings, confirmations)

**Zone 5 (Output Reserve)**:
- Analyze actual output lengths to right-size the reserve
- Use streaming to avoid over-allocation

### Step 4: Produce the Optimized Budget

Generate a before/after comparison:

```markdown
## Token Budget Report

### Model: [model name] | Context Window: [size]

### Before Optimization
| Zone | Component | Tokens | % | Status |
|------|-----------|--------|---|--------|
| 1 | System prompt | 3,200 | 25% | ⚠️ Over |
| 2 | Examples | 1,500 | 12% | ⚠️ Over |
| 3 | RAG chunks (8) | 4,000 | 31% | ✅ OK |
| 4 | History (20 turns) | 3,500 | 27% | ⚠️ Over |
| 5 | Output reserve | 800 | 6% | ⚠️ Under |
| **Total** | | **13,000** | **100%** | |

### After Optimization
| Zone | Component | Tokens | % | Savings |
|------|-----------|--------|---|---------|
| 1 | System prompt | 1,800 | 14% | -44% |
| 2 | Examples | 600 | 5% | -60% |
| 3 | RAG chunks (5) | 3,800 | 29% | -5% |
| 4 | History (summary) | 2,000 | 15% | -43% |
| 5 | Output reserve | 2,000 | 15% | +150% |
| **Free budget** | | **2,800** | **22%** | NEW |
| **Total** | | **13,000** | **100%** | |

### Cost Impact
- Estimated cost reduction: ~35% per request
- Quality impact: Neutral to positive (less noise, more output room)
```

### Step 5: Cost Projection

Calculate the financial impact:

```
Daily requests × avg input tokens × input price / 1M = Daily input cost
Daily requests × avg output tokens × output price / 1M = Daily output cost
Total daily cost = input + output
Monthly cost = daily × 30
Savings = (before - after) / before × 100%
```

## Guidelines

- **Never sacrifice Zone 5**: Insufficient output reserve causes truncated responses
  and degrades user experience. Minimum 10%.
- **Zone 3 quality > quantity**: 3 highly relevant RAG chunks outperform 10 mediocre ones.
- **Zone 4 is the biggest optimization opportunity**: Conversation history is typically
  the most compressible zone. Prioritize summarization here.
- **Monitor the "lost in the middle" effect**: Information placed in the middle of
  long contexts is less likely to be used. Put critical information at the
  beginning and end of each zone.
- **Budget for growth**: Leave 10-15% free headroom for unexpected inputs.

## Examples

### Example: Optimizing a Customer Support Agent

**Before**: 128K context, using 95% capacity, frequent truncation
- System: 25K (20%) — includes full product catalog
- RAG: 50K (39%) — retrieves 20 chunks per query
- History: 45K (35%) — full conversation, no summarization
- Output: 8K (6%) — frequently truncated

**After optimization**:
- System: 12K (9%) — product catalog moved to RAG, conditional loading
- RAG: 35K (27%) — top-5 chunks only, extractive summaries
- History: 20K (16%) — sliding window (5 turns full + summary)
- Output: 20K (16%) — generous reserve for detailed responses
- Free: 41K (32%) — headroom for peak loads

**Result**: 3x more output room, zero truncation, 30% cost reduction.
