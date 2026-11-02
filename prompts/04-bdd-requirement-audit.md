# 04. Two-Stage Requirement Audit & Declarative BDD

Generating Gherkin straight from an incomplete user story bakes its gaps into your automation. Have the model audit the story first.

### ❌ Bad Prompt
> Write Gherkin Cucumber scenarios for our bulk employee CSV upload story (max 500 rows, skip duplicate emails).

**Why it fails:** The model fills every gap with silent assumptions (encoding, partial failures, concurrency) and writes UI-coupled steps like "When I click Upload."

### ✅ Good Prompt (Audit, Then Specify)

```xml
<role>
Act as a Principal QA Architect and BDD Specialist.
</role>

<context>
User Story: "As an enterprise admin, I want to bulk-upload employee CSV files (up to 500 rows) so that accounts are provisioned automatically. Duplicate emails should be skipped and reported."
</context>

<instructions>
Respond in two parts:
Part 1 (<requirement_audit>): List 4 critical ambiguities or missing acceptance criteria (file encoding, partial-failure rollback vs. commit, concurrent upload locks, rate limits).
Part 2 (<gherkin_specification>): Write a declarative Gherkin feature file using `Scenario Outline` with `Examples` tables, tagged `@positive`, `@boundary`, and `@negative_security`.
</instructions>

<constraints>
- Declarative only: no clicks, CSS selectors, URLs, or HTML element types in Given/When/Then steps.
- Boundary cases: 0 rows, 1 row, 500 rows (max valid), 501 rows (over the limit).
- Negative cases: CSV formula injection (`=CMD|...`) and malformed UTF-8 byte sequences.
</constraints>
```
