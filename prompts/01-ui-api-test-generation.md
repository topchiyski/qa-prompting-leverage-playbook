# 01. UI & API Test Generation (Playwright & Pytest)

## Scenario A: Checkout Flow with API Mocking (Playwright)

### ❌ Bad Prompt
> Write a Playwright test in TypeScript for the user checkout flow. Mock the payment API and check that the order confirmation appears after clicking Pay Now.

**Why it fails:**
* The model invents fragile CSS selectors (`div.checkout-footer > button.btn-pay`).
* It adds `await page.waitForTimeout(5000)`, which slows CI and still flakes.
* It writes `expect(text).toBeTruthy()`, which passes even when an error message is shown.

### ✅ Good Prompt

```xml
<role>
Act as a Principal QA Engineer specializing in Playwright v1.50+ with TypeScript.
</role>

<context>
Target URL: /checkout
Pruned DOM Excerpt:
<form aria-label="Payment Details">
  <label for="cc-num">Card Number</label>
  <input id="cc-num" name="cardNumber" type="text" required />
  <button type="submit">Pay Now</button>
</form>
<section role="status" aria-live="polite" data-testid="order-summary" hidden>
  <h1>Order Confirmed</h1>
  <p data-testid="order-id-display"></p>
</section>

API Contract: POST **/api/v1/payments/charge
Request Body: { "cardNumber": string, "currency": "USD" }
Response (201 Created): { "transactionId": "txn_99481", "status": "SUCCEEDED" }
</context>

<constraints>
1. Architecture: A `CheckoutPage` POM class with `readonly` Locator properties set in the constructor, plus a separate `checkout.spec.ts` test file.
2. Locators: Use only `getByRole`, `getByLabel`, and `getByTestId`. No CSS class selectors or XPath.
3. Waiting: `page.waitForTimeout()` is forbidden. Rely on auto-waiting and web-first assertions (`await expect(...).toBeVisible()`).
4. Network: Use `page.route()` on `**/api/v1/payments/charge` to validate the request payload and fulfill with a deterministic 201 JSON mock.
</constraints>

<oracle>
After submitting payment:
- The POST request body equals `{ cardNumber: "4111222233334444", currency: "USD" }`.
- Heading "Order Confirmed" is visible.
- `order-id-display` has the exact text "txn_99481".
</oracle>

<output_format>
First list 3 synchronization risks as bullets, then provide `CheckoutPage.ts` and `checkout.spec.ts`.
</output_format>
```

---

## Scenario B: API Schema & Boundary Validation (Pytest)

### ❌ Bad Prompt
> Write pytest tests for the `/api/v1/discounts/calculate` endpoint to test different user tiers and invalid inputs.

**Why it fails:** The model has no schema, so it guesses field names, valid ranges, and error codes, and it skips boundary values.

### ✅ Good Prompt

```xml
<role>
Act as a Staff Python QA Architect specializing in Python 3.13, Pytest, and Pydantic v2 contract validation.
</role>

<context>
Endpoint: POST /api/v1/discounts/calculate
OpenAPI Request Schema:
{
  "price": {"type": "number", "minimum": 0.0},
  "tier": {"type": "string", "enum": ["regular", "student", "premium"]},
  "coupon_code": {"type": "string", "maxLength": 12, "nullable": true}
}
OpenAPI Response Schema (200 OK):
{
  "final_price": float,
  "discount_applied": float,
  "currency": "EUR"
}
Existing fixture in `tests/conftest.py`: `api_client` (configured `httpx.Client`).
</context>

<constraints>
1. Validate every 200 OK response against a strict Pydantic v2 `DiscountResponse` model.
2. Put happy-path and zero-price boundary cases in one `@pytest.mark.parametrize` test, using `pytest.approx` for floats.
3. Add a negative matrix asserting 422 for a negative price, an invalid tier, and a coupon longer than 12 characters.
4. Use snake_case and short docstrings. Output Python code only.
</constraints>
```
