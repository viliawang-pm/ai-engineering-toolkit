---
name: eval-harness-builder
description: >
  Build systematic evaluation frameworks for LLM applications and AI agent
  systems. Use this skill when you need to measure LLM output quality,
  compare model performance, evaluate RAG accuracy, benchmark agent
  reliability, or set up continuous evaluation pipelines. Covers both
  automated metrics and LLM-as-a-Judge approaches with bias mitigation.
license: MIT
metadata:
  author: ai-engineering-toolkit
  version: "1.0.0"
  category: ai-engineering
---

# Eval Harness Builder

Design and implement comprehensive evaluation frameworks for LLM applications.

## When to Use

- Setting up quality metrics for a new LLM application
- Comparing model versions or prompt changes objectively
- Building automated regression testing for AI features
- Evaluating RAG retrieval quality and answer faithfulness
- Designing LLM-as-a-Judge pipelines with bias mitigation
- Setting up continuous evaluation in CI/CD for AI products

## Evaluation Taxonomy

### Level 1: Component Evaluation

Evaluate individual components in isolation.

| Component | Key Metrics | Method |
|-----------|------------|--------|
| **Embedding model** | Recall@K, MRR, NDCG | Benchmark on labeled query-doc pairs |
| **Retrieval** | Precision@K, Recall@K, Hit Rate | Compare retrieved vs. relevant docs |
| **Re-ranker** | NDCG improvement over base retrieval | A/B on re-ranked vs. base results |
| **Generation prompt** | Task completion rate, format compliance | Automated checks + LLM judge |
| **Tool calling** | Accuracy of tool selection, parameter correctness | Ground truth comparison |

### Level 2: End-to-End Evaluation

Evaluate the complete pipeline from input to output.

| Metric | What It Measures | Scoring Method |
|--------|-----------------|----------------|
| **Correctness** | Is the answer factually right? | Ground truth comparison or LLM judge |
| **Faithfulness** | Does the answer stick to provided context? | LLM judge: check each claim against sources |
| **Relevance** | Does the answer address the question? | LLM judge with rubric |
| **Completeness** | Are all aspects of the question covered? | Checklist evaluation |
| **Harmlessness** | Is the output free of harmful content? | Safety classifier + human review |
| **Latency** | Response time (P50, P95, P99) | Automated timing |
| **Cost** | Token usage and API costs per query | Automated tracking |

### Level 3: Behavioral Evaluation

Test how the system handles edge cases and adversarial inputs.

| Test Category | Examples | Pass Criteria |
|--------------|---------|---------------|
| **Refusal** | Unanswerable questions, out-of-scope requests | Correctly refuses with explanation |
| **Consistency** | Same question rephrased 5 ways | Same answer (or semantically equivalent) |
| **Robustness** | Typos, grammar errors, mixed languages | Correct answer despite noisy input |
| **Calibration** | Questions with varying confidence | Appropriate uncertainty expression |
| **Adversarial** | Prompt injection, jailbreaks | Attack blocked, no information leak |

## Workflow

### Step 1: Define Evaluation Criteria

For each application, select the relevant metrics:

```yaml
eval_config:
  name: "my-rag-app-eval"
  version: "1.0"
  metrics:
    required:
      - correctness    # weight: 0.3
      - faithfulness   # weight: 0.3
      - relevance      # weight: 0.2
    optional:
      - completeness   # weight: 0.1
      - conciseness    # weight: 0.1
  pass_threshold: 0.75  # minimum weighted score to pass
  sample_size: 100      # minimum eval dataset size
```

### Step 2: Build the Eval Dataset

**Dataset structure**:

```json
{
  "id": "eval-001",
  "query": "What is the refund policy for digital products?",
  "expected_answer": "Digital products can be refunded within 14 days...",
  "relevant_docs": ["policy-doc-v3.md#section-4"],
  "difficulty": "easy",
  "category": "policy-factual",
  "metadata": {
    "created_by": "human",
    "reviewed": true,
    "last_updated": "2026-03-01"
  }
}
```

**Dataset requirements**:
- Minimum 50 examples for prototype, 200+ for production eval
- Balance across difficulty levels: 30% easy, 40% medium, 30% hard
- Include: factual, multi-hop reasoning, unanswerable, ambiguous, adversarial
- Version control the dataset — never modify without incrementing version
- Never use eval data for training or prompt tuning (data contamination)

### Step 3: Implement Scoring

**Automated scoring** (where possible):

| Metric | Automated Method |
|--------|-----------------|
| Exact match | String comparison after normalization |
| Contains answer | Check if key phrases appear in response |
| Format compliance | Regex or schema validation |
| Latency | Wall-clock timing |
| Token count | Tokenizer count |
| Citation accuracy | Parse citations, verify against source docs |

**LLM-as-a-Judge scoring** (for subjective quality):

```markdown
## Judge Prompt Template

You are an expert evaluator. Score the following AI response.

### Evaluation Criteria: {metric_name}
{metric_description_and_rubric}

### Scoring Scale
1 - Completely fails the criterion
2 - Major deficiencies
3 - Partially meets the criterion
4 - Mostly meets with minor issues
5 - Fully meets the criterion

### Input
**Question**: {query}
**Reference Answer**: {expected_answer}
**AI Response**: {actual_response}
**Context Provided**: {context_if_applicable}

### Your Evaluation
Provide your score and a 2-3 sentence justification.

Score: [1-5]
Justification: [explanation]
```

### Step 4: Mitigate Judge Bias

LLM judges have known biases. Apply these countermeasures:

| Bias | Description | Mitigation |
|------|-------------|-----------|
| **Position bias** | Prefers first/last option in comparisons | Randomize order, average both orderings |
| **Verbosity bias** | Prefers longer responses | Include "conciseness" as an explicit criterion |
| **Self-enhancement** | Prefers outputs from the same model | Use a different model as judge |
| **Anchoring** | Over-influenced by the reference answer | Evaluate without reference, then with reference |
| **Leniency** | Tends to give high scores | Calibrate with known-bad examples (score should be 1-2) |

**Multi-judge approach**: For critical evaluations, use 3 judges (2 LLM + 1 human) and take the median.

### Step 5: Build the Pipeline

```python
# Evaluation pipeline structure (pseudocode)

class EvalHarness:
    def __init__(self, config, dataset, system_under_test):
        self.config = config
        self.dataset = dataset
        self.system = system_under_test
        self.results = []

    def run(self):
        for example in self.dataset:
            # 1. Get system response
            response = self.system.generate(example.query)

            # 2. Score with automated metrics
            auto_scores = self.auto_score(example, response)

            # 3. Score with LLM judge (if configured)
            judge_scores = self.judge_score(example, response)

            # 4. Combine scores
            combined = self.combine_scores(auto_scores, judge_scores)

            self.results.append({
                "id": example.id,
                "query": example.query,
                "response": response,
                "scores": combined,
                "pass": combined["weighted_avg"] >= self.config.pass_threshold
            })

        return self.generate_report()

    def generate_report(self):
        # Summary statistics
        # Per-metric breakdowns
        # Failure analysis
        # Regression detection (compare with previous run)
        pass
```

### Step 6: Continuous Evaluation

Integrate into CI/CD:

```yaml
# Example: GitHub Actions eval workflow
name: AI Eval Pipeline
on:
  pull_request:
    paths: ['prompts/**', 'rag/**', 'config/**']

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - name: Run eval harness
        run: python eval/run_eval.py --config eval/config.yaml
      - name: Check pass rate
        run: |
          PASS_RATE=$(cat eval/results/summary.json | jq '.pass_rate')
          if (( $(echo "$PASS_RATE < 0.75" | bc -l) )); then
            echo "❌ Eval pass rate $PASS_RATE below threshold 0.75"
            exit 1
          fi
      - name: Post results to PR
        run: python eval/post_results.py --pr ${{ github.event.number }}
```

## Output Format

Always produce evaluation results in this structure:

```markdown
## Evaluation Report

### Summary
- **System**: [name and version]
- **Eval dataset**: [name, version, size]
- **Date**: [timestamp]
- **Overall pass rate**: XX% (N/M examples passed)
- **Verdict**: ✅ PASS / ❌ FAIL

### Per-Metric Results
| Metric | Mean Score | Median | Min | P5 | Pass Rate |
|--------|-----------|--------|-----|-----|-----------|
| Correctness | X.XX | X.XX | X.XX | X.XX | XX% |
| ... | ... | ... | ... | ... | ... |

### Failure Analysis
| Failure Category | Count | % | Example ID |
|-----------------|-------|---|------------|
| Hallucination | N | XX% | eval-023 |
| ... | ... | ... | ... |

### Regression Check
- Compared with: [previous run ID]
- Metrics improved: [list]
- Metrics degraded: [list]
- New failures: [list of example IDs]

### Recommendations
1. [Specific actionable improvement]
2. [Specific actionable improvement]
```

## Guidelines

- **Eval before optimize**: Always build the eval harness before making changes.
  Otherwise you cannot measure if changes helped or hurt.
- **Separate eval from training data**: Data contamination is the #1 cause
  of misleading eval results. Strict separation is non-negotiable.
- **Human-in-the-loop for v1**: For the first version, have humans validate
  at least 20% of the LLM judge's scores to calibrate.
- **Track over time**: Individual scores matter less than trends. A 5% regression
  after a prompt change is more informative than any absolute score.
- **Test the judge**: Periodically inject known-good and known-bad responses
  to verify the judge is calibrated correctly.
