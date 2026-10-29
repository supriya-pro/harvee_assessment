# Defect Report: Price sort order incorrect

| Field | Value |
|---|---|
| **Issue type** | Bug |
| **Summary** | Product list sorted by "Price (low to high)" shows a $29.99 item before a $15.99 item |
| **Affects version** | 4.2.0 |
| **Last known good version** | 4.1.x (to be confirmed by re-running on the previous build) |
| **Component** | Product listing / Sorting |
| **Severity** | Major |
| **Suggested priority** | High |
| **Regression** | Yes (reproduced on every run since 4.2.0) |
| **Reproducibility** | 100% (every run on 4.2.0) |
| **Labels** | regression, sorting, release-4.2.0 |

## Title

**[4.2.0][Chrome] Sort by "Price (low to high)" lists $29.99 item above $15.99 item**

## Environment

- **Application:** Client web shop, release **4.2.0**
- **Environment/URL:** `<test or staging URL>`
- **Browser:** Google Chrome `<version>`
- **OS:** `<OS and version>`
- **User/account:** `<test account used, e.g. standard user>`
- **Test data:** Default product catalogue (no filters applied)
- **Not yet verified:** other browsers, previous release 4.1.x (see Notes)

## Steps to Reproduce

1. Open the application in Chrome and log in with a standard test user.
2. Go to the product listing page.
3. Open the sort dropdown and select **"Price (low to high)"**.
4. Read the prices from top to bottom of the list.

## Expected Result

Items are ordered from the lowest price to the highest. The $15.99 item appears **before** the $29.99 item, and every item has a price greater than or equal to the one above it.

## Actual Result

The $29.99 item (`<item name>`) appears **above** the $15.99 item (`<item name>`), so the list is not in ascending price order.

- Observed order (top to bottom): `<fill in all prices as displayed>`
- Happens on every run since 4.2.0; not intermittent.

## Severity and Reasoning

**Severity: Major**

- The sort is a core browsing feature, and it fails at its one job. A customer relying on "low to high" to find the cheapest option is shown wrong information, which can lead to lost sales and loss of trust in the site.
- It reproduces 100% of the time and is a regression introduced in 4.2.0, so it will affect every user who uses this sort once released.
- It is not Critical: there is no data loss, no security impact, no crash, and the checkout flow is not blocked. Users can still see every price and buy the item they want.
- A workaround exists (scan prices manually or choose another sort), which keeps it below Critical, but the workaround defeats the purpose of the feature, so it stays above Minor.
- Release recommendation: should be fixed before 4.2.0 goes to production, or be an explicit, documented release decision.

## Evidence to Attach

1. **Screenshot** of the listing with "Price (low to high)" selected, with the two items and the sort control visible and the out-of-order prices circled.
2. **Screen recording** (short) of steps 1 to 4, showing it happens from a clean start.
3. **Page text or DOM export** of the product names and prices in displayed order, so the exact order can be checked without relying on the image.
4. **Browser console log** (checking for errors at the time of sorting).
5. **Network capture (HAR)** of the sort action, to show whether the order is wrong in the API response or only in the front-end rendering.
6. **Build/version proof:** screenshot or text of the app version showing 4.2.0.
7. **Comparison run on the previous build** (4.1.x), showing the correct order, to confirm the regression.
8. **Automated test result**, if the Section A test failed: test name, run ID, and failure output with the actual vs expected order.

## Notes for the Developer

- Please check whether the sort compares prices as text instead of numbers, or whether the sort key changed in 4.2.0 (for example, a change in how prices are stored or formatted). I have not confirmed the root cause; this is a lead, not a conclusion.
- The network capture will show whether the API returns the wrong order or the front end reorders it.
- Not yet checked: other sort options (high to low, A to Z, Z to A), other browsers, and whether the issue depends on the data set. I will add results as a comment once tested.
