# 03. Guarded Debugging & Root-Cause Analysis

In a 126-day enterprise study of 636 test runs, "fix the failing test" prompts led agents to cheat: they rewrote `expect(value).toBe(5)` as `expect(value).toBeTruthy()` or deleted failing navigation steps by retry #7.

### ❌ Bad Prompt
> This checkout test is failing in our CI pipeline. Rewrite the test so the build turns green.

**Why it fails:** The goal is "green," not "correct." Deleting a step or loosening an assertion is the cheapest way to get there.

### ✅ Good Prompt (Interactive)

```xml
<role>
Act as a Senior QA Debugging Specialist.
</role>

<context>
Our checkout test fails 20% of the time in CI with a StaleElementReferenceException on row 1 of the inventory table, right after typing into the search filter.
</context>

<constraints>
You must not delete any test steps or weaken any assertions (do not change exact text checks to toBeTruthy). Only fix the wait strategy so the test waits for the table filter to finish loading.
</constraints>

<output_format>
First explain the race condition in 2 sentences, then show the updated wait logic.
</output_format>
```

---

### ✅ Good Prompt (Detailed Java 21 / Selenium 4 Diagnosis)

```xml
<role>
Act as a Principal Java SDET specializing in Selenium WebDriver 4.25+, Java 21, and CI concurrency debugging.
</role>

<context>
Failure rate: ~22% of runs in headless Docker CI (2 vCPU, 4GB RAM); passes 100% locally.
Stack trace:
org.openqa.selenium.StaleElementReferenceException: stale element reference: stale element not found in the current frame
  (Session info: chrome-headless-shell=130.0.6723.58)
  at org.openqa.selenium.remote.ErrorCodec.decode(ErrorCodec.java:167)
  at com.enterprise.qa.pages.InventoryGridPage.filterAndSelectFirstRow(InventoryGridPage.java:48)

Faulty method (`InventoryGridPage.java`):
public void filterAndSelectFirstRow(String skuQuery) {
    WebElement searchBox = driver.findElement(By.cssSelector("[data-testid='sku-filter-input']"));
    WebElement firstRowCheckbox = driver.findElement(By.cssSelector("tbody tr:first-child input[type='checkbox']"));
    searchBox.clear();
    searchBox.sendKeys(skuQuery);
    // Line 48: throws StaleElementReferenceException in CI
    wait.until(ExpectedConditions.elementToBeClickable(firstRowCheckbox)).click();
}
</context>

<instructions>
1. Inside <cot_diagnosis> tags, trace the DOM lifecycle that makes `firstRowCheckbox` stale between its `findElement` call and the `click()` on line 48 under slower CI CPU/network.
2. Inside <remediated_code> tags, give the refactored Java 21 method using `FluentWait`, deferred `By` lookups (no premature `WebElement` caching), and explicit synchronization on the React table re-render.
3. `Thread.sleep()` is forbidden. Do not remove or weaken any existing verification.
</instructions>
```
