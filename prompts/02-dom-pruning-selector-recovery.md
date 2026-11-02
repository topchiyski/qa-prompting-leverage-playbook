# 02. DOM Pruning & Self-Healing Selector Recovery

**90.6% of raw HTML tokens carry no meaning for an LLM** (inline CSS/SVGs, styling wrappers, nav bars, hydration JSON). With raw HTML, grounding accuracy is **46.8%**. With a pruned list of interactive elements it rises to **88.28%**, and token cost drops by over 90%.

### ❌ Bad Prompt
> My test failed because it can't find the Submit button anymore. Here is the entire 60,000-line HTML source of the page: `[FULL PAGE HTML WITH SCRIPTS, SVGS, AND CSS WRAPPERS]`. Find the button and fix my test.

**Why it fails:** The button is buried in noise. The model latches onto a wrapper class or invents an attribute, and when the button really is gone it still "finds" one.

### ✅ Good Prompt (Interactive)

```xml
<role>
Act as a Test Automation Architect.
</role>

<context>
The old selector .btn-submit-old broke on the /transfer page. These are the only visible interactive elements in the Transfer Form:
1. Dropdown labeled 'From Account' (id="fromAcct")
2. Input labeled 'Amount' (id="transferAmt")
3. Button with role 'button' and visible text 'Submit Transfer'
</context>

<constraints>
Pick the most resilient user-facing locator from the list above. Do not invent attributes that are not listed. If no element matches with 80%+ confidence, tell me the feature is missing instead of guessing.
</constraints>

<output_format>
Return the recommended Playwright locator and one sentence on why it won't break on future styling changes.
</output_format>
```

---

### ✅ Good Prompt (CI Self-Healing System Prompt)

```xml
<role>
You are a Deterministic Self-Healing Locator Recovery Engine for Playwright TypeScript.
</role>

<instructions>
Analyze <broken_locator_metadata> and <top_20_dom_candidates> extracted from the live page.
Identify the element that fulfills the original interaction intent and produce the most resilient Playwright selector.
</instructions>

<constraints>
1. Prefer selectors strictly in this order: getByRole > getByLabel > getByTestId > getByText > resilient CSS.
2. Never invent attributes not present in <top_20_dom_candidates>.
3. If the feature was removed or no candidate matches the intent with >= 0.80 confidence, set "selectedCandidateId" to null and "confidenceScore" below 0.50 so the test fails legitimately.
4. Output only valid JSON matching <output_schema>.
</constraints>

<output_schema>
{
  "selectedCandidateId": 3,
  "recommendedPlaywrightSelector": "page.getByRole('button', { name: 'Submit Transfer' })",
  "locatorStrategy": "getByRole | getByLabel | getByTestId | getByText | css",
  "confidenceScore": 0.92,
  "structuralChangeDiagnosis": "One sentence on what changed in the DOM"
}
</output_schema>

<broken_locator_metadata>
{original_selector_and_intent}
</broken_locator_metadata>

<top_20_dom_candidates>
{pruned_candidate_json}
</top_20_dom_candidates>
```

> Wrap the DOM candidates with the sandwich defense from [06, Pattern B](06-react-failure-triage-and-security.md#pattern-b-sandwich-defense-against-indirect-prompt-injection), because page text is untrusted input.
