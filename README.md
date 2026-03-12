# AI Engineering Toolkit

> A collection of production-ready Agent Skills for AI engineering — covering
> prompt evaluation, context budget planning, RAG pipeline design, agent
> safety, and systematic evaluation frameworks.

[![Agent Skills Standard](https://img.shields.io/badge/Agent_Skills-v1.0-blue)](https://agentskills.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/Skills-5-orange)]()

## Why This Toolkit?

Most AI engineering resources teach you **what** context engineering or RAG
is. This toolkit gives you **executable workflows** — structured, step-by-step
skills that an AI agent can load and follow to actually build, evaluate, and
secure your AI systems.

| Existing Resources | This Toolkit |
|-------------------|-------------|
| Theory-oriented | Execution-oriented |
| "Here's what RAG is" | "Here's how to build, evaluate & optimize a RAG pipeline" |
| Conceptual frameworks | Scoring rubrics, checklists, decision trees |
| English-only | Designed for global adoption |

## Skills Included

| # | Skill | Description |
|---|-------|------------|
| 1 | **[prompt-evaluator](skills/prompt-evaluator/)** | Evaluate LLM prompts across 8 quality dimensions with quantitative scoring and automated rewrite suggestions |
| 2 | **[context-budget-planner](skills/context-budget-planner/)** | Plan token allocation across 5 context zones, identify waste, and optimize cost-quality tradeoffs |
| 3 | **[rag-pipeline-architect](skills/rag-pipeline-architect/)** | Design end-to-end RAG systems from document ingestion to retrieval evaluation, covering 3 architecture tiers |
| 4 | **[agent-safety-guard](skills/agent-safety-guard/)** | Implement defense-in-depth security for AI agents with 5-layer protection and red-team testing checklists |
| 5 | **[eval-harness-builder](skills/eval-harness-builder/)** | Build systematic evaluation frameworks with automated metrics, LLM-as-a-Judge, and CI/CD integration |

## Quick Start

### Claude Code / WorkBuddy

```bash
# Add as a plugin marketplace source
/plugin marketplace add viliawang-pm/ai-engineering-toolkit

# Or install individual skills
/plugin install prompt-evaluator@ai-engineering-toolkit
/plugin install rag-pipeline-architect@ai-engineering-toolkit
```

### Manual Installation

```bash
# Clone the repository
git clone https://github.com/viliawang-pm/ai-engineering-toolkit.git

# Copy skills to your project
cp -r ai-engineering-toolkit/skills/prompt-evaluator .claude/skills/
```

### Claude.ai / API

Upload individual `SKILL.md` files as project knowledge or include them
in your system prompt.

## Usage Examples

### Evaluate a Prompt

> "Use the prompt-evaluator skill to evaluate this system prompt for my
> customer support chatbot: [paste prompt]"

The agent will produce a scored rubric (8 dimensions, 0-100 overall)
with specific improvement suggestions and an optimized rewrite.

### Plan Context Budget

> "Use the context-budget-planner skill to optimize my RAG agent's context
> window. I'm using Claude 3.5 Sonnet with a 200K context window."

The agent will analyze your current token allocation, identify waste,
and produce a before/after budget with cost projections.

### Design a RAG Pipeline

> "Use the rag-pipeline-architect skill to help me build a RAG system for
> our internal documentation (500 Markdown files, technical content)."

The agent will guide you through document analysis, chunking strategy
selection, embedding model choice, retrieval design, and evaluation setup.

### Security Audit

> "Use the agent-safety-guard skill to red-team test our production chatbot
> before launch."

The agent will run through the 65+ test checklist across 5 attack
categories and produce a security assessment report.

### Build an Eval Framework

> "Use the eval-harness-builder skill to set up evaluation for our Q&A bot.
> We need to measure correctness, faithfulness, and relevance."

The agent will design the eval config, help build the dataset, implement
scoring (automated + LLM judge), and set up CI/CD integration.

## Compatibility

These skills follow the [Agent Skills Standard](https://agentskills.io)
and are compatible with:

- ✅ Claude Code / WorkBuddy
- ✅ Claude.ai (paid plans)
- ✅ Claude API
- ✅ OpenAI Codex
- ✅ Gemini CLI
- ✅ Cursor
- ✅ Windsurf
- ✅ GitHub Copilot

## Project Structure

```
ai-engineering-toolkit/
├── README.md
├── LICENSE
├── skills/
│   ├── prompt-evaluator/
│   │   └── SKILL.md
│   ├── context-budget-planner/
│   │   └── SKILL.md
│   ├── rag-pipeline-architect/
│   │   └── SKILL.md
│   ├── agent-safety-guard/
│   │   └── SKILL.md
│   └── eval-harness-builder/
│       └── SKILL.md
└── examples/
    └── (coming soon)
```

## Contributing

Contributions are welcome! To add a new skill:

1. Fork this repository
2. Create a new directory under `skills/`
3. Add a `SKILL.md` following the [Agent Skills spec](https://agentskills.io)
4. Submit a pull request with a description of the skill and example usage

## License

MIT License — see [LICENSE](LICENSE) for details.

## Acknowledgments

- [Anthropic](https://anthropic.com) for the Agent Skills standard
- [agentskills.io](https://agentskills.io) for the open specification
- The Claude Code and AI engineering community
