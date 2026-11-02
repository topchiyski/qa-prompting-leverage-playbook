# 05. Legacy Migration: Java/Selenium 4 → TypeScript/Playwright

### ❌ Bad Prompt
> Convert this Java Selenium test class into TypeScript Playwright.

**Why it fails:** The model transliterates line by line instead of mapping semantics:
* `Thread.sleep(3000)` becomes `await page.waitForTimeout(3000)`.
* `Assert.assertEquals(el.getText(), ...)` becomes a non-retrying `expect(await el.innerText()).toBe(...)`, which races immediately.
* XPath `@FindBy` locators are copied over unchanged.

### ✅ Good Prompt (Semantic Equivalence + Auto-Waiting)

```xml
<role>
Act as a Principal Migration Architect specializing in converting legacy Java Selenium/TestNG suites to TypeScript Playwright v1.50+.
</role>

<context>
Legacy Java Selenium class (`AccountTransferTest.java`):
public class AccountTransferTest extends BaseSeleniumTest {
    @FindBy(xpath = "//div[@class='acct-panel']//select[@id='fromAcct']")
    private WebElement fromAccountDropdown;
    @FindBy(id = "transferAmt")
    private WebElement amountField;
    @FindBy(xpath = "//button[contains(text(),'Submit Transfer')]")
    private WebElement submitBtn;
    @FindBy(css = ".toast-msg.success")
    private WebElement successBanner;
    @FindBy(css = "#balance-val")
    private WebElement remainingBalance;

    @Test(dataProvider = "transferAmounts")
    public void testValidTransfer(String acctId, String amount, String expectedBalance) throws InterruptedException {
        driver.get(baseUrl + "/banking/transfer");
        new Select(fromAccountDropdown).selectByValue(acctId);
        amountField.clear();
        amountField.sendKeys(amount);
        submitBtn.click();
        Thread.sleep(3000);
        Assert.assertTrue(successBanner.isDisplayed(), "Success banner must be visible");
        Assert.assertEquals(remainingBalance.getText().trim(), expectedBalance, "Balance mismatch");
    }
}
</context>

<constraints>
1. Keep every assertion: the success notification must be visible, and the trimmed remaining balance must match exactly for each dataset row.
2. Remove `Thread.sleep(3000)`. Convert state reads (`isDisplayed()`, `getText()`) into web-first assertions (`toBeVisible()`, `toHaveText()`).
3. Replace `@FindBy` XPaths with semantic Playwright locators (`getByRole`, `getByLabel`, `locator.selectOption`).
4. Put the POM class (`TransferPage.ts`) and the parameterized suite (`transfer.spec.ts`) in separate files.
</constraints>
```
