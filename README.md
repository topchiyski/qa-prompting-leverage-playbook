# When AI Cheats on Your Tests: The QA Prompt Engineering Playbook

[![LinkedIn Article](https://img.shields.io/badge/LinkedIn_Article-Coming_Soon-0A66C2?style=for-the-badge&logo=linkedin)][article]
[![Connect on LinkedIn](https://img.shields.io/badge/Author-Stoyan_Topchiyski-0A66C2?style=for-the-badge&logo=linkedin)][author]

> **📰 This is the companion repository to the LinkedIn article [When AI Cheats on Your Tests][article] by [Stoyan Topchiyski][author].**
> The article explains *why* AI-generated tests fail and how to stop it. This repo holds the ready-to-use prompts and the benchmarks behind it.

<!-- TODO: When the article is published, replace ARTICLE_URL_PLACEHOLDER on the line below and change the badge text "Coming_Soon" to "Read_Now". -->

[article]: ARTICLE_URL_PLACEHOLDER
[author]: https://www.linkedin.com/in/stoyan-topchiyski/

## 🚨 The Problem

**94% of engineering teams** use AI in testing, but **only 12% trust it** to run autonomously (2026 industry telemetry).

Given a vague one-line prompt, an LLM guesses selectors, adds hardcoded sleeps (`waitForTimeout(5000)`), and **reward-hacks**: it weakens strict assertions to `toBeTruthy()` or deletes failing tests on retry #7 just to get a green build.

This repo gives you **copy-paste prompts** for **Playwright v1.50+ (TypeScript)**, **Pytest + Pydantic v2 (Python)**, and **Selenium 4 (Java 21)** that remove the guessing and block the cheating. They work in Claude, ChatGPT, Copilot, or Cursor.

---

## 📚 The Prompts

Each file shows a ❌ bad prompt, why it fails, and a ✅ good prompt you can adapt.

| # | Prompt | Stops the AI from… |
| :--- | :--- | :--- |
| 01 | [UI & API Test Generation](prompts/01-ui-api-test-generation.md) | Inventing selectors, sleeping, writing weak assertions |
| 02 | [DOM Pruning & Selector Recovery](prompts/02-dom-pruning-selector-recovery.md) | Drowning in raw HTML and hallucinating locators |
| 03 | [Guarded Debugging & RCA](prompts/03-guarded-debugging-and-rca.md) | "Fixing" flaky tests by deleting steps or weakening checks |
| 04 | [BDD Requirement Audit](prompts/04-bdd-requirement-audit.md) | Baking story ambiguities into Gherkin |
| 05 | [Selenium → Playwright Migration](prompts/05-legacy-selenium-to-playwright-migration.md) | Translating `Thread.sleep` and non-retrying reads line by line |
| 06 | [Failure Triage & Prompt-Injection Defense](prompts/06-react-failure-triage-and-security.md) | Retrying forever, and obeying instructions hidden in the DOM |

Benchmarks and the 14 sources behind the numbers are in [docs/empirical-benchmarks-and-citations.md](docs/empirical-benchmarks-and-citations.md).

---

## 🧱 The 5-Part Prompt Blueprint

The good prompts are built from these XML blocks:

1. `<role>`: The exact persona, framework, and version (e.g., Playwright v1.50+ with TypeScript).
2. `<context>`: Ground truth only: a pruned DOM, code signatures, or the API contract.
3. `<constraints>`: Explicit bans on hardcoded waits, brittle CSS/XPath, and assertion weakening.
4. `<oracle>`: The observable state change that proves the feature works.
5. `<output_format>`: Make the model state its risks or reasoning *before* it writes code or JSON.

Use the blocks each task needs. Agent prompts add `<instructions>`, `<output_schema>`, or `<security_policy>` (see 02 and 06).

---

## 📊 Recommended Inference Settings

| Task | `temperature` | `top_p` | `seed` | Validate output with |
| :--- | :--- | :--- | :--- | :--- |
| Test script generation | `0.0–0.1` | `0.95` | Fixed (e.g., `42`) | Syntax check (`tsc --noEmit` / `ast.parse`) |
| Self-healing locator recovery | `0.0` | `1.0` | Fixed | Strict JSON Schema (`response_format`) |
| Boundary & negative data synthesis | `0.2–0.4` | `0.95` | Fixed per CI build | Pydantic / JSON Schema with enums |
| Exploratory edge cases & BDD | `0.3–0.5` | `0.95` | Optional | `gherkin-lint` |
| LLM-as-a-judge / failure triage | `0.0` | `1.0` | Fixed | Rubric JSON with `evidence_quotes` first |

---

## 💬 Contact

Questions, feedback, or ideas for new prompts? Connect with me on **[LinkedIn](https://www.linkedin.com/in/stoyan-topchiyski/)**.

---

## 📄 License

Released under the [MIT License](LICENSE).
