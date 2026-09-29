---
name: wcag-2.2-aa
description: Use when writing, modifying, or reviewing UI code — JSX/HTML/Vue templates, interactive components (forms, buttons, modals, menus, accordions), images, videos, color/contrast decisions, focus management, keyboard handlers, ARIA attributes, or interactive copy (error messages, link text, button labels) — before declaring UI work done. Also use when asked to run or interpret axe-core, eslint-plugin-jsx-a11y, or pa11y results. Targets WCAG 2.2 Level AA (which by definition includes all Level A criteria).
---

# WCAG 2.2 AA Accessibility Review

Real accessibility requires automated tools AND human judgment. This skill runs the tools, states exactly what they did and did not cover, and applies the semantic checks tools cannot. Target: **WCAG 2.2 Level AA** (all Level A plus all Level AA criteria). **SC 4.1.1 Parsing was removed in WCAG 2.2 — do not cite it.**

> **Not a compliance attestation.** This skill is a structured checklist to help catch common accessibility issues earlier. Using it does not guarantee conformance with WCAG, Section 508, EN 301 549, ADA, AODA, or any other legal or regulatory standard. For regulated contexts (healthcare, finance, government contracts, VPAT/ACR authoring, settlements, or statutory compliance), engage a qualified accessibility auditor and test with real users of assistive technology. Level AAA is out of scope for this skill.

## Nudge-firmly protocol

UI work is NOT complete until:

1. **Automated scan was actually run AND passed** — both layers:
   - `eslint-plugin-jsx-a11y` on modified files (static markup issues at lint time)
   - **axe-core on rendered pages (REQUIRED)** — computed contrast, ARIA state, focus visibility on the real DOM. If you skipped it, the audit is INCOMPLETE: say so explicitly and either (a) wire it up now using Step 2, or (b) document why you couldn't.
2. **Semantic checks pass** (Step 3 — the value tools can't replace).
3. **Known-but-unfixed issues are explicitly acknowledged by the user.**

> **What axe actually covers.** axe-core tests roughly 16 of the 50 Level A/AA criteria, but those are the most frequent defects, so it finds about **57% of issues by volume** (Deque, 13k+ pages). Everything else is manual — including every criterion new in 2.2 except 2.5.8: as of axe-core 4.13 the `wcag22aa` tag contains only the `target-size` rule. Never let a report imply that 2.4.11, 2.5.7, 3.2.6, 3.3.7, or 3.3.8 were machine-checked.

When you find likely A/AA issues: cite the success criterion (e.g., "SC 1.1.1"), explain what's wrong, propose a fix. Do not declare the task complete. If the user overrides ("ship it, fix in a follow-up"), proceed and leave `// TODO(a11y SC X.Y.Z): <brief>` at the site.

### Scope gate — which steps run

| The change touches | Lint | Runtime axe | Semantic review |
|---|---|---|---|
| Markup, CSS, or interaction (components, styles, handlers) | always | yes | full Step 3 |
| Copy only (text, headings, link labels, error strings) | always | at PR review | link text, headings, error wording only |
| Any UI change at PR-review time | always | yes | full Step 3 |

## Step 1 — Toolchain check (once per project)

```bash
grep -E '"(eslint-plugin-jsx-a11y|axe-core|@axe-core/[a-z]+|pa11y|jest-axe)"' package.json
ls scripts/qa/a11y.* tests/a11y* playwright.config.* 2>/dev/null
```

In a Next.js project `axe-core` and `eslint-plugin-jsx-a11y` are usually only **transitive** dependencies (`eslint-config-next → eslint-plugin-jsx-a11y → axe-core`). That works under npm's flat layout but breaks under pnpm and can vanish on an upgrade — if a scan script relies on them, add them as explicit devDependencies. `eslint-plugin-jsx-a11y` supports ESLint ≤ 9; on ESLint 10 `eslint-config-next` cannot install and a11y linting silently disappears. Stay on ESLint 9 until you deliberately migrate, or switch to `eslint-plugin-jsx-a11y-x`.

If nothing is present, recommend (don't auto-install): `@axe-core/playwright` + `@playwright/test` for runtime scans, `eslint-plugin-jsx-a11y` for lint, `pa11y` for one-shot CLI audits.

## Step 2 — Automated scan (REQUIRED)

Run fast-to-slow, before semantic review.

### 2a. Lint

```bash
npm run lint            # or scoped: npx eslint <changed-files>
```

`eslint-config-next` enables only six jsx-a11y rules, at `warn`. For the full recommended set at `error`, spread `jsxA11y.flatConfigs.recommended.rules` into the flat config.

### 2b. Runtime axe scan

Three paths, in priority order:

1. **The project has a scan script** (`scripts/qa/a11y.mjs`, `npm run test:a11y`, or similar). Run it against the preview or dev URL.
2. **The project uses `@axe-core/playwright` in its test suite.** Run `npx playwright test`.
3. **Nothing wired up — wire it in-line.** Don't punt:

   ```bash
   npm i -D @axe-core/playwright @playwright/test && npx playwright install chromium
   ```

   ```js
   // ad-hoc-axe-scan.mjs
   import { chromium } from "@playwright/test";
   import { AxeBuilder } from "@axe-core/playwright";

   const url = process.argv[2] || "http://localhost:3000";
   const browser = await chromium.launch();
   let blocking = 0;
   // Scan desktop AND 375px: axe's target-size results are viewport-dependent.
   for (const viewport of [{ width: 1440, height: 900 }, { width: 375, height: 812 }]) {
     // bypassCSP: a strict Content-Security-Policy otherwise blocks axe's injected script.
     const context = await browser.newContext({ viewport, bypassCSP: true });
     const page = await context.newPage();
     await page.goto(url, { waitUntil: "load" }); // not "networkidle" — Playwright marks it DISCOURAGED
     await page.waitForSelector("main");
     const { violations } = await new AxeBuilder({ page })
       .withTags(["wcag2a", "wcag2aa", "wcag21a", "wcag21aa", "wcag22aa"])
       .analyze();
     for (const v of violations) {
       console.log(`[${viewport.width}px] [${v.impact}] ${v.id} — ${v.help}\n  ${v.helpUrl}\n  ${v.nodes.length} node(s)`);
       if (["critical", "serious"].includes(v.impact)) blocking++;
     }
     await context.close();
   }
   await browser.close();
   process.exit(blocking ? 1 : 0);
   ```

   ```bash
   node ad-hoc-axe-scan.mjs http://localhost:3000   # exit 1 on any critical/serious violation
   ```

   Fallback when you cannot add dependencies: read `node_modules/axe-core/axe.min.js`, inject it with `page.addScriptTag({ content })`, then `page.evaluate(() => axe.run(document, { runOnly: { type: "tag", values: [...] } }))`. Same `bypassCSP` and wait strategy apply.

Also scan the **primary interactive states** (modal open, menu open, error state shown), not just the landing state.

**axe does not scan cross-origin iframes.** A page whose only form is an embedded widget (Go High Level calendar or chat, Stripe Checkout, YouTube) scans "clean" while the form is untested. Require `<iframe title="…">`, then do a manual keyboard pass inside the embed and note it in the report.

### 2c. Optional CLI sanity check

```bash
npx pa11y <url>
```

Quicker than wiring Playwright for a one-off look; axe has wider rule coverage.

### What to do with results

Report each finding with its SC and severity, grouped by route. Fix or triage axe findings **before** semantic review. If you genuinely cannot run axe (offline, site can't be served, no real DOM), say so in the report header and mark the audit **INCOMPLETE — automated scan skipped**.

## Step 3 — Semantic review (the LLM-specific value)

Apply each heuristic to changed UI code. For field-tested code patterns and Next.js/Tailwind/Radix/embed specifics, load [patterns.md](patterns.md).

### Images & media — SC 1.1.1, 1.2.1–1.2.5, 1.4.5
Alt text describes what the image *actually shows in context*; `<img src="duck.jpg" alt="A dog">` passes every linter. Decorative images: `alt=""` (empty, not missing). **Images of text** — hero graphics with slogans, rendered buttons — fail SC 1.4.5 (AA) unless the presentation is essential or customisable; use real text. Prerecorded video needs captions (1.2.2, A) and audio description (1.2.5, AA). Live video needs captions (1.2.4, AA). Auto-generated captions are a draft, not compliance.

### Link text & button labels — SC 2.4.4, 2.5.3
SC 2.4.4 (A) is **Link Purpose (In Context)**: the purpose must be determinable from the link text *or* its programmatically determined context — the same sentence, paragraph, list item, or table cell, or the heading that immediately precedes the link (technique H80). So "Read more" after a card's `<h3>` title passes, and "Learn more" inside a paragraph that names the destination passes; three identical "Learn more" links in sibling cards with no heading or paragraph naming their destinations do not. Link-text-alone is 2.4.9 (AAA): recommend it, don't report it as a failure. Icon-only controls need an accessible name. When `aria-label` and visible text coexist, the label must contain the visible text (2.5.3, A).

### Headings & landmarks — SC 1.3.1, 2.4.1, 2.4.6
Headings describe their section (2.4.6). Skipped levels (h2 → h4) and multiple `<h1>`s are **best practices, not SC failures** (axe tags `heading-order` as best-practice) — report them as advisory. **SC 2.4.1 Bypass Blocks (A):** every page needs a skip link to `<main>` or proper landmarks (`<header>`, `<nav aria-label>`, `<main>`, `<footer>`).

### Forms — SC 1.3.1, 1.3.5, 3.3.1, 3.3.2, 3.3.3, 4.1.2
Every input has a programmatic label. Required fields are marked programmatically (`required`), not just with an asterisk. Errors are associated via `aria-describedby` and announced. `autocomplete` on identity, contact, and payment fields. Error messages **suggest a fix** (3.3.3, AA), not just flag the problem.

### Keyboard & focus — SC 2.1.1, 2.1.2, 2.4.3, 2.4.7, 2.4.11, 1.4.11
Everything interactive is reachable by Tab, in visual order, with a visible focus indicator (never `outline: none` without a replacement); no traps (Esc closes modals). **2.2 NEW — SC 2.4.11 Focus Not Obscured (Minimum) (AA):** the focused element must not be *entirely* hidden by author-created content (sticky header, cookie banner, chat bubble). **Multi-background trap (1.4.11):** a single global focus-ring colour passes 3:1 on dark sections and fails on light ones, or vice versa — scope the ring per section or use a two-tone halo.

### Colour & contrast — SC 1.4.1, 1.4.3, 1.4.11, 1.4.12
Text 4.5:1; large text (≥ 24 px, or ≥ 18.66 px bold) 3:1. UI components, focus indicators, and state icons 3:1 (1.4.11). Colour alone never conveys information (1.4.1). Layout survives user overrides of line-height, letter-, word-, and paragraph-spacing (1.4.12).

### Resize, reflow, hover — SC 1.4.4, 1.4.10, 1.4.13
200% zoom without loss of content or function (1.4.4). Reflow at 320 CSS px without horizontal scrolling (1.4.10). Hover/focus popovers are dismissable, hoverable, and persistent (1.4.13).

### Components & ARIA — SC 4.1.2, 4.1.3
Native HTML first (`<button>`, `<dialog>`, `<details>`). ARIA per the WAI-ARIA Authoring Practices (non-normative). Never add a role an element already has. Dialogs: focus trap, return focus on close, `aria-modal`, an accessible name. Status messages go in a live region that exists in the DOM **before** the message appears (4.1.3, AA).

### Motion & input — SC 2.2.2, 2.3.1, 2.5.7, 2.5.8
**SC 2.2.2 Pause, Stop, Hide (A):** anything that auto-plays, moves, blinks, or scrolls for more than 5 s (carousels, autoplay video, marquees, animated backgrounds) needs a visible pause/stop control. A `prefers-reduced-motion` query does not satisfy it. No content flashes more than three times per second (2.3.1). Grep `@keyframes` and trace where each applies — runtime DOM sweeps miss animations on `::before`/`::after`. **2.5.7 Dragging Movements (AA):** every drag has a single-pointer alternative. **2.5.8 Target Size (Minimum) (AA):** 24×24 CSS px, with five exceptions — spacing, equivalent control, inline in text, user-agent default, essential. Spacing arithmetic is in patterns.md.

### Page-level — SC 2.4.2, 3.1.1, 3.2.3, 3.2.4, 3.2.6
Unique, descriptive `<title>` per page. `<html lang>` set. Navigation and same-function components named consistently across pages (3.2.3, 3.2.4). **2.2 NEW — SC 3.2.6 Consistent Help (A):** help mechanisms (contact link, phone number, chat launcher) appear in the same relative order on every page where they appear.

### Auth & multi-step forms — SC 3.3.7, 3.3.8
**2.2 NEW — SC 3.3.7 Redundant Entry (A):** don't make users retype information they entered earlier in the same process; auto-populate or offer a selection. **2.2 NEW — SC 3.3.8 Accessible Authentication (Minimum) (AA):** authentication cannot require a cognitive function test (memorising, transcribing, solving a puzzle) unless one of four exceptions applies — (1) an alternative method that isn't a cognitive test, (2) a mechanism such as paste or autofill that assists the user, (3) **object recognition** — image-grid "select the traffic lights" CAPTCHAs are permitted at AA, (4) recognising personal content the user provided. Distorted-text, transcription, and puzzle CAPTCHAs need an alternative or mechanism. Never block paste or `autocomplete` on login fields. SC 3.3.9 (AAA) removes the object-recognition exception and is out of scope.

## Report format

Every audit report MUST start with an automated-scan header so the reviewer knows the audit's actual coverage. Then findings, then verified-passing items.

### Report header (required)

```
## Automated scan results
- Tool: axe-core <version> via @axe-core/playwright (or project script, pa11y, or "skipped — see note")
- Routes: /, /services, /about, /faq, /book, /privacy, /terms — at 1440px and 375px
- States: landing, booking modal open
- Violations: 0 blocking (critical/serious), 2 moderate, 0 minor
- Lint: npm run lint clean
- Not machine-checked: cross-origin iframes (GHL calendar); 2.4.11, 2.5.7, 3.2.6, 3.3.7, 3.3.8 (manual below)
- Timestamp: 2026-09-29T14:32:00Z
```

If the scan was skipped, the header MUST say so and why:

```
## Automated scan results
**INCOMPLETE — axe-core was not run.** Reason: [site can't be served, no internet, Storybook-only review, ...]
The semantic review below covers human-grade checks only. Re-run with axe before declaring done.
```

### Findings (per issue)

```
[SC 1.1.1 — Non-text Content] (A)
File: src/components/Gallery.tsx:42
<img> has alt="" but shows a product photo (meaningful content).
Fix: alt="<describe what the image depicts in context>"
```

Group: Level A issues > Level AA issues > advisory notes (best practices, AAA). A and AA block completion unless the user explicitly defers.

### Verified passing (required)

List the SCs that were checked and confirmed, crediting the verifier: `1.4.3 Contrast ✅ axe: 0 violations` for axe-covered criteria; `2.4.4 Link Purpose ✅ manual: all card CTAs disambiguated by their heading context` for judgment criteria. Listing only failures leaves the reviewer guessing at coverage.

## Scope limits

Code-level review only. Not evaluated:

- Live assistive-tech behaviour (NVDA, JAWS, VoiceOver, TalkBack) — manual testing, ideally with real users
- Cognitive load and plain-language sufficiency — needs user testing
- The inside of third-party embeds (GHL widgets, YouTube, Stripe Checkout, reCAPTCHA) — flag them, keyboard-test them, fixes are upstream
- Non-web surfaces (native mobile, PDF/Office, email templates, kiosks)
- Legal or regulatory conformance — consult qualified counsel and auditors

## Spec reference

- W3C WCAG 2.2 Recommendation: https://www.w3.org/TR/WCAG22/
- WAI-ARIA Authoring Practices (APG, non-normative): https://www.w3.org/WAI/ARIA/apg/

When explaining a finding, cite the SC fragment (e.g., `/TR/WCAG22/#non-text-content`) so the user can read the normative text.
