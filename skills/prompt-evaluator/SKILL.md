---
name: prompt-evaluator
description: >
  Evaluate and optimize LLM prompts for quality, clarity, and effectiveness.
  Use this skill when reviewing system prompts, user prompt templates, or
  any instructional text destined for an LLM. Produces a scored rubric
  with actionable rewrite suggestions. Supports single prompts, A/B
  comparison, and batch evaluation workflows.
license: MIT
metadata:
  author: ai-engineering-toolkit
  version: "1.0.0"
  category: ai-engineering
---

# Prompt Evaluator

Systematically evaluate LLM prompts across multiple quality dimensions,
produce quantitative scores, and generate optimized rewrites.

## When to Use

- Reviewing or auditing system prompts before production deployment
- Comparing two or more prompt variants to choose the best one
- Optimizing an existing prompt that produces inconsistent results
- Establishing prompt quality standards for a team or organization
- Debugging why an LLM is not following instructions correctly

## Evaluation Rubric

Score every prompt on the following **8 dimensions** (each 1-10):

| # | Dimension | What It Measures |
|---|-----------|-----------------|
| 1 | **Clarity** | Is the instruction unambiguous? Could a human follow it without asking questions? |
| 2 | **Specificity** | Does it define expected output format, length, style, and constraints? |
| 3 | **Completeness** | Are all necessary inputs, edge cases, and fallback behaviors covered? |
| 4 | **Conciseness** | Is every sentence load-bearing? Is there redundancy or filler? |
| 5 | **Structure** | Does it use clear sections, numbered steps, or formatting that aids parsing? |
| 6 | **Grounding** | Does it provide examples, few-shot demonstrations, or reference material? |
| 7 | **Safety** | Does it include guardrails against misuse, hallucination, or harmful output? |
| 8 | **Robustness** | Will it perform consistently across varied inputs and edge cases? |

### Scoring Guidelines

- **9-10**: Production-ready, best-in-class
- **7-8**: Good, minor improvements possible
- **5-6**: Functional but with notable weaknesses
- **3-4**: Significant issues that will cause failures
- **1-2**: Fundamentally broken, needs complete rewrite

## Workflow

### Mode 1: Single Prompt Evaluation

1. **Receive** the prompt text from the user.
2. **Parse** the prompt to identify: role definition, task description, constraints,
   output format, examples, and safety guardrails.
3. **Score** each of the 8 dimensions independently using the rubric above.
4. **Calculate** the overall weighted score:
   - Clarity × 1.5 + Specificity × 1.3 + Completeness × 1.2 + Conciseness × 1.0
     + Structure × 1.0 + Grounding × 1.1 + Safety × 1.3 + Robustness × 1.2
   - Normalize to a 0-100 scale.
5. **Identify** the top 3 weakest dimensions.
6. **Generate** specific, actionable improvement suggestions for each weak area.
7. **Produce** an optimized rewrite incorporating all suggestions.
8. **Output** a structured evaluation report.

### Mode 2: A/B Comparison

1. **Receive** two prompt variants (Prompt A and Prompt B).
2. **Evaluate** each independently using the full rubric.
3. **Produce** a side-by-side comparison table.
4. **Declare** a winner with detailed justification.
5. **Suggest** a merged "Prompt C" that combines the best elements of both.

### Mode 3: Batch Evaluation

1. **Receive** a directory path or list of prompt files.
2. **Evaluate** each prompt using the rubric.
3. **Produce** a summary table ranked by overall score.
4. **Identify** systemic patterns (e.g., "all prompts lack safety guardrails").
5. **Generate** a team-level recommendations report.

## Output Format

Always produce the evaluation in this structure:

```markdown
## Prompt Evaluation Report

### Metadata
- **Evaluated at**: [timestamp]
- **Mode**: [Single / A/B / Batch]
- **Prompt length**: [token count estimate]

### Scores

| Dimension | Score | Assessment |
|-----------|-------|-----------|
| Clarity | X/10 | [one-line assessment] |
| Specificity | X/10 | [one-line assessment] |
| ... | ... | ... |

### Overall Score: XX/100

### Top Issues
1. [Issue description + specific location in prompt]
2. [Issue description + specific location in prompt]
3. [Issue description + specific location in prompt]

### Recommendations
[Numbered list of specific, actionable changes]

### Optimized Rewrite
[Full rewritten prompt with changes highlighted]
```

## Anti-Patterns to Flag

Always check for and flag these common prompt anti-patterns:

- **Vague role assignment**: "You are a helpful assistant" (too generic)
- **Missing output format**: No specification of expected response structure
- **Contradictory instructions**: Conflicting directives in different sections
- **Token waste**: Excessive preamble, repeated instructions, or filler text
- **Missing edge cases**: No guidance for when the model doesn't know the answer
- **Over-constraining**: So many rules that the model struggles to satisfy all of them
- **Under-constraining**: So few rules that the model produces inconsistent outputs
- **Prompt injection vulnerability**: No defense against adversarial user inputs
- **Hallucination risk**: Encouraging the model to speculate without grounding

## Examples

### Example 1: Evaluating a Customer Support Prompt

**Input prompt**:
```
You are a customer support agent. Help users with their questions.
Be nice and helpful.
```

**Evaluation result**:
- Clarity: 4/10 — "Help users" is too vague; no scope defined
- Specificity: 2/10 — No output format, tone guidelines, or escalation rules
- Completeness: 2/10 — Missing: product knowledge, refund policy, hours, fallbacks
- Overall: 28/100

**Optimized rewrite**:
```
You are a customer support agent for [Company Name].

## Scope
You handle inquiries about: orders, shipping, returns, and product information.
You do NOT handle: billing disputes, account security, or technical bugs.

## Response Guidelines
- Tone: Professional, empathetic, concise
- Length: 2-4 sentences per response unless the issue requires detailed steps
- Always greet the user by name if available
- If you cannot resolve the issue, escalate with: "I'll connect you with a specialist."

## Output Format
1. Acknowledge the user's concern
2. Provide the solution or next step
3. Ask if there's anything else you can help with
```

### Example 2: A/B Comparison

**Prompt A**: A 500-token system prompt for code review
**Prompt B**: A 200-token system prompt for code review

Result: Prompt B scores higher on Conciseness (9 vs 4) but lower on
Completeness (5 vs 8). Recommendation: Merge into a 300-token Prompt C
that retains A's completeness with B's concise style.
