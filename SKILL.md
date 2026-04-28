---
name: wcag-2.2-aa
description: Use when writing, modifying, or reviewing UI code — JSX/HTML/Vue templates, interactive components (forms, buttons, modals, menus, accordions), images, videos, color/contrast decisions, focus management, keyboard handlers, ARIA attributes, or interactive copy (error messages, link text, button labels) — before declaring UI work done. Orchestrates automated tools (eslint-plugin-jsx-a11y, axe-core, pa11y) plus semantic heuristics tools can't detect. Targets WCAG 2.2 Level AA (which by definition includes all Level A criteria). Load patterns.md for concrete DO/DON'T examples per category.
---

# WCAG 2.2 AA Accessibility Review

Real accessibility requires automated tools AND human judgment. This skill orchestrates the tools, flags what's missing, and applies semantic checks tools can't. Target: **WCAG 2.2 Level AA** (meeting AA requires meeting all Level A criteria plus all Level AA criteria).

> **Not a compliance attestation.** This skill is a structured checklist to help catch common accessibility issues earlier. Using it does not guarantee conformance with WCAG, Section 508, EN 301 549, ADA, AODA, or any other legal or regulatory standard. For regulated contexts (healthcare, finance, government contracts, VPAT/ACR authoring, settlements, or statutory compliance), engage a qualified accessibility auditor and test with real users of assistive technology. Level AAA is aspirational for critical content only and is out of scope for this skill.

## Nudge-firmly protocol

UI work is NOT complete until:

1. **Automated scan passes** (eslint-plugin-jsx-a11y on modified files; axe/pa11y if available)
2. **Semantic checks pass** (apply heuristics below to changed code)
3. **Known-but-unfixed issues are explicitly acknowledged by the user**

When you find likely AA issues: cite the specific success criterion (e.g., "SC 1.1.1"), explain what's wrong, propose a fix. Do not declare the task complete. If the user overrides ("ship it, we'll fix in a follow-up"), proceed — and leave a `// TODO(a11y SC X.Y.Z): <brief>` comment at the site so it's discoverable.

## Step 1 — Toolchain check (once per project)

```bash
find . -name package.json -not -path '*/node_modules/*' -print0 \
  | xargs -0 grep -l -E '"(eslint-plugin-jsx-a11y|axe-core|@axe-core|pa11y|jest-axe)"' \
  || echo "No a11y tooling detected."
```

If missing, recommend (don't auto-install):

- **React/Next.js:** `eslint-plugin-jsx-a11y` — included by `eslint-config-next`; verify your `.eslintrc` extends it
- **Test harness:** `@axe-core/playwright`, `@axe-core/react`, or `jest-axe`
- **CLI audit:** `pa11y <url>` — one-shot audit of a running URL

## Step 2 — Automated scan

Run fast-to-slow:

1. `npm run lint` (or `eslint <changed-files>`) — catches missing alt, button-has-type, click-events-have-key-events, invalid ARIA roles, etc.
2. If Playwright/axe wired in: `npx playwright test` — catches runtime issues (contrast, missing landmarks, ARIA state)
3. If site is live: `npx pa11y https://<url>` — one-shot audit

Report each finding with its WCAG SC reference and severity. Remember: axe, pa11y, and linters find a meaningful fraction of issues, not all of them. Research consistently shows automated tools surface roughly 30–40% of WCAG problems; the remainder require manual review and assistive-tech testing.

## Step 3 — Semantic review (the LLM-specific value)

Tools miss these. Apply each heuristic to changed UI code. For concrete code examples in any category below, load [patterns.md](patterns.md).

### Images & media — SC 1.1.1, 1.2.1–1.2.5
Alt text must describe what the image *actually shows* in context. `<img src="duck.jpg" alt="A dog">` passes every linter and fails every screen-reader user. Decorative images: `alt=""` (empty, not missing). Prerecorded video requires captions (SC 1.2.2, A) and audio description or media alternative (SC 1.2.3, A; SC 1.2.5 strengthens this at AA). Live video needs captions (SC 1.2.4, AA). Auto-generated captions are a starting point, not compliance.

### Link text & button labels — SC 2.4.4, 2.5.3
Link text must describe the destination *out of context* — "Click here," "Learn more," "Read more" all fail the screen-reader links list. Icon-only buttons need an accessible name (`aria-label` or visually-hidden text). When an `aria-label` exists on a control with visible text, the `aria-label` MUST contain the visible text (SC 2.5.3 "Label in Name" — voice-control users say what they see).

### Heading hierarchy — SC 1.3.1, 2.4.6
Headings must describe their section, not just "big text." Do not skip levels (h2 → h4 is a structural break). Using a single `<h1>` per page is a strong best practice — not a normative WCAG rule, but it improves screen-reader navigation.

### Forms — SC 1.3.1, 1.3.5, 3.3.1, 3.3.2, 3.3.3, 4.1.2
Every input has a programmatic label. Required fields marked programmatically (not just with a red asterisk). Errors are announced and associated via `aria-describedby`. `autocomplete` set on identity/contact/payment fields (name, email, tel, cc-number, address-*). Error messages should **suggest a fix** (SC 3.3.3, AA), not just flag the problem.

### Keyboard — SC 2.1.1, 2.1.2, 2.4.3, 2.4.7, 2.4.11
Everything interactive reaches via Tab. Focus order matches visual order. Focus visible (never `outline: none` without replacement). No keyboard traps (modals must escape via Esc). **2.2 NEW — SC 2.4.11 "Focus Not Obscured (Minimum)" (AA):** when an element receives focus, it must not be entirely hidden by author-created content (sticky headers, cookie banners, chat widgets). **Watch for the multi-background failure mode (SC 1.4.11):** a single global focus-ring color often passes 3:1 on dark sections but fails on light sections (or vice versa). When the page has multiple distinct background colors — dark hero, light body, brand-color CTA — scope the ring color per section via a CSS custom property, or use a two-color halo (`outline` + `box-shadow`). See `patterns.md` "Focus rings on multi-background pages."

### Color & contrast — SC 1.4.1, 1.4.3, 1.4.11, 1.4.12
Body text 4.5:1 vs background. Large text (≥18pt or ≥14pt bold) 3:1. UI components, focus indicators, and icons conveying state 3:1 (SC 1.4.11). Color alone never conveys info — pair with icon/text (SC 1.4.1). Layout must still function when users override text spacing (SC 1.4.12, AA).

### Resize, reflow, hover — SC 1.4.4, 1.4.10, 1.4.13
Text must scale to 200% without loss of content or function (SC 1.4.4, AA). Content must reflow at 320 CSS px wide without horizontal scrolling (SC 1.4.10, AA). Tooltips and hover/focus-triggered popovers must be dismissable, hoverable, and persistent (SC 1.4.13, AA).

### Interactive components & ARIA — SC 4.1.2, 4.1.3
Prefer native HTML (`<button>` over `<div role="button">`). If using ARIA, follow the WAI-ARIA Authoring Practices (APG) patterns — note these are non-normative design guides, not WCAG requirements. Modals need focus trap, return focus on close, `role="dialog"` + `aria-modal="true"`. Never put an ARIA role on an element that already has the equivalent native role (`<button role="button">` is an anti-pattern). Dynamic status updates go in a live region or element with `role="status"`/`role="alert"` (SC 4.1.3, AA).

### Motion & input — SC 2.3.1, 2.3.3, 2.5.1, 2.5.7, 2.5.8
No content flashes more than three times per second (SC 2.3.1). Respect `prefers-reduced-motion` for animations, parallax, and autoplay (SC 2.3.3 at AAA, but widely expected). **Auditor's note: animations on `::before` / `::after` pseudo-elements** are a common blind spot — JS `document.querySelectorAll('*')` doesn't reach them, so a runtime sweep can miss them. Grep the stylesheet for `@keyframes` and `animation:` declarations and trace where each is applied; check pseudo-element targets explicitly. **2.2 NEW — SC 2.5.7 "Dragging Movements" (AA):** drag interactions must have a single-pointer non-drag alternative. **2.2 NEW — SC 2.5.8 "Target Size (Minimum)" (AA):** pointer targets are at least 24×24 CSS pixels, with defined exceptions (spacing, inline in text, equivalent elsewhere, user-agent defaults, essential).

### Page-level — SC 2.4.2, 3.1.1, 3.2.3, 3.2.4, 3.2.6
Unique descriptive `<title>` per page. Root `<html lang="...">` set correctly (SC 3.1.1). Navigation and same-function component labels consistent across pages (SC 3.2.3, 3.2.4). **2.2 NEW — SC 3.2.6 "Consistent Help" (A):** if a help mechanism (contact link, chat widget, self-serve help link) appears on multiple pages, it must appear in the same relative order on each page.

### Auth & forms — 2.2 NEW
- **SC 3.3.7 "Redundant Entry" (A):** multi-step forms auto-populate or let the user select previously entered info — don't make them retype it.
- **SC 3.3.8 "Accessible Authentication (Minimum)" (AA):** login cannot require a cognitive function test (remembering, transcribing, puzzle-solving) without an accessible alternative. Don't block paste on password fields (breaks password managers). Don't disable autofill on login forms. Conventional CAPTCHAs need an accessible alternative (e.g., risk-based, or a non-cognitive challenge).
- SC 3.3.9 "Accessible Authentication (Enhanced)" is Level AAA and is out of scope for this skill.

## Report format

When you find likely issues:

```
[SC 1.1.1 — Non-text Content] (A)
File: src/components/Gallery.tsx:42
<img> has alt="" but shows a product photo (meaningful content).
Fix: alt="<describe what the image depicts in context>"
```

Group: Level A issues > Level AA issues > advisory notes. A and AA block completion unless the user explicitly defers. AAA notes are informational.

**Also include a "Verified passing" section** listing the SCs that were checked and confirmed compliant (e.g., `1.4.10 Reflow ✅ no horizontal scroll at 320px`, `2.5.8 Target Size ✅ 0 interactive elements below 24×24`). Listing only failures leaves the reviewer guessing at coverage; an explicit pass list shows the audit's scope and builds trust that nothing was skipped.

## Scope limits

This skill covers code-level review only. It does **not** evaluate:

- Live assistive-tech behavior (NVDA/JAWS/VoiceOver, TalkBack, VoiceOver iOS — manual testing required, ideally with real users)
- Cognitive load and plain-language sufficiency in real use (needs user testing with the target disability community)
- Third-party embeds not under your control (GHL widgets, YouTube, Stripe Checkout, reCAPTCHA) — flag them; fixes are upstream or must be replaced
- Non-web surfaces (native mobile, PDF/Office docs, email templates, kiosks)
- Legal or regulatory conformance. WCAG conformance is a prerequisite for many legal standards (Section 508, EN 301 549, ADA settlements, AODA) but meeting this checklist does not by itself demonstrate compliance — consult qualified counsel and auditors for that.

## Spec reference

- W3C WCAG 2.2 Recommendation: https://www.w3.org/TR/WCAG22/
- WAI-ARIA Authoring Practices (APG, non-normative): https://www.w3.org/WAI/ARIA/apg/

When explaining a finding, cite the SC fragment (e.g., `/TR/WCAG22/#non-text-content`) so the user can read the normative text.
