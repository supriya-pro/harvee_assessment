# A1. Manual Test Suite – Shop and Check Out (Sauce Demo)

## 1. Questions and assumptions

| # | Question for the product owner | Assumption I test against if unanswered |
|---|-------------------------------|------------------------------------------|
| Q1 | Is the postal code validated against a format or length (e.g. digits only, max length)? | Any non-empty value is accepted; whitespace-only is treated as empty only if the app does so today. |
| Q2 | Are first/last name trimmed, and are special characters or very long values allowed? | Non-empty values of any characters are accepted; no max length is enforced. |
| Q3 | Should the cart persist after logout/login, or across browser sessions? | Cart is client-side state and may persist for the same browser; I report the behaviour, not fail it. |
| Q4 | What exactly is the tax rule (fixed %, rounded how)? | Tax is about 8% of item total, rounded to 2 decimals; Total = item total + tax. |
| Q5 | Is the product sort order remembered after navigating away? | Not remembered; defaults to Name (A to Z) on each visit. |
| Q6 | Which browsers and screen sizes are supported? | Latest Chrome, Firefox, Edge, Safari; desktop 1280px and mobile 375px. |
| Q7 | Should error messages for locked-out and invalid users differ, and are they allowed to reveal that an account exists? | Messages as currently shown are the expected behaviour. |
| Q8 | What is the expected behaviour when the cart is empty and the user presses Checkout? | Checkout proceeds today; I log it as a question/risk rather than a defect. |
| Q9 | Is there an API or unit layer for pricing, tax and sorting that tests can target? | Unknown; the strategy assumes one may exist but this suite is UI only. |

## 2. Test cases

Test data: password for all users is the shared password shown on the login page. Users: `standard_user`, `locked_out_user`, `problem_user`, `performance_glitch_user`, `error_user`, `visual_user`.

| ID | Area | Type | Title | Pre-conditions | Steps | Test data | Expected result | Priority | Automate? |
|----|------|------|-------|----------------|-------|-----------|-----------------|----------|-----------|
| TC01 | Login | Positive | Standard user logs in | On login page | 1. Enter username 2. Enter password 3. Click Login | standard_user / shared password | Products page shown with 6 items | P1 | Yes – smoke test on every build |
| TC02 | Login | Negative | Locked-out user is rejected | On login page | Log in as locked-out user | locked_out_user | Error says the user has been locked out; stays on login page | P1 | Yes – business rule, stable |
| TC03 | Login | Negative | Wrong password rejected | On login page | Enter valid user and wrong password, Login | standard_user / wrong | Username and password do not match error | P2 | Yes – cheap, data-driven candidate |
| TC04 | Login | Negative | Empty username and empty password | On login page | Click Login with empty fields | none | "Username is required" error | P2 | Yes – parameterize with TC03 |
| TC05 | Sorting | Positive | Sort by price low to high | Logged in as standard_user | Select Price (low to high) | standard_user | All 6 prices in ascending order | P1 | Yes – whole list check |
| TC06 | Sorting | Positive | Sort by price high to low | Logged in | Select Price (high to low) | standard_user | All prices descending | P2 | Yes – same page object |
| TC07 | Sorting | Positive | Sort by name A to Z and Z to A | Logged in | Select each name sort | standard_user | Names ascending, then descending | P2 | Yes – list comparison |
| TC08 | Cart | Positive | Add two items, remove one | Logged in, empty cart | Add 2 items, remove 1 on products page | standard_user | Badge shows 1; cart has the remaining item only | P1 | Yes – core flow |
| TC09 | Cart | Positive | Remove an item from the cart page | 2 items in cart | Open cart, click Remove on one | standard_user | Badge decreases, item gone from list | P2 | Yes |
| TC10 | Cart | Boundary | Badge hidden when cart empty | Logged in, empty cart | Add one item, remove it | standard_user | Badge disappears at 0 | P3 | Yes – cheap |
| TC11 | Cart | Boundary | Add all 6 items | Logged in | Click Add to cart for all | standard_user | Badge shows 6; cart lists 6 | P3 | No – low value, covered by TC08 |
| TC12 | Checkout | Positive | Full checkout with totals | Logged in, 2 items in cart | Checkout, fill info, Continue, check overview, Finish | A / B / 12345 | Total = item total + tax; order confirmation; cart empty | P1 | Yes – primary revenue flow |
| TC13 | Checkout | Negative | Missing first name | On checkout step one | Leave first name blank, Continue | blank / B / 12345 | "First Name is required" | P1 | Yes – parameterized |
| TC14 | Checkout | Negative | Missing last name | On checkout step one | Leave last name blank | A / blank / 12345 | "Last Name is required" | P1 | Yes – parameterized |
| TC15 | Checkout | Negative | Missing postal code | On checkout step one | Leave postal code blank | A / B / blank | "Postal Code is required" | P1 | Yes – parameterized |
| TC16 | Checkout | Boundary | Very long names and special characters | On checkout step one | Enter 255-char name, `<script>` text, unicode | long / `<b>x</b>` / 12345 | Accepted or cleanly rejected; nothing rendered as HTML on overview | P2 | No – exploratory, needs judgement (Q2) |
| TC17 | Checkout | Positive | Cancel from overview keeps cart | At overview page | Click Cancel | standard_user | Returns to products; order not placed; items still in cart | P2 | Yes |
| TC18 | Checkout | Boundary | Whitespace-only postal code | On checkout step one | Enter spaces in postal code | A / B / "   " | Behaviour matches Q1 assumption; record result | P3 | No – depends on unanswered Q1 |
| TC19 | Security | Negative | Products page by URL after logout | Logged in, then logged out | Open /inventory.html directly | standard_user | Redirected to login with an error about needing to log in | P1 | Yes – access control |
| TC20 | Security | Negative | Checkout page by URL when logged out | Logged out | Open /checkout-step-one.html | none | Redirected to login, no data shown | P1 | Yes – extend TC19 |
| TC21 | Usability | Positive | Error messages are clear and dismissible | On login page | Trigger error, click the error close icon | locked_out_user | Message readable, close works, field highlighted | P3 | No – judgement |
| TC22 | Accessibility | Positive | Keyboard-only checkout | Logged in | Complete login and checkout using Tab/Enter only; run axe scan | standard_user | All controls reachable, visible focus, no critical axe violations, form fields have labels | P2 | No – manual plus separate axe job |
| TC23 | Browser/Screen | Positive | Layout at 375px and in Firefox/Safari | Logged in | Resize to 375px; repeat TC12 in Firefox and Safari | standard_user | No overlap or cut-off controls; flow completes | P2 | No – cross-browser matrix as a later grid job |
| TC24 | Resilience | Positive | Performance glitch user can still complete checkout | On login page | Log in as performance_glitch_user, run TC12 | performance_glitch_user | Flow completes; login within an agreed time (assume 5s) | P3 | No – timing varies, tracked manually |

### Exploratory charter  – problem_user

- **Mission:** Explore the shopping flow as `problem_user` to find where behaviour differs from `standard_user`.
- **Areas:** product images and detail pages, add/remove buttons on products and cart pages, sorting, checkout form fields (especially last name).

## 3. Strategy (under 150 words)

UI automation covers the P1 flows and business rules: login (TC01–04), sorting (TC05–07), cart (TC08–10), checkout (TC12–15, TC17) and the logout URL guard (TC19–20). These are stable, high value and run on every build. If the client has an API or unit layer, I would move tax and total calculation, sorting logic and the access-control check there, since they run faster and fail with clearer causes. The UI keeps one end-to-end smoke test. Manual only: usability, accessibility, cross-browser and responsive checks, long and special-character inputs, performance glitch timing, and the problem_user exploratory session. Biggest remaining risk: pricing and tax correctness across all products and users, because the UI tests check one cart combination and the tax rule is unconfirmed (Q4).
