# Prompt Pipeline Analysis — Rebuilding a Web App with AI Agents

A post-mortem analysis of an experiment: **can a real web application be recreated from a single specification document, using only a pipeline of structured prompts and AI coding agents, without falling into the classic context-rot failure modes?**

## The experiment

The same application — a stock-trading simulator with social groups, virtual currency and role-based permissions — exists in two versions, both included in this repository as **git submodules**:

| Submodule | Repository | What it is |
|---|---|---|
| [`tradinGroups/`](tradinGroups) | [NECKER55/tradinGroups](https://github.com/NECKER55/tradinGroups) | The **handwritten reference implementation** (Express + Prisma + PostgreSQL, Vite + React). ~16.8k lines of TypeScript. |
| [`AppFromPrompts/`](AppFromPrompts) | [NECKER55/AppFromPrompts](https://github.com/NECKER55/AppFromPrompts) | The **AI-generated implementation**, built by GitHub Copilot agents through a 7-phase prompt pipeline starting from `specifiche.md`. ~50k lines (source + tests), 384 commits. |

The pipeline expanded the spec into 4 technical documents, decomposed them into 42 atomic features, gave each feature to an isolated TDD agent (feature file + a shared `MASTER_SYSTEM_CONTEXT.md` as its *entire* context), then ran QA gap-analysis and integration-test phases.

## The analysis

The full 14-page report — detailed pipeline description, metrics, evidence with file-level references, root-cause analysis and concrete recommendations for the next pipeline iteration — is available in two languages:

- 🇬🇧 **English**: [`ENG/pipeline_analysis_finApp.pdf`](ENG/pipeline_analysis_finApp.pdf) ([LaTeX source](ENG/pipeline_analysis_finApp.tex))
- 🇮🇹 **Italiano**: [`ITA/analisi_pipeline_finApp.pdf`](ITA/analisi_pipeline_finApp.pdf) ([LaTeX source](ITA/analisi_pipeline_finApp.tex))

## The findings (TL;DR)

**What worked**

- `tsc --noEmit`: **0 errors** across ~50k lines written by 42 different agents
- Prisma schema essentially interchangeable with the handwritten one; `Decimal` used everywhere for money (never float)
- Complete frontend coverage of every page in the spec
- Exemplary git discipline: granular Conventional Commits, one commit per green TDD cycle

**What failed**

- **81 / 265 test suites fail (370 / 963 tests)** — while every per-feature report claims green
- **Three parallel implementations of the same business logic** (Italian-named services from doc 1, English-named services from doc 2, plus a 1,317-line inline re-implementation created during final wiring)
- The **entire BullMQ/Redis queue architecture is dead code** — the runtime uses plain `setInterval`, exactly like the handwritten app
- The app ran on **hardcoded mocks** for most of its life; real wiring happened only at the very end and broke previously green tests
- 34× documentation expansion invented requirements (DDoS protection, performance monitoring, achievements…) that became entire features

**Root causes:** agent isolation without executable shared contracts · integration as a final phase instead of a continuous constraint · local "green" mistaken for global "green" · documentation expansion as a hallucination multiplier · global rules losing against local instructions.

Context rot did not strike *inside* the agents' contexts — it struck **in the spaces between agents**.

## Cloning

To get this repository **including both application repositories**:

```bash
git clone --recurse-submodules https://github.com/NECKER55/prompt-pipeline-analysis.git
```

If you already cloned it without submodules:

```bash
git submodule update --init --recursive
```

## Building the PDFs

```bash
cd ENG && pdflatex pipeline_analysis_finApp.tex && pdflatex pipeline_analysis_finApp.tex
cd ITA && pdflatex analisi_pipeline_finApp.tex && pdflatex analisi_pipeline_finApp.tex
```
