# Live Validation and Dogfooding

Sibling concern to `subagent-lifecycle.md`'s "verify, don't just relay" — that rule is about trusting a sub-agent's own report; this one is about what "done" means for a feature at all, regardless of who built it.

## The gap unit tests and direct API calls both miss

A unit test validates a function against the inputs you thought to give it. A direct API call (curl, a script) validates an endpoint against the payload you thought to send it. Neither exercises the actual **contract** between a real client and a real backend, or the actual **copy** a user would read on a real screen — and both are exactly where integration bugs hide, because they only exist at the seam between two pieces of code that were each individually reasonable.

The reliable way to close that gap: drive the real client, in a real (headless) browser or app runtime, against the real backend, as an actual user would — click the actual buttons, read the actual rendered text, submit the actual form. If the project already has any browser-automation tooling for another purpose (screenshot generation, visual regression, marketing content capture), it's usually just as effective repurposed as a validation harness — "does the real screen show the real thing" is the same question either way.

## When to do this

Not every change needs a live click-through — a config tweak or an isolated bug fix with a good unit test doesn't. Reach for full-stack simulated-user validation when:

- The change spans the client/server seam (a new endpoint plus the screen that calls it, a changed response shape plus the component that renders it).
- The feature depends on a real external integration (a third-party API, a payment provider) where fixture data can't represent every real-world quirk (naming conventions, rate-limit shapes, partial failures).
- The feature has multiple UI states driven by combinations of backend results (e.g. two independent legs each found/not-found) — these combinatorial states are exactly what's easy to individually unit-test and collectively get wrong when summarized into one line of copy for a human to read.

## How, concretely

- Bring up the real stack locally — real database, real backend, real dev server, not a mock.
- Use a disposable test account (register fresh, elevate role/rank directly in the local DB for gated features — never touch a real user's row) so the real auth/permission path is exercised, not bypassed.
- Where the feature needs real external state (a position, a payment, an order), create the smallest real version of it via whatever credentialed test-scripts convention the project already has, rather than mocking the external call — a mock can't be wrong about a real API's actual field names or conventions, because it never asks the real API anything.
- Drive the browser (or app) with automation tooling reused from wherever the project already has it — inject a session token into local storage to skip a manual login if that's the fastest path to a real authenticated session, then click through the actual flow and read the actual rendered copy, not just the network response.
- Clean up afterward: close any real position/order opened for the test, delete the disposable account, stop any services started only for this.

## What this catches that the alternatives don't

Two failure shapes worth watching for specifically, because they pass unit tests and manual API checks by construction:

- **A contract bug that only surfaces with real, mismatched data.** Test fixtures often use the same convention on both sides of an integration by construction (e.g. the same identifier format for a value that, in the real world, can differ per source) — the bug only becomes visible once a real case with the mismatch is driven end-to-end, because the fixture never modeled the mismatch to begin with.
- **A messaging/summary bug where each underlying piece is individually correct.** Multiple independently-correct results (e.g. per-field or per-leg outcomes) can get collapsed into one summary line that's misleading even though nothing in the API response itself is wrong — a state that's very unlikely to be caught by asserting on the response shape alone, since the response was correct; the bug is purely in how it reads as rendered copy.

The validation pass doesn't need to be a separate QA step bolted on afterward — running it as part of finishing the feature (or producing a walkthrough/demo of it) is often what reveals there's anything to fix in the first place.
