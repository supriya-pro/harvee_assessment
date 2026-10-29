# B1. Review of AI-Generated Cart Total Test

A teammate asked an AI assistant to "write a test that checks the cart total" and wants to merge the result. Below are the problems found, followed by a rewritten test.

## Original Code

```ts
test('cart total is correct', async ({ page }) => {
  await page.goto('https://www.saucedemo.com');
  await page.fill('#user-name', 'standard_user');
  await page.fill('#password', 'secret_sauce');
  await page.click('#login-button');
  await page.click('text=Add to cart');
  await page.waitForTimeout(3000);
  await page.click('.shopping_cart_link');
  await page.click('#checkout');
  await page.fill('#first-name', 'A');
  await page.fill('#last-name', 'B');
  await page.fill('#postal-code', '1');
  await page.click('#continue');
  const total = await page.locator('.summary_total_label').textContent();
  expect(total).toBeTruthy();
});
```

## Problems

### Correctness (the test can't fail when it should)

1. **`expect(total).toBeTruthy()` verifies nothing.** Any non-empty string passes, including "Total: $0.00" or a wrong amount. The test is named "cart total is correct" but never checks the total.
2. **No expected value is defined.** A real test needs a known item, its price, and the expected tax and total (Backpack $29.99 + $2.40 tax = $32.39).
3. **Nothing is checked on the way.** Login success, cart contents and the item subtotal are never asserted. If the wrong item were added, the test would still pass.

### Flakiness and speed

4. **`waitForTimeout(3000)` is a hard sleep.** It adds 3 seconds to every run and still isn't a guarantee. Playwright auto-waits on actions and web-first assertions, so it should be deleted.
5. **`textContent()` doesn't retry.** It reads the value once, so it can race the page. Use `expect(locator).toHaveText(...)`, which retries.
6. **`text=Add to cart` is ambiguous.** It matches six buttons, and `click` silently takes the first. The test depends on product ordering and doesn't say which item it added.

### Maintainability and hygiene

7. **Brittle selectors.** `.shopping_cart_link`, `.summary_total_label` and `#id` selectors are tied to styling and markup. Prefer `data-test` attributes or role/label locators (`getByRole`, `getByPlaceholder`).
8. **Hardcoded URL and credentials.** They belong in `baseURL` config and env vars or fixtures. Login is also repeated inline, so it should be a fixture, `beforeEach`, or reused `storageState`.
9. **Vague name and no structure.** There are no `test.step`s, no `describe`, and the imports for `test` and `expect` are missing.
10. **Dummy data (`'A'`, `'B'`, `'1'`) is unexplained.** Name it as test data.
11. **No scope note.** It runs against a public third-party demo site, so the team should know that dependency exists.

## Rewrite

```ts
import { test, expect } from '@playwright/test';

// Known catalogue data for the demo store (8% tax, rounded to cents)
const ITEM = { id: 'sauce-labs-backpack', name: 'Sauce Labs Backpack', price: 29.99 };
const TAX = 2.40;
const TOTAL = 32.39;

const user = {
  name: process.env.SAUCE_USER ?? 'standard_user',
  password: process.env.SAUCE_PASS ?? 'secret_sauce',
};
const shipping = { first: 'Test', last: 'User', zip: '12345' };

test.describe('Checkout totals', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/'); // baseURL: 'https://www.saucedemo.com' in playwright.config
    await page.locator('[data-test="username"]').fill(user.name);
    await page.locator('[data-test="password"]').fill(user.password);
    await page.locator('[data-test="login-button"]').click();
    await expect(page).toHaveURL(/inventory\.html/);
  });

  test('order total equals item price plus tax for a single item', async ({ page }) => {
    await test.step('add a specific item to the cart', async () => {
      await page.locator(`[data-test="add-to-cart-${ITEM.id}"]`).click();
      await expect(page.locator('[data-test="shopping-cart-badge"]')).toHaveText('1');
    });

    await test.step('verify the cart contents', async () => {
      await page.locator('[data-test="shopping-cart-link"]').click();
      await expect(page.locator('[data-test="inventory-item-name"]')).toHaveText(ITEM.name);
    });

    await test.step('enter checkout details', async () => {
      await page.locator('[data-test="checkout"]').click();
      await page.locator('[data-test="firstName"]').fill(shipping.first);
      await page.locator('[data-test="lastName"]').fill(shipping.last);
      await page.locator('[data-test="postalCode"]').fill(shipping.zip);
      await page.locator('[data-test="continue"]').click();
    });

    await test.step('verify the order summary amounts', async () => {
      await expect(page.locator('[data-test="subtotal-label"]')).toHaveText(`Item total: $${ITEM.price.toFixed(2)}`);
      await expect(page.locator('[data-test="tax-label"]')).toHaveText(`Tax: $${TAX.toFixed(2)}`);
      await expect(page.locator('[data-test="total-label"]')).toHaveText(`Total: $${TOTAL.toFixed(2)}`);
    });
  });
});
```

## Notes

- Hardcoding the expected total is deliberate. Computing it from the page would repeat the app's own logic and could hide a bug. If you want a second guard, add a check that subtotal + tax equals total.
- This has not been run against the live site, so confirm the `data-test` values (the current Sauce Demo ones were used) before merging.
- For more than one test, move the login into a fixture or reuse `storageState` to keep the suite fast.
