# WCAG 2.2 AA — Field patterns

Traps that models, linters, and axe reliably miss, plus framework specifics. Textbook patterns (labels, tabs, accordions, modal markup, contrast tables) are deliberately omitted — you already reproduce them on demand. For composite widgets, use a library (Radix, react-aria, headless-ui) rather than hand-rolling.

Examples are React/JSX and CSS; the principles port to Vue, Svelte, and plain HTML. Examples are simplified to show one pattern at a time.

---

## Focus rings on multi-background pages — SC 2.4.7, 1.4.11

Focus indicators are "visual information required to identify a state," so their contrast against adjacent colours is governed by **1.4.11 (3:1)**, not only 2.4.7. A single global ring colour is the most common real-site failure: a yellow or cyan accent passes on a dark hero and fails on the light body section.

```css
/* DO — per-section ring colour via a custom property; each hits 3:1 against ITS background */
:root           { --focus-ring: #FFCB05; }   /* yellow on dark chrome */
.section--light { --focus-ring: #0E2747; }   /* navy on bone → 13:1 */
.section--dark  { --focus-ring: #FFCB05; }

:where(a, button, [tabindex]):focus-visible {
  outline: 2px solid var(--focus-ring);
  outline-offset: 3px;
}

/* DO (alternative) — two-tone halo works on any background */
:where(a, button, [tabindex]):focus-visible {
  outline: 2px solid #0F1112;          /* dark ring, survives forced-colors mode */
  outline-offset: 2px;
  box-shadow: 0 0 0 4px #FFFFFF;       /* light ring — DISCARDED in Windows forced-colors mode */
}
```

`box-shadow` is dropped in Windows High Contrast / forced-colors mode; `outline` survives. Keep the ring that must always be present on `outline`.

Verify on a live page: focus the same control inside each section and read `getComputedStyle(el).outlineColor`. Same colour across different backgrounds means at least one is probably failing.

---

## Focus not obscured — SC 2.4.11 (AA, 2.2 NEW)

When an element receives focus it must not be **entirely** hidden by author-created content. Real-world offenders: opaque sticky headers, cookie banners, chat-widget bubbles, "we use cookies" bottom sheets over a form's submit button. Partial obscuring is allowed at AA (2.4.12 AAA forbids any). Content the **user** opened themselves (an expanded chat panel) may obscure focus without failing.

```css
/* DO — reserve space so focus-driven scrolling keeps the element below the sticky nav */
html    { scroll-padding-top: 96px; }   /* sticky-nav height + breathing room */
:target { scroll-margin-top: 96px; }    /* anchor-target scrolling too */
```

CSS alone can't fix a persistent overlay that sits on top of form fields. The real fix is design-level: make the overlay dismissable, non-opaque, or not full-width. Test each control with Tab at realistic scroll positions and at 375 px.

---

## Skip link — SC 2.4.1 (A)

```tsx
// First focusable element in <body>, visually hidden until focused
<a href="#main" className="skip-link">Skip to main content</a>
…
<main id="main" tabIndex={-1}>…</main>   {/* tabIndex=-1 makes the target focusable in Safari */}
```

```css
.skip-link { position: absolute; left: -999px; top: 0; }
.skip-link:focus { left: 8px; top: 8px; z-index: 1000; }
```

Next.js App Router: place it in `app/layout.tsx` before the header so it is on every route.

---

## Live regions must exist before the message — SC 4.1.3, 3.3.1

Rendering `<div role="alert">` only when an error appears fails to announce in several screen readers: the live region did not exist when the page mounted. Keep the container always present and toggle its children.

```tsx
// DON'T
{error && <div role="alert">{error}</div>}

// DO — container pre-mounted; role="alert" already implies aria-live="assertive" + aria-atomic
<div id="email-error" role="alert" className="error-text">
  {error ? "Please enter a valid email address (e.g., name@example.com)" : null}
</div>
<input id="email" type="email" aria-invalid={!!error} aria-describedby="email-error" />
```

Do not add `aria-live` on top of `role="alert"`; some AT combinations double-announce. Use `role="status"` (polite) for non-error updates such as "Saved" toasts.

---

## Animations: pseudo-elements and the pause control — SC 2.2.2, 2.3.1

**Pseudo-element blind spot.** Animations declared on `::before`/`::after` (pulsing status dot, shimmer overlay) are invisible to `document.querySelectorAll('*')` sweeps, so runtime audits miss them.

```bash
grep -nE '@keyframes|animation(-name)?:' src/**/*.css
# for each hit, find the selector — often a ::before / ::after
```

```js
// Playwright: sample pseudo-element computed styles directly
await page.evaluate(() => {
  const s = getComputedStyle(document.querySelector('.status-dot'), '::before');
  return { name: s.animationName, duration: s.animationDuration };
});
// Re-run with browser.newContext({ reducedMotion: 'reduce' }) and expect ~0.01ms
```

**SC 2.2.2 Pause, Stop, Hide (A)** — any auto-playing, moving, or auto-updating content that lasts more than 5 s (carousel, autoplay video, marquee, animated background) needs a visible control to pause, stop, or hide it. `prefers-reduced-motion` respects a preference; it does not provide the control.

```tsx
<button type="button" aria-pressed={paused} onClick={() => setPaused(!paused)}>
  {paused ? "Play animation" : "Pause animation"}
</button>
```

---

## Animated accordions — `hidden` vs animation — SC 4.1.2

`hidden` removes the panel from layout and the accessibility tree, which blocks CSS expand/collapse animations. Modern fix: animate `display` with `transition-behavior: allow-discrete`.

```css
.panel { transition: height 200ms, display 200ms allow-discrete, overflow 200ms; }
.panel[hidden] { height: 0; overflow: hidden; }
```

Older fallback: keep the panel in the DOM, hide it with `inert` + `aria-hidden="true"` while collapsed, animate height, and keep `aria-expanded` on the trigger in sync.

---

## Audio description reality — SC 1.2.5 (AA)

`<track kind="descriptions">` is in the HTML spec but has no meaningful support in shipping browsers. A conformant audio description is a **separate described-video file** the user can choose, or a description track mixed into the main video. Treat a descriptions VTT as a signal, not the mechanism.

---

## Label in Name — SC 2.5.3 (A)

Voice-control users say what they see. When a control has visible text and an `aria-label`, the label must contain the visible text.

```tsx
// DON'T — visible "Search", accessible name "Find products"
<button aria-label="Find products">Search</button>

// DO
<button aria-label="Search for products">Search</button>
```

---

## Target size spacing math — SC 2.5.8 (AA, 2.2 NEW)

Targets are at least **24×24 CSS px** unless one of five exceptions applies:

1. **Spacing** — a 24 px diameter circle centred on the target's bounding box does not intersect another target or another undersized target's circle.
2. **Equivalent** — the same function is available through another target on the page that conforms.
3. **Inline** — the target is within a sentence or block of text.
4. **User agent control** — a default browser control, not restyled by the author.
5. **Essential** — the size is legally required or essential to the information.

**Spacing arithmetic.** Circles are centred, so adjacent undersized targets need their **centres ≥ 24 px apart**. Two 16 px icons need an **8 px gap**. A 16 px icon beside a full-size (≥ 24 px) target needs a **4 px gap** (12 px radius minus 8 px half-icon). "12 px of clearance on every side" is three times too conservative.

```css
/* DO — prefer enlarging the hit area; the glyph can stay 16px */
.icon-btn { min-width: 24px; min-height: 24px; padding: 4px; }

/* DO — spacing exception when the box really can't grow: centres 24px apart */
.icon-btn--tiny { width: 16px; height: 16px; }
.icon-btn--tiny + .icon-btn--tiny { margin-left: 8px; }
```

axe's `target-size` rule is **viewport-dependent** — always scan at 375 px as well as desktop.

---

## Accessible authentication — SC 3.3.8 (AA, 2.2 NEW)

Authentication must not require a **cognitive function test** — memorising a password or code, transcribing characters from one field to another, solving a puzzle or distorted-text CAPTCHA — unless at least one of four exceptions applies:

1. **Alternative** — another authentication method that is not a cognitive test.
2. **Mechanism** — assistance is available, e.g. paste and password-manager autofill work, or a WebAuthn/passkey path exists.
3. **Object recognition** — the test is to recognise objects (image-grid CAPTCHAs are **permitted** at AA).
4. **Personal content** — the test is to identify non-text content the user themselves provided.

Practical rules: never block paste on password fields, never set `autocomplete="off"` on login fields, prefer risk-based or invisible challenges, and offer a magic link or passkey alongside any CAPTCHA that isn't object recognition.

```tsx
// DON'T
<input type="password" onPaste={(e) => e.preventDefault()} autoComplete="off" />

// DO
<input type="password" autoComplete="current-password" />
```

SC 3.3.9 (AAA) removes the object-recognition and personal-content exceptions; it is out of scope for AA and must not be double-counted.

---

## Next.js / React / Tailwind / Radix / embeds

- **`next/image`** requires `alt`; use `alt=""` for decorative images. Never ship slogans or CTAs as image pixels — that fails **1.4.5 Images of Text**. `eslint-config-next` maps `jsx-a11y/alt-text` to the `Image` component.
- **`next/link`** renders the `<a>` itself. Don't nest `<a>` or `<button>` inside it (invalid nesting → 4.1.2 name/role problems).
- **Route announcer** (App Router) reads the new page's `<title>` after client-side navigation. Every route needs `metadata.title`, or navigation announces nothing (2.4.2).
- **Focus management** — `ref.current.focus()` after navigation, dialog return-focus, `useEffect` focus moves — must live in a `'use client'` component.
- **Tailwind:** never bare `outline-none`. Use `focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[var(--focus-ring)]` (or `focus-visible:ring-*`). `sr-only` is the visually-hidden utility; `motion-reduce:` variants for animations (still add the 2.2.2 pause control).
- **Radix:** `Dialog` requires `Dialog.Title` (wrap in `VisuallyHidden` if not shown) or it warns and the dialog has no name. `Tooltip` content is hoverable by default — keep it that way (1.4.13). Use `Toast` with `role="status"` for non-error messages.
- **Native `<dialog>`:** call `showModal()` (not `show()`) to get the focus trap, `aria-modal`, and Esc; return focus to the trigger yourself on close.
- **Embedded widgets** (Go High Level calendar and chat, Stripe, YouTube): every `<iframe>` needs a `title`. axe cannot see inside cross-origin iframes, so keyboard-test the embed manually and check **2.4.11** (does sticky modal chrome cover focused fields?), **2.5.8** at 375 px (calendar day cells), and **1.4.3** (the widget's placeholder text). A chat bubble is a help mechanism: same relative order on every page (**3.2.6**), and it must not fully cover focused controls (**2.4.11**).

---

Full spec: https://www.w3.org/TR/WCAG22/ — published as a W3C Recommendation under the W3C Document License. This file paraphrases the normative text; it does not reproduce it verbatim.
