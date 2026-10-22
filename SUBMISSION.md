# SauceDemo QA Assessment

**Application:** https://www.saucedemo.com/  
**Account used for observed standard-user checks:** `standard_user` / `secret_sauce`  
**Execution date:** 2026-10-22  
**Scope note:** Findings below are limited to the browser observations recorded in this session. No defect is claimed without a reproduced mismatch against expected behavior.

## A — Risk-based exploratory testing

### Prioritized journeys and checks

| Priority / journey | Risk or exploratory question | Status | Execution/result |
|---|---|---|---|
| P1 — Login | Can the supplied standard account enter the inventory? | Test | **Pass observed.** Login reached `/inventory.html`. |
| P1 — Inventory to cart | Does adding a selected product produce the matching cart item and price? | Test | **Pass observed.** Added Sauce Labs Backpack; cart badge was `1`; cart showed the Backpack at `$29.99`. |
| P1 — Checkout | Are cart contents and computed totals consistent through order completion? | Test | **Pass observed.** One Backpack; subtotal `$29.99`, tax `$2.40`, total `$32.39`; completion displayed “Thank you for your order!”. |
| P1 — Required checkout data | Does checkout block submission if required information is missing? | Test | **Pass observed.** Empty form remained on checkout information and displayed `Error: First Name is required`. This is validation, not a defect. |
| P1 — Sorting | Do price sort options order displayed prices correctly? | Test | **Pass observed.** Low-to-high: `$7.99, $9.99, $15.99, $15.99, $29.99, $49.99`; high-to-low: `$49.99, $29.99, $15.99, $15.99, $9.99, $7.99`. Equal-price tie order was not checked. |
| P1 — Special account behavior | Does `locked_out_user` show its intended restriction, and are other demo accounts' differences intentional? | Needs validation | Not tested. Account names are listed by the login page, but their outcomes were not exercised. Do not report an expected demo-user variant as a defect without first identifying the account and reproducing it. |
| P2 — Session handling | After logout/re-login or a reload, what happens to authentication and cart state? Does a direct protected-page visit redirect? | Needs validation | Not tested. No session persistence/expiry claims made. |
| P2 — Cart edits | Can a user remove an item, return to shopping, and continue checkout with an empty or changed cart? | Needs validation | Not tested. |
| P2 — Checkout data boundaries | What formats/lengths are accepted for name and postal fields; what happens if a required field is omitted individually? | Needs validation | Only the all-empty form was checked; individual missing fields, whitespace, and boundary values were not. |
| P3 — UX/accessibility | Keyboard-only navigation, focus/error announcement, small viewport layout, image alt text and contrast | Needs validation | Not tested; no accessibility or responsive claims made. |
| P3 — Performance | Measure route timings and repeated-load behavior on controlled network/device conditions | Needs validation | Not measured. A single interactive run is not a performance test. |

### Additional high-value checks not yet executed

These remain hypotheses for a follow-up session, not claims about current behavior.

| Journey | Functional / edge risk | UX/UI or performance question | Status |
|---|---|---|---|
| Login | Wrong password, blank username/password, and account-specific outcomes | Is the error understandable, keyboard-accessible, and recoverable? Measure login response only under a controlled baseline. | Needs validation |
| Inventory | Product detail navigation, add/remove state consistency, and all four sort choices (only price directions were checked) | Do controls and item order remain clear at narrow viewport / keyboard use? Does the list render within an agreed performance budget? | Needs validation |
| Cart | Remove one of several items, return to shopping, and reconcile badge with remaining rows | Are empty-cart and changed-cart states understandable? Observe route responsiveness under repeatable conditions. | Needs validation |
| Checkout | Each required field omitted separately; whitespace and representative postal formats | Is validation associated with the relevant field and usable by keyboard/screen reader? No checkout performance baseline was collected. | Needs validation |
| Session | Logout, reload, direct inventory URL, and state after browser-context restart | Does the UI communicate logged-out state and preserve only intended state? No session-duration/security conclusion is drawn. | Needs validation |

### Rejected suggestions

| Suggestion | Status | Reason |
|---|---|---|
| Treat a `locked_out_user` login denial as a product defect | Discard | The application explicitly lists this as a distinct demo account; its expected account-specific behavior must not be confused with standard-user login. The account was not exercised here. |
| Test payment-card authorization, shipping address selection, inventory stock depletion, or refunds | Discard | These features were not observed in the tested SauceDemo workflow. Inventing those screens or backend semantics would make the test invalid. |
| Raise the required-name validation as a defect | Discard | The blank submission was blocked with an explicit “First Name is required” message, consistent with the field being required. |
| Claim that equal-priced products are mis-sorted | Discard | Both `$15.99` products appeared in each price-sorted list; no incorrect ordering requirement between equal values was established. |

## B — Defect report

**No confirmed defects.** The exercised standard-user login, price sorting, cart contents, checkout arithmetic, completion, and blank-form validation matched the observations expected for those checks. This is not a claim that the application has no defects. Other user variants, session behavior, individual validation edges, responsive/accessibility behavior, and performance were not tested.

No screenshots/recordings were saved during the live checks. The report intentionally avoids fabricated evidence and severity ratings.

## C — Automated test

One Playwright TypeScript test is in `tests/checkout.spec.ts`. It performs login → add Backpack → cart → checkout → confirmation, and asserts the product name/price in cart, subtotal, tax, total, and final confirmation. `tests/pages/locators.ts` keeps selectors separate from page actions in `tests/pages/actions.ts`. The test relies on Playwright locator auto-waiting and assertions, not fixed sleeps. Each test receives a fresh Playwright browser context.

**Verification status:** `npx tsc --noEmit` passed. The final standard `npm test` run passed 1/1 against live SauceDemo in Chromium. Several no-retry authoring runs failed at initial navigation with Chromium `net::ERR_ABORTED`, while other runs completed in a few seconds. One Playwright retry is configured, and tests that pass only on retry are reported as flaky. Do not read one green run as proof that external navigation is stable. Use `README.md` for a clean-machine run. This workspace is not a public GitHub repository.

### AI assistance, review, and limitations

- AI helped organize a risk-based checklist, draft the POM/test structure, and assemble the written submission.
- AI output was not accepted as evidence. Live browser interaction was used to verify standard login, actual product names/prices, sorting, one-item cart state, checkout totals, successful completion, and required-field feedback.
- A first checkout-field lookup used the wrong test-id casing. The live page showed `data-test="firstName"`; the selector was corrected. Separately, the first local test run timed out because Playwright's default `getByTestId` attribute is `data-testid`; the app exposes `data-test`. Live DOM inspection established the actual attribute, so `testIdAttribute: 'data-test'` was set in Playwright config. These were automation mistakes, not app defects.
- A later fresh browser navigation intermittently failed with Chromium `net::ERR_ABORTED`; a subsequent retry passed. A retry is configured to expose this as flaky rather than silently treating all attempts as clean.
- The test uses observed SauceDemo `data-test` selectors, but they remain coupled to the demo's current markup. The item-add selector is derived from the product name using SauceDemo's observed slug convention; verify it against the live app if the site changes.
- One happy path does not cover negative authentication, all user variants, cart mutations, other validation cases, browser engines, mobile layouts, accessibility, or meaningful load/performance thresholds.
- The test's credentials are demo credentials publicly shown by the application; do not replace them with private credentials or commit secrets.

## D — AI-driven QA strategy

AI is useful for drafting risk-based scenarios from requirements, proposing edge cases, summarizing failures, identifying coverage gaps, clustering regressions, and suggesting which tests to run after a code change. It can accelerate test scaffolding and repetitive data variations.

Human review is mandatory for requirement interpretation, deciding whether unusual behavior is intentional (especially distinct demo users), selecting business severity, checking test oracles and selectors against the actual UI, protecting credentials/data, and approving release decisions. An AI-generated claim is a hypothesis until reproduced or corroborated.

Before a generated test enters regression, a reviewer should: map assertions to a requirement or observed contract; verify behavior against the application; inspect selectors and test isolation; remove fixed waits and flaky state assumptions; run repeatedly and in CI; check failure diagnostics; and confirm the test fails for the intended regression rather than incidental implementation details. Keep exploratory notes separate from verified defects.

For defect analysis, AI may cluster logs and suggest likely causes, but engineers reproduce issues and validate the proposed cause. For coverage, compare journeys and risk areas rather than maximizing test count. For regression optimization, use ownership/change mapping and historical failures to prioritize tests, while retaining a periodic full suite to detect gaps.

In CI/CD, run deterministic smoke tests on pull requests, broader regression on scheduled or pre-release builds, and block promotion on agreed critical checks. Preserve traces/screenshots on failure, report flakiness separately, and never let an AI summary alone override a failed quality gate.

Key risks are fabricated behavior, biased/incomplete coverage, brittle selectors, nondeterministic test code, privacy leakage, and false confidence from green but weak assertions. Mitigate with source/DOM verification, human review, stable observable assertions, least-privilege test data, repeated CI runs, explicit uncertainty, and independent monitoring.

## E — AI prompt/usage log

Entries below quote actual user prompts from this conversation, not reconstructed prompts.

| Prompt | Purpose | Useful output / validation and correction |
|---|---|---|
| “Identify the highest-risk user journeys (login, inventory, cart, checkout, sorting, session handling, etc.). List functional risks, edge cases, UX/UI risks, performance considerations and exploratory questions for each. Mark each suggestion as Test / Discard / Needs validation. Discard anything irrelevant, redundant, incorrect, or based on assumptions the app doesn't support, and say why. Actually test the kept items in the app and record the results.” | Generate and then validate a risk-based exploratory set | Kept and executed the standard-user purchase, price-sort and required-field checks; other scenarios remain `Needs validation`. Excluded payment/shipping/stock checks because those flows were not observed; no behavior was inferred. |
| “Automate one end-to-end flow: login → select product → add to cart → checkout → verify successful completion. Page Object Model with locators separated from actions. Stable selectors (prefer data-test), meaningful assertions (cart contents, price/total, confirmation message), test isolation. No hard-coded waits (use auto-waiting/explicit conditions).” | Request an implemented end-to-end test with POM, stable selectors, and meaningful assertions | Produced the test in `tests/checkout.spec.ts`, with selectors/actions split into separate files. First run exposed Playwright's default `data-testid` mismatch; live DOM inspection verified `data-test`, so configuration was corrected. Type-check passed and the final live Chromium run passed; earlier `ERR_ABORTED` navigation failures are recorded above. |
| “do not hallucinate” | Reinforce evidence-only reporting | Changed the reporting standard to explicitly distinguish observed results from untested questions. No defect is claimed without reproduced evidence. |
| “Run it in stages (A → B → C → D), not in one shot. The log needs real prompts, and 25% of the score is on prompting quality. Keep the actual prompts you send. The log should show you correcting or rejecting AI output, which is worth another 10%.” | Require staged work and genuine prompt/use records | Work was staged: browser exploration before code and report. A selector casing assumption was corrected against the live DOM. The prompt log quotes actual user-provided wording; no synthetic prompts are presented as sent prompts. |

## F — Final submission readiness

| Deliverable | Status |
|---|---|
| Up to five execution-verified defects | Ready: none confirmed in tested scope; untested scope is listed above. |
| AI prompt/usage log | Ready: actual prompts quoted; corrections and limits recorded. |
| Automated test and setup instructions | Authored; TypeScript type-check passed and the live Chromium test passed; initial navigation was intermittent and is noted above. |
| AI-driven QA strategy (one page) | Included above as a concise strategy section. |
| Tools used | Copilot assistant; Chromium via Playwright browser interaction and Playwright Test; TypeScript compiler; shell/file tools. Playwright retained local failure screenshots/video/trace; no successful-flow recording was saved. Generated artifacts are ignored by Git. |
| Public GitHub repository | Not published. The target repository [Demosauce_assesment](https://github.com/bloodlineak/Demosauce_assesment) was confirmed public and empty; no files were pushed. |

## Appendix — Local setup and publishing status

### Run the automated test

From the project directory, on a machine with Node.js 20+ and npm installed:

```sh
npm install
npx playwright install chromium
npm test
npx tsc --noEmit
```

The Playwright configuration enables one retry; a test that passes only on retry is reported as flaky. See `README.md` for the available npm scripts.

### Publish to the requested GitHub repository

Target: https://github.com/bloodlineak/Demosauce_assesment  
Status: not published. The repository was confirmed publicly accessible and empty. The current environment's Git client cannot run because macOS Command Line Tools are not installed; `xcode-select --install` reported that installation was requested, but a follow-up check still found no active developer directory. After installing Command Line Tools, authenticate with GitHub and run:

```sh
cd /Users/shrinithkulal/Downloads/saucedemo-qa
git init -b main
git add .
git commit -m "Add SauceDemo QA assessment"
git remote add origin https://github.com/bloodlineak/Demosauce_assesment.git
git push -u origin main
```

Do not paste access tokens or passwords into chat. The target repository does not currently contain these project files.
