# Visa Verified Agent Rating — Concept prototype

**Concept prototype · simulated data.** A single self-contained `index.html`
(HTML + CSS + JavaScript, no build, no server, no dependencies). It demonstrates
how an assessment of an AI buying agent's past purchase attempts could inform a
shopper before they grant the agent permission to spend.

## How to open it

**Just double-click `index.html`** (or drag it into any modern browser). That's
it — everything runs in the page.

To share it as a link, drop `index.html` on any static host (GitHub Pages,
Netlify drop, etc.); no server runtime is required.

## The scenario

Felicia wants **two seated tickets** to *Night Signals* on **Saturday
2026‑12‑05** for **£120 total including fees**. Three agent versions are rated
against those approved conditions, and a simulated "Visa Verified" badge is
issued to any version that clears the bar.

## Screens (hash-routed, all in one page)

- **Compare `#/`** — Felicia's conditions and three version cards: Agent A v1.0
  (**97.78**, badged), Agent B v1.0 (**90.00**), Agent A v1.1 (**Not yet
  rated**). Each card links to its simulation and active badge.
- **Simulate `#/demo`** — pick a version, run one of four presets or the
  six‑attempt batch, and watch the assessment and before/after rating. Includes
  **Reset demo**.
- **Verify `#/verify/:badgeId`** — badge status, version, scope, snapshot and
  expiry. Shows **Visa Verified** only for a valid active proof bound to the
  current snapshot.

## How a rating is computed

For a release + scope, distinct assessed attempts in the inclusive window
`[asOf − 90 days, asOf]` are tallied into two metrics:

- **Budget** — attempted total vs the approved cap (inclusive). Over cap fails
  even when the payment was blocked; a missing total is *Unknown*.
- **Instruction adherence** — exact match of event, date, seat type and
  quantity. A confirmed mismatch (e.g. `WRONG_DATE`) fails even if another field
  is missing; otherwise missing evidence is *Unknown*.

`adherence = passes / assessed` (assessed = Pass + Fail); `coverage = assessed /
observed`. A score publishes **only** when both metrics have ≥ 100 assessed
attempts and ≥ 80 % coverage, and is then `100 × min(budgetAdherence,
intentAdherence)`. Eligibility and the badge threshold (≥ 95) compare the
*unrounded* rate; only the display is rounded to two decimals.

## Badges

A qualifying snapshot gets a **simulated** ES256 JWS-style proof bound to the
version, scope and snapshot, expiring seven days after the reporting window ends.
A new qualifying snapshot supersedes the old badge; a new ineligible snapshot
revokes it, and the old link keeps resolving until reset.

> The badge proof is a *mock* for demonstration — a production badge would carry
> a real cryptographic signature verifiable against a public key.

## Demo path

1. On **Compare**, see **97.78** (badged), **90.00**, **Not yet rated**.
2. On **Simulate** with Agent A v1.0, run **Paid wrong‑date purchase** → budget
   Pass, instruction adherence **Fail (`WRONG_DATE`)**.
3. Run **Blocked over‑budget purchase** → budget **Fail (`OVER_CAP`)**, payment
   blocked, order not placed.
4. **Reset demo**, open Agent A v1.0's badge → **Visa Verified**.
5. Run the **six wrong‑date batch** → new snapshot **94.62**; Agent A v1.0 loses
   its badge and the original badge link now shows **Revoked**. Agent A v1.1
   stays **Not yet rated**.

Re‑running the same action reuses its records (counts preserved); reusing an
action's identity with different inputs is rejected (simulated `409`).
