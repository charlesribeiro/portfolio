# 15 — Representing AI Engineering

## 1. Problem

AI engineering is a real and growing specialization, but the conversion target is primarily **Senior Frontend** roles. If AI dominates, Charles reads as a generalist following a trend. If it is hidden, Charles loses AI-enabled roles and misses a real differentiator.

## 2. Framing

> **A senior frontend engineer who builds AI-powered product experiences and uses AI rigorously in engineering.**

AI is presented as an **extension of product and frontend engineering**: interfaces for LLM features (streaming UX, tool-call visualization, uncertainty and error states, evals in the loop), integrations (MCP, RAG), and disciplined AI-assisted development.

## 3. Rules

| ID | Rule |
|---|---|
| AI-01 | The `<h1>`, `<title>`, `jobTitle` (JSON-LD) and `/llms.txt` summary lead with **Senior Frontend Engineer**. AI appears in the second sentence or line (`secondaryFocus`). |
| AI-02 | AI never has its own separate job title on the site ("AI Engineer") unless a real role held that title. |
| AI-03 | Home page weight: AI gets at most one strip and at most one of the three flagship cards. |
| AI-04 | Every AI capability in `/ai` links to evidence (a project, publication, course or repo). Nothing is listed without evidence (CONTENT-09). |
| AI-05 | No hype vocabulary ("revolutionary", "10×", "AI-native ninja"). Plain, specific language: "built an MCP server that…", "co-authored a book on Small Language Models". |
| AI-06 | The AI hub shows judgement: when *not* to use an LLM, evals, cost and latency trade-offs, and privacy. Senior readers look for this. |
| AI-07 | The site's own agent-driven development process (19) is presented as a case study of AI-assisted engineering done with guardrails. It connects the two identities directly. |

## 4. Where AI appears

| Surface | Treatment |
|---|---|
| Home | Supporting line + one strip + ≤ 1 flagship |
| `/ai` | Full hub (02) |
| `/work` | Theme filter `ai-engineering` |
| `/writing` | SLM and Tailwind CSS + AI publications, Alura AI content |
| `/colophon` | Agent workflow, CI gates, agent-readability eval results |
| JSON-LD | `knowsAbout` includes LLM, RAG, MCP, AI agents as DefinedTerms with Wikidata `sameAs` where they exist, each backed by visible content |
