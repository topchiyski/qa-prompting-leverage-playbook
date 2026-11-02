# Empirical Benchmarks & Sources

| Failure mode | Why it happens | Evidence | Countermeasure |
| :--- | :--- | :--- | :--- |
| **Raw HTML token bloat** | Inline SVGs (27.9%), CSS wrappers (22.9%), nav bars (21.1%), and hydration JSON (19.0%) flood the context window. | **90.6%** of raw HTML tokens are non-semantic; grounding accuracy is only **46.8%**. | **DOM pruning** (e.g., Prune4Web) cuts tokens 90–99.7% and lifts accuracy to **88.28%**. → [Prompt 02](../prompts/02-dom-pruning-selector-recovery.md) |
| **Regex / `.text()` pruning trap** | Stripping tags with regex or Cheerio `.text()` loses hierarchy and attributes. | Keeps only **9.7–16.7%** of structure score, breaking selector synthesis. | Filter on the parsed tree (**LinkeDOM / Parse5**) or send **tiered candidate JSON** (92.4%+ structure score). |
| **Autonomous repair cheating** | Repair agents optimize for a green exit code with no correctness boundary. | **20-point gap** between naive (70%) and strict (50%) convergence, caused by weakened assertions and deleted tests. | **Immutable assertion guardrails**; log every self-heal as a reviewable CI diff. → [Prompt 03](../prompts/03-guarded-debugging-and-rca.md) |
| **Infinite repair loops** | Retrying symptoms of an underlying **Generator** (stale schema, environment outage). | **37.7%** code-gen collapse; up to 16 futile retries at $0.03–$0.15 per run. | Classify **Layers vs. Generators**; enforce a **retry budget of 3–4**. → [Prompt 06](../prompts/06-react-failure-triage-and-security.md) |
| **Prompt drift & judge flakiness** | Unpinned models, high temperature, and unvalidated prose outputs in CI. | Prompts are tested in only **~1%** of agent test suites; flaky judges flip pass/fail on formatting. | `temperature=0`, link every CI trace to its prompt hash, use **evidence-first rubric schemas**. |

## Sources

1. [TestDino (2026) - State of AI in Test Automation: Data + Reality](https://testdino.com/blog/state-of-ai-automation)
2. [Santana et al. (2025) - Software Testing with Large Language Models: An Interview Study (arXiv)](https://arxiv.org/pdf/2510.17164)
3. [Lee (2026) - Practical Limits of Autonomous Test Repair: A Multi-Agent Case Study (arXiv)](https://arxiv.org/html/2605.01471v1)
4. [Yang et al. (2026) - Large Language Models for Software Testing Education (arXiv)](https://arxiv.org/html/2603.26329v1)
5. [NashTech Global - 7 Prompt Engineering Practices Every Tester Should Know](https://blog.nashtechglobal.com/7-prompt-engineering-practice-every-tester-should-know/)
6. [QASkills (2026) - AI Test Generation with Playwright + Copilot Guide](https://qaskills.sh/blog/ai-test-generation-playwright-copilot)
7. [Nath (2026) - How to Clean Scraped Web HTML to Reduce LLM Token Costs by 90%](https://plainenglish.io/web-scraping/how-to-clean-scraped-web-html-to-reduce-llm-token-costs-by-90-in-nodejs-ykz1z6)
8. [Prune4Web (2025) - DOM Tree Pruning Programming for Web Agents (arXiv)](https://arxiv.org/html/2511.21398v1)
9. [FocusAgent - Simple Yet Effective Ways of Trimming Large Web Contexts (OpenReview)](https://openreview.net/forum?id=mINaJKSy7A)
10. [Karate Labs - The Test Pyramid Isn't Inverting. It's Widening](https://karatelabs.io/blog/test-pyramid-widening)
11. [AIMS-AI (2026) - Playwright AI Test Generation with Claude & MCP](https://playwright.aims-ai.com/blog/playwright-ai-test-generation-claude)
12. [Augusto et al. (2025) - Large Language Models for Software Testing: A Research Roadmap (arXiv)](https://arxiv.org/abs/2509.25043)
13. [CloudQA (2026) - The Ultimate Guide to GPT Prompts for Test Case Generation](https://cloudqa.io/gpt-prompts-test-case-generation-qa-automation-2025/)
14. [DevelopersVoice - The AI Test Engineer: Automating Test Case Generation](https://developersvoice.com/blog/ai/llms-for-test-case-generation/)
