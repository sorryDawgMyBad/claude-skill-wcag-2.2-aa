# WCAG 2.2 AA — DO/DON'T Patterns

Concrete code patterns per category. Load this when implementing or reviewing a specific area — not when scanning the checklist in SKILL.md.

Examples are primarily React/JSX with parallel HTML. The principles port to Vue, Svelte, plain HTML, etc.

> **Reminder:** Every codebase has specifics that change what "correct" looks like. Use these patterns as a starting point; verify with real assistive-tech testing for anything critical. Examples are simplified to highlight one pattern at a time — they may omit framework-specific concerns, error handling, and production details that matter in your codebase.

---

## Images & media — SC 1.1.1, 1.2.x

**The rule:** Screen-reader users receive `alt` text in place of the image. It must convey the same meaning the image does, in context.

### Meaningful images

```tsx
// DON'T — generic, useless
<img src="/photos/wallet.jpg" alt="image" />
<img src="/photos/wallet.jpg" alt="photo" />
<img src="/photos/wallet.jpg" alt="wallet.jpg" />

// DON'T — describes the image source, not the content
<img src="/photos/duck.jpg" alt="A dog" />  // Wrong animal!

// DO — describes what the image shows, in context
<img src="/photos/wallet.jpg" alt="Red leather bifold wallet, front view" />
```

### Decorative images

When the image adds visual decoration but no information (the surrounding text conveys everything):

```tsx
// DON'T — empty but missing alt attribute (screen readers announce the filename)
<img src="/decor/swirl.svg" />

// DO — explicit empty alt tells the screen reader to skip
<img src="/decor/swirl.svg" alt="" />

// DO — or use a CSS background image for purely decorative visuals
// (no attribute needed; the element has no semantics to suppress)
<div className="hero-decor" />
```

Note: `role="presentation"` on a plain `<div>` is a no-op — a `<div>` has no implicit role, so there is nothing to remove. Some validators flag it as an error.

### Icons

```tsx
// DON'T — icon with no accessible name, button has no label
<button onClick={onClose}>
  <XIcon />
</button>

// DO — icon is decorative, button has an accessible name
<button onClick={onClose} aria-label="Close dialog">
  <XIcon aria-hidden="true" />
</button>

// DO — when the icon is next to visible text, hide the icon from AT
<button onClick={onSubmit}>
  <CheckIcon aria-hidden="true" />
  Submit
</button>
```

### Complex images (charts, diagrams)

A short `alt` can't convey a chart. Provide a long description nearby or via `aria-describedby`.

```tsx
<figure>
  <img
    src="/chart-q4-revenue.png"
    alt="Q4 2025 revenue by product line"
    aria-describedby="chart-desc"
  />
  <figcaption id="chart-desc">
    Widget A: $1.2M (42%). Widget B: $800k (28%). Widget C: $860k (30%).
    Widget A grew 15% YoY; B and C were flat.
  </figcaption>
</figure>
```

### Video & audio — SC 1.2.1, 1.2.2, 1.2.3, 1.2.4, 1.2.5

Per WCAG 2.2, prerecorded video requires **captions** (SC 1.2.2, Level A) and **audio description or full media alternative** (SC 1.2.3, Level A). SC 1.2.5 (Level AA) strengthens 1.2.3 by removing the text-alternative escape hatch — at AA you need actual audio description. Live video needs captions (SC 1.2.4, Level AA).

```tsx
// Prerecorded video: captions + audio description
<video controls>
  <source src="demo.mp4" type="video/mp4" />
  <track kind="captions" src="demo.en.vtt" srclang="en" label="English" default />
</video>
```

**About audio description:** `<track kind="descriptions">` is defined in the HTML spec but has effectively no support in shipping browsers. A WCAG-conformant audio description is almost always produced as either (a) a separate described-video file the user can select, or (b) a description audio track mixed into the main video. Treat a `<track kind="descriptions">` file as a nice-to-have signal, not the mechanism that actually delivers description to users.

Auto-generated YouTube captions don't meet WCAG — they require human review for accuracy.

---

## Link text & button labels — SC 2.4.4, 2.5.3

**The rule:** Link text describes the destination; button labels describe the action. Must make sense when read out of context (a screen reader often pulls a list of links on the page).

```tsx
// DON'T — "click here" tells a screen-reader user nothing
<p>To read our pricing, <a href="/pricing">click here</a>.</p>

// DO — descriptive link text
<p>Read <a href="/pricing">our pricing</a>.</p>

// DON'T — "Read more" as the whole link
<article>
  <h3>Article title</h3>
  <a href="/blog/article-slug">Read more</a>
</article>

// DO — link wraps the heading OR includes sr-only context
<article>
  <h3><a href="/blog/article-slug">Article title</a></h3>
</article>

// OR
<article>
  <h3>Article title</h3>
  <a href="/blog/article-slug">
    Read more<span className="sr-only"> about the article title</span>
  </a>
</article>
```

### Label in Name (SC 2.5.3)

When a component has visible text AND an `aria-label`, the aria-label MUST contain the visible text (voice-control users say what they see).

```tsx
// DON'T — visible text says "Search", aria-label says "Find products"
<button aria-label="Find products">Search</button>

// DO — aria-label contains the visible label
<button aria-label="Search for products">Search</button>
```

### Icon-only controls

```tsx
// DON'T — no accessible name
<button onClick={togglePlay}><PlayIcon /></button>

// DO — accessible name via aria-label
<button onClick={togglePlay} aria-label={isPlaying ? "Pause" : "Play"}>
  {isPlaying ? <PauseIcon aria-hidden="true" /> : <PlayIcon aria-hidden="true" />}
</button>
```

---

## Headings & landmarks — SC 1.3.1, 2.4.1, 2.4.6

### Heading hierarchy

```tsx
// DON'T — skipping levels, multiple h1s, or using <div> for visual heading
<div className="big-text">Our services</div>

<h1>Home</h1>
<h1>Why choose us</h1>  // second h1

<h1>Home</h1>
<h3>Services</h3>  // skipped h2

// DO — descriptive h1, sequential levels, semantic elements
<h1>Company or page name</h1>
  <h2>Section one</h2>
  <h2>Section two</h2>
    <h3>Subsection</h3>
    <h3>Subsection</h3>
  <h2>Section three</h2>
```

Using a single `<h1>` per page is a strong best practice for screen-reader navigation; multiple `<h1>`s (one per sectioning root) do not fail a WCAG success criterion but make the structure harder for AT users to reason about.

### Landmarks

```tsx
// DON'T — nothing but <div>s, no landmarks
<div className="nav">...</div>
<div className="content">...</div>
<div className="footer">...</div>

// DO — semantic landmarks let AT users navigate by region
<header>
  <nav aria-label="Main">...</nav>
</header>
<main>...</main>
<footer>...</footer>

// If you have multiple nav regions, disambiguate with aria-label
<nav aria-label="Main">...</nav>
<nav aria-label="Footer">...</nav>
```

---

## Forms — SC 1.3.1, 1.3.5, 3.3.1, 3.3.2, 3.3.3, 4.1.2

### Labels

```tsx
// DON'T — placeholder is NOT a label (disappears on focus, low contrast)
<input type="email" placeholder="Email address" />

// DON'T — floating text near a field doesn't associate with it
<div>Email address</div>
<input type="email" />

// DO — explicit <label> with htmlFor
<label htmlFor="email">Email address</label>
<input id="email" type="email" />

// DO — wrapping label (works for native inputs only — if the input is a
// custom component that renders a wrapping div, prefer htmlFor + id)
<label>
  Email address
  <input type="email" />
</label>

// DO — when a visible label isn't feasible, aria-label or aria-labelledby
<input
  type="search"
  aria-label="Search"
  placeholder="Search..."
/>
```

### Required fields — SC 3.3.2

```tsx
// DON'T — red asterisk with no programmatic signal
<label>Email *</label>
<input type="email" />

// DO — visible marker AND programmatic required via the native HTML attribute
<label htmlFor="email">
  Email <span aria-hidden="true">*</span>
  <span className="sr-only">(required)</span>
</label>
<input id="email" type="email" required />
```

Use the native `required` attribute, not `aria-required="true"`. They are redundant together, and the first rule of ARIA is to prefer a native attribute when one exists. Some screen-reader + browser combinations double-announce when both are present.

### Autocomplete — SC 1.3.5

Let browsers and password managers autofill. Reduces cognitive load, helps motor-impaired users.

```tsx
// DO — use standardized autocomplete values
<input id="name" autoComplete="name" />
<input id="email" type="email" autoComplete="email" />
<input id="tel" type="tel" autoComplete="tel" />
<input id="cc-number" autoComplete="cc-number" />
<input id="street" autoComplete="street-address" />
<input id="zip" autoComplete="postal-code" />
```

Full list: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#autofill

### Error messages — SC 3.3.1, 3.3.3

```tsx
// DON'T — visual-only error, no association
<input type="email" className="error" />
<div className="error-text">Please enter a valid email</div>

// DO — error text associated via aria-describedby; rendered into a persistent live region
<input
  id="email"
  type="email"
  aria-invalid={hasError}
  aria-describedby="email-error"
/>
<div id="email-error" role="alert" className="error-text">
  {hasError ? "Please enter a valid email address (e.g., name@example.com)" : ""}
</div>
```

A few subtleties worth knowing:

- `role="alert"` already implies `aria-live="assertive"` + `aria-atomic="true"`. Do not also add `aria-live` — specifying both is redundant and double-announces in some assistive-tech combinations.
- Rendering an element with `role="alert"` *only when* an error appears can fail to announce in some screen readers, because the live region did not exist when the page mounted. Prefer an always-present `role="alert"` container whose children toggle.
- Error messages should **suggest a fix** (SC 3.3.3, AA), not just state the problem. "Invalid email" is worse than "Please enter a valid email (e.g., name@example.com)."

---

## Keyboard & focus — SC 2.1.1, 2.1.2, 2.4.3, 2.4.7, 2.4.11

### Focus visible

```css
/* DON'T — removes focus indicator entirely */
:focus { outline: none; }

/* DON'T — only removes for mouse users but keyboard users still lose it */
button:focus { outline: none; }

/* DO — remove the default, provide a custom indicator */
:focus-visible {
  outline: 2px solid var(--focus-color);
  outline-offset: 2px;
}

/* DO — if you want zero visual change (rare, usually wrong), use a native alternative */
/* Actually, don't. Focus must be visible. */
```

### Clickable non-buttons

```tsx
// DON'T — <div> with onClick isn't keyboard-accessible
<div onClick={handleClick} className="button-looking">
  Submit
</div>

// DO — use a real button
<button onClick={handleClick}>Submit</button>

// DO (only if button is truly not possible) — add keyboard + role + tabIndex
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === "Enter" || e.key === " ") {
      e.preventDefault();
      handleClick();
    }
  }}
>
  Submit
</div>
```

### Focus trap in modals

```tsx
// When a modal opens:
// 1. Move focus INTO the modal (usually the first focusable element or a close button)
// 2. Trap Tab/Shift+Tab within the modal
// 3. Esc closes the modal
// 4. Return focus to the triggering element on close

// Don't hand-roll this — use a library (react-aria, radix-ui, headless-ui)
import { Dialog } from "@radix-ui/react-dialog";
// Radix handles focus trap, Esc, return-focus, aria-modal, etc.
```

### SC 2.4.11 — Focus Not Obscured (Minimum) (AA, 2.2 NEW)

When an element receives focus, it must not be **entirely** hidden by author-created content (sticky headers, cookie banners, chat widgets, live-chat nudges). The "Minimum" flavor allows partial obscuring; the AAA flavor (2.4.12) is stricter.

```css
/* DON'T — fully-opaque sticky nav that covers focused fields sitting beneath it */
.sticky-nav { position: fixed; top: 0; height: 80px; background: white; }

/* DO — reserve space in the scroll container so focus-driven scrolling keeps
   the focused element in the visible area, below the sticky nav */
html { scroll-padding-top: 96px; /* sticky-nav height + breathing room */ }

/* Also helpful for anchor-target scrolling */
:target { scroll-margin-top: 96px; }
```

CSS alone can't always satisfy 2.4.11. If your layout has overlays that fully cover focused content (e.g., a sticky nav that always sits on top of form fields beneath it without reserving layout space, or a modal-style cookie banner that persists during form use), the real fix is design-level — make the overlay dismissable, non-opaque, or not full-width. Test each interactive control with Tab while the page is scrolled to realistic positions.

---

## Color & contrast — SC 1.4.1, 1.4.3, 1.4.11

### Contrast ratios (AA)

- Body text (< 18pt or < 14pt bold): **4.5:1** vs background
- Large text (≥ 18pt or ≥ 14pt bold): **3:1**
- UI components (form borders, icon buttons, custom checkboxes): **3:1**
- Graphical objects (chart elements, icons conveying state): **3:1**

Check with browser devtools (Chrome's contrast checker in the color picker) or https://webaim.org/resources/contrastchecker/.

```css
/* Borderline cases that fail */
color: #767676; /* on white — exactly 4.54:1, barely passes */
color: #999;    /* on white — 2.85:1, FAILS body text */
color: #aaa;    /* on white — 2.32:1, FAILS */

/* Safe */
color: #595959; /* on white — 7:1 */
color: #000;    /* on white — 21:1 */
```

### Color alone — SC 1.4.1

```tsx
// DON'T — status conveyed only by color
<span style={{ color: status === "error" ? "red" : "green" }}>
  {status === "error" ? "Failed" : "Passed"}
</span>

// DO — icon + color + text
<span style={{ color: colorForStatus(status) }}>
  {status === "error" ? <XIcon aria-hidden="true" /> : <CheckIcon aria-hidden="true" />}
  {status === "error" ? "Failed" : "Passed"}
</span>

// DON'T — links distinguished only by color (red among black text)
<p>Some text with a <span style={{ color: "red" }}>link</span> inside.</p>

// DO — links also have underline or non-color indicator
<p>Some text with a <a href="/link" style={{ textDecoration: "underline" }}>link</a> inside.</p>
```

---

## ARIA components — SC 4.1.2, 4.1.3

### Prefer native HTML

```tsx
// DON'T — div with role="button" is brittle, needs manual keyboard handling
<div role="button" tabIndex={0} onClick={...} onKeyDown={...}>Submit</div>

// DO — <button> gives you keyboard, focus, and semantics for free
<button onClick={...}>Submit</button>
```

### Accordion

```tsx
// DO — expanded state announced, content is controlled
<h3>
  <button
    aria-expanded={isOpen}
    aria-controls="panel-1"
    onClick={toggle}
  >
    Section title
  </button>
</h3>
<div id="panel-1" hidden={!isOpen}>
  Content
</div>
```

Note: `hidden` fully removes the panel from both the accessibility tree and layout, which makes it incompatible with CSS expand/collapse animations. If you need animation, hide the panel with `inert` + `aria-hidden="true"` and animate its height; keep keyboard and screen-reader semantics in sync with `aria-expanded`.

### Tabs

```tsx
// DO — follow WAI-ARIA tabs pattern exactly (or use a library)
<div role="tablist" aria-label="Sections">
  <button
    role="tab"
    aria-selected={activeTab === 0}
    aria-controls="panel-0"
    id="tab-0"
    tabIndex={activeTab === 0 ? 0 : -1}
  >
    Overview
  </button>
  {/* ...more tabs */}
</div>
<div
  role="tabpanel"
  id="panel-0"
  aria-labelledby="tab-0"
  hidden={activeTab !== 0}
>
  {/* panel content */}
</div>
```

Tabs require arrow-key navigation (Left/Right/Home/End). Don't implement from scratch — use react-aria, radix-ui, or headless-ui.

### Modal / Dialog

```tsx
// DO — aria-modal, focus trap, return focus on close
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="dialog-title"
  aria-describedby="dialog-desc"
>
  <h2 id="dialog-title">Confirm deletion</h2>
  <p id="dialog-desc">This action cannot be undone.</p>
  <button onClick={onCancel}>Cancel</button>
  <button onClick={onConfirm}>Delete</button>
</div>
```

Again — use a library (Radix, headless-ui). Hand-rolled modals are a common source of violations.

### Live regions

For dynamic content changes that screen-reader users need to know about (form submission results, toast notifications):

```tsx
// polite — announced when user is idle, doesn't interrupt
<div aria-live="polite" aria-atomic="true">
  {status && <p>{status}</p>}
</div>

// assertive — interrupts current speech; use sparingly for errors
// role="alert" already implies aria-live="assertive" + aria-atomic="true"
// — do not specify both.
<div role="alert">
  {error && <p>{error}</p>}
</div>
```

Keep live-region containers present in the DOM from page load; screen readers sometimes miss announcements from regions that are inserted at the same time as their content.

---

## Motion & input — SC 2.3.1, 2.3.3, 2.5.7, 2.5.8

### Reduced motion — SC 2.3.3

```css
/* DO — respect user's motion preference */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* DO — also respect it for specific animations */
.parallax {
  transform: translateY(var(--scroll-offset));
}
@media (prefers-reduced-motion: reduce) {
  .parallax { transform: none; }
}
```

### No flashing — SC 2.3.1

Anything that flashes more than 3 times per second can trigger photosensitive seizures. Avoid rapid strobe effects entirely. Slow pulses and subtle ambient animations are fine.

### SC 2.5.7 — Dragging alternatives (2.2 NEW)

```tsx
// DON'T — drag is the ONLY way to reorder
<DraggableList items={items} onReorder={setItems} />

// DO — drag AND buttons/menu
<DraggableList items={items} onReorder={setItems}>
  {items.map((item, i) => (
    <Item key={item.id}>
      {item.name}
      <button onClick={() => moveUp(i)} aria-label={`Move ${item.name} up`}>↑</button>
      <button onClick={() => moveDown(i)} aria-label={`Move ${item.name} down`}>↓</button>
    </Item>
  ))}
</DraggableList>
```

### SC 2.5.8 — Target size minimum (2.2 NEW)

Interactive targets must be at least **24×24 CSS pixels** (with exceptions for inline text links and controls in native user-agent UI).

```css
/* DON'T — tiny close button */
.close-btn {
  width: 16px;
  height: 16px;
}

/* DO — enlarge the target via padding, or use min-width/min-height */
.close-btn {
  min-width: 24px;
  min-height: 24px;
  padding: 8px;  /* real tap target is 32x32; icon inside can be 16x16 */
}

/* DO — when an element can't be 24x24, use the spacing exception: a 24 CSS-pixel
   diameter circle centered on the target must not intersect any other target.
   That means at least 12px of clearance on every side between adjacent targets. */
.inline-icon-button {
  width: 16px;
  height: 16px;
  margin: 12px;  /* adjacent targets are then ≥24px apart */
}
```

---

## Auth & forms — 2.2 NEW

### SC 3.3.7 — Redundant Entry (Level A)

Multi-step flows within the same process must not require re-entering information the user already provided in that session, except where re-entry is essential (security confirmation), the information has changed, or the information was never provided by the user.

```tsx
// DON'T — user enters address on step 1, re-enters it on step 2
// because you built two separate forms

// DO — persist form state across steps (React state, form library like
// react-hook-form, URL params, or localStorage)
const [formData, setFormData] = useState<FormData>({});

<Step1 data={formData} onNext={(data) => setFormData({...formData, ...data})} />
<Step2
  data={formData}
  // Offer pre-filled values or an explicit "use shipping address" selector
  onNext={...}
/>
```

### SC 3.3.8 — Accessible Authentication (Minimum) (Level AA)

Login cannot require solving a cognitive function test (memorizing, transcribing a code from a separate field, solving a puzzle, identifying objects in an image) unless an accessible alternative is provided, or an assistive mechanism (paste, autofill) is permitted, or the test is object-recognition/identification of non-text content the user provided.

Practical implications:

- Do not block paste on password fields (breaks password managers).
- Do not disable autofill on login forms.
- If you use CAPTCHAs, provide an accessible alternative (e.g., risk-based challenge, hardware token, email magic-link).

```tsx
// DON'T
<input type="password" onPaste={(e) => e.preventDefault()} autoComplete="off" />

// DO
<input
  type="password"
  autoComplete="current-password"
/>
```

### SC 3.3.9 — Accessible Authentication (Enhanced) — AAA, out of scope

Stricter version of 3.3.8 with fewer exceptions. Not required for AA conformance; noted here so you are not tempted to double-count it.

### SC 3.2.6 — Consistent Help (Level A, 2.2 NEW)

If a help mechanism is available on multiple pages (a contact link, a chat widget launcher, a self-serve help link, a phone number, a contact form), it must appear in the **same relative order** on every page where it appears. Same-order — not same-position; you can use different layouts, but the help link must sit in the same place in the sequence of content, header, or footer where users encountered it before.

```tsx
// DO — help entry lives at the same position in the header on every page
<header>
  <Logo />
  <PrimaryNav />
  <HelpLink /> {/* same relative position site-wide */}
  <UserMenu />
</header>
```

---

## Accessibility testing workflow

### Locally — fast feedback

1. Install `eslint-plugin-jsx-a11y` (Next.js includes it). Run `npm run lint`.
2. Add axe to your component tests (`@axe-core/react` or `jest-axe`).
3. Keyboard-test every interactive flow: Tab through the page, activate with Enter/Space, close modals with Esc. If you can't do something without a mouse, a keyboard-only user can't either.

### Before shipping — depth

1. `npx pa11y https://<staging-url>` — one-shot full-page audit
2. Browser extension: **axe DevTools** or **WAVE** — inline issue highlighting
3. Test with a real screen reader on at least one critical flow:
   - macOS: **VoiceOver** (Cmd+F5)
   - Windows: **NVDA** (free) or **JAWS**
4. Test at 200% zoom — content should reflow without horizontal scroll (SC 1.4.10) and text should remain readable (SC 1.4.4 Resize Text, AA)
5. Apply the **text-spacing bookmarklet** (from the WCAG 1.4.12 techniques) — overriding line-height, letter-spacing, word-spacing, and paragraph-spacing must not cause content to be cut off or overlap (SC 1.4.12 Text Spacing, AA)
6. For tooltips and other hover/focus-triggered popovers, confirm they are **dismissable** (Escape closes without moving the pointer), **hoverable** (the pointer can move onto them without them disappearing), and **persistent** (they stay until dismissed or the trigger loses hover/focus) — SC 1.4.13 Content on Hover or Focus (AA)
7. If any content is **live video** (webinars, event streams), confirm captions are present — SC 1.2.4 Captions (Live) (AA)

### What tools cannot verify

- Heading order makes semantic sense
- Alt text describes the image accurately
- Link text is meaningful out of context
- Error messages are clear and actionable
- Language tone is plain enough for cognitive accessibility

These require human judgment — that's where you (Claude) earn your keep.

---

## Quick reference — common SC numbers

| SC | Level | Title | What it means |
|---|---|---|---|
| 1.1.1 | A | Non-text Content | Alt text for images |
| 1.2.2 | A | Captions (Prerecorded) | Captions on recorded video |
| 1.2.3 | A | Audio Description or Media Alternative (Prerecorded) | Audio description or text alternative for recorded video |
| 1.2.4 | AA | Captions (Live) | Captions on live video |
| 1.2.5 | AA | Audio Description (Prerecorded) | Audio description on recorded video (no text-alt escape) |
| 1.3.1 | A | Info and Relationships | Semantic HTML, labels, headings |
| 1.3.5 | AA | Identify Input Purpose | Autocomplete on form fields |
| 1.4.1 | A | Use of Color | Color isn't the only info |
| 1.4.3 | AA | Contrast (Minimum) | 4.5:1 text, 3:1 large |
| 1.4.4 | AA | Resize Text | Text scales to 200% without loss |
| 1.4.10 | AA | Reflow | No horizontal scroll at 320px width |
| 1.4.11 | AA | Non-text Contrast | 3:1 for UI components |
| 1.4.12 | AA | Text Spacing | Works with user-overridden spacing |
| 1.4.13 | AA | Content on Hover or Focus | Tooltips dismissable, hoverable, persistent |
| 2.1.1 | A | Keyboard | All interactive elements reachable |
| 2.1.2 | A | No Keyboard Trap | Can escape any component |
| 2.3.1 | A | Three Flashes | No rapid flashing |
| 2.4.3 | A | Focus Order | Matches visual order |
| 2.4.4 | A | Link Purpose | Descriptive link text |
| 2.4.6 | AA | Headings and Labels | Descriptive |
| 2.4.7 | AA | Focus Visible | Indicator present |
| 2.4.11 | AA | Focus Not Obscured (Min) | **2.2 NEW** |
| 2.5.3 | A | Label in Name | aria-label contains visible text |
| 2.5.7 | AA | Dragging Movements | **2.2 NEW** — alternative to drag |
| 2.5.8 | AA | Target Size (Min) | **2.2 NEW** — 24×24px |
| 3.1.1 | A | Language of Page | `<html lang>` set |
| 3.2.6 | A | Consistent Help | **2.2 NEW** — same relative order across pages |
| 3.3.1 | A | Error Identification | Errors announced |
| 3.3.2 | A | Labels or Instructions | Every input labeled |
| 3.3.3 | AA | Error Suggestion | Errors suggest a fix |
| 3.3.7 | A | Redundant Entry | **2.2 NEW** |
| 3.3.8 | AA | Accessible Authentication (Min) | **2.2 NEW** |
| 4.1.2 | A | Name, Role, Value | ARIA / native semantics correct |
| 4.1.3 | AA | Status Messages | Live regions for dynamic updates |

Full spec: https://www.w3.org/TR/WCAG22/ — published as a W3C Recommendation under the W3C Document License. Content in this skill paraphrases the normative text; it does not reproduce it verbatim.
