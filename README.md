# WCAG 2.2 Level AA — Claude Code Skill

A Claude Code skill that brings [WCAG 2.2 Level AA](https://www.w3.org/TR/WCAG22/) accessibility checks into your coding workflow. When you write, modify, or review UI code (JSX/HTML/Vue templates, interactive components, images, videos, color/contrast decisions, focus management, keyboard handlers, ARIA attributes, error messages), Claude auto-loads this skill, walks the relevant WCAG success criteria with you, and provides concrete DO/DON'T code patterns. It also orchestrates complementary tooling — `eslint-plugin-jsx-a11y`, `axe-core`, and `pa11y`.

---

## ⚠️ Disclaimer — Please Read

**This skill does not guarantee accessibility, WCAG conformance, or legal compliance.** It is provided **"AS IS" without warranty of any kind**, express or implied. See [LICENSE](LICENSE) for the full warranty disclaimer and limitation of liability under Apache License 2.0.

Specifically:

- **No guarantee of WCAG conformance.** Following every suggestion in this skill does **not** demonstrate conformance with WCAG 2.2 at Level A, AA, or any other level. Formal conformance requires a documented evaluation by qualified auditors against the full normative spec, covering pages you build, assets you host, content you publish, third-party components you embed, and interactions you've never tested. This skill is an aid during authoring — not a substitute for that evaluation.
- **No guarantee of legal or regulatory compliance.** Using this skill does **not** by itself satisfy the Americans with Disabilities Act (ADA), Section 508 of the US Rehabilitation Act, the European Accessibility Act, EN 301 549, AODA, UK Equality Act 2010, or any other statute, regulation, court order, settlement, or contractual accessibility obligation. Legal standards change, vary by jurisdiction, depend on facts specific to your organization and users, and often require documentation (VPAT/ACR authoring, audit trails) this skill does not produce. **If accessibility conformance has legal, regulatory, or contractual consequences for you, engage qualified counsel and an independent, qualified accessibility auditor.** Do not treat this repository's output as legal advice.
- **No substitute for real assistive-technology testing.** Tools and checklists cannot evaluate whether a flow actually works for a blind user with NVDA, a low-vision user at 400% zoom, a motor-impaired user on a switch, a cognitively disabled user under time pressure, or any real user of assistive technology. Research consistently shows that automated tools catch somewhere around 30–40% of WCAG issues. The rest require manual review and, ideally, testing with users of the relevant assistive technologies and disability communities.
- **No substitute for accessibility expertise.** Accessibility involves legal, design, content, and user-research judgment that a coding skill cannot provide. For complex interactive components, regulated contexts, content migrations, or accessibility remediation at scale, engage accessibility professionals.
- **Code examples are illustrative.** Examples in this skill are simplified to highlight one pattern at a time. They may omit framework-specific concerns, error handling, edge cases, browser quirks, or production details relevant to your codebase. Adapt them carefully; do not copy-paste without review.
- **Third-party components are not in scope.** Embedded widgets (payment forms, chat widgets, video players, analytics overlays, CMS-authored content, marketing scripts) can introduce accessibility failures regardless of how conformant your first-party code is. This skill flags the pattern but cannot fix upstream problems.
- **Spec coverage is not complete.** This skill focuses on Level AA of WCAG 2.2, which by definition includes Level A. It does not cover Level AAA. It does not cover other normative specifications (ARIA Authoring Practices are referenced but non-normative; WAI-ARIA, ATAG, UAAG, the PDF/UA spec, mobile platform guidelines, and EPUB accessibility are out of scope).
- **No liability.** The authors and contributors accept no liability for damages, costs, legal exposure, lost revenue, or harm arising from use, misuse, or inability to use this skill, to the maximum extent permitted by law. If your use case involves material legal, financial, reputational, or safety consequences, **obtain independent professional advice.**

In short: this is a well-researched checklist and set of code patterns intended to help developers catch common accessibility problems earlier and learn what to look for. It is not a conformance attestation, not a compliance tool, and not a replacement for qualified accessibility review. Treat its output as a starting point.

---

## What this skill does

When active, the skill:

1. **Triggers automatically** when you work on code matching UI surfaces — JSX/HTML/Vue templates, interactive components (forms, buttons, modals, menus, accordions), images, videos, color/contrast decisions, focus management, keyboard handlers, ARIA attributes, or interactive copy.
2. **Runs a toolchain check** and recommends installing `eslint-plugin-jsx-a11y`, `axe-core`, or `pa11y` if they are not present (it does not auto-install).
3. **Applies WCAG 2.2 AA semantic heuristics** that automated tools cannot detect — alt-text meaning, link-text-out-of-context, heading hierarchy intent, error-message actionability, ARIA misuse.
4. **Surfaces 2.2 NEW success criteria** (2.4.11 Focus Not Obscured, 2.5.7 Dragging Movements, 2.5.8 Target Size, 3.2.6 Consistent Help, 3.3.7 Redundant Entry, 3.3.8 Accessible Authentication) that many developers have not yet internalized.
5. **Loads concrete DO/DON'T code patterns** from [patterns.md](patterns.md) for specific categories — images, labels, keyboard, contrast, ARIA components, forms, authentication, target size.
6. **Refuses to declare UI work complete** while known, unfixed AA issues exist, unless the user explicitly defers with a `TODO(a11y SC X.Y.Z)` marker.

It is complementary to external auditing and assistive-tech testing — not a replacement.

## Requirements

- [Claude Code](https://claude.com/claude-code) (CLI, desktop app, or IDE extension)
- The skill is installed as a user-level Claude Code skill

## Installation

Clone this repo into your Claude Code user skills directory:

**macOS / Linux:**

```bash
git clone https://github.com/sorryDawgMyBad/claude-skill-wcag-2.2-aa ~/.claude/skills/wcag-2.2-aa
```

**Windows (PowerShell):**

```powershell
git clone https://github.com/sorryDawgMyBad/claude-skill-wcag-2.2-aa $env:USERPROFILE\.claude\skills\wcag-2.2-aa
```

That's it. Claude Code discovers skills in `~/.claude/skills/` automatically on next session start.

### Updating

```bash
cd ~/.claude/skills/wcag-2.2-aa
git pull
```

## Verifying it works

Start a new Claude Code session and ask something that touches a UI surface, e.g.:

> *"Add a close button to this modal in src/components/Dialog.tsx."*

The `wcag-2.2-aa` skill should activate, and Claude should:

- Check for an accessible name on the button (SC 2.4.4, 2.5.3)
- Check for keyboard support — Esc to dismiss, focus trap, return focus on close (SC 2.1.1, 2.1.2, 2.4.3)
- Check target size against SC 2.5.8 (24×24 CSS pixels)
- Recommend a library (`@radix-ui/react-dialog`, `react-aria`, `headless-ui`) rather than a hand-rolled modal
- Ask you to run `npm run lint` and, if wired in, `npx playwright test` before calling the work done

If the skill doesn't activate, force it by referencing it by name or by asking Claude to "check the WCAG 2.2 AA skill."

## What's inside

| File | Purpose |
|---|---|
| [`SKILL.md`](SKILL.md) | Main skill entry point — description, toolchain protocol, WCAG 2.2 AA heuristics, report format, scope limits |
| [`patterns.md`](patterns.md) | Per-category DO/DON'T code patterns (React/JSX primarily; principles port to HTML/Vue/Svelte) |
| [`NOTICE`](NOTICE) | Attribution to the W3C for WCAG 2.2 source material |

## Scope

**Covers:**

- WCAG 2.2 Level AA (which, by definition, includes all Level A criteria)
- Common web component patterns: images, forms, buttons, links, headings, landmarks, modals, accordions, tabs, live regions, contrast, keyboard navigation, focus management, target size, dragging alternatives, authentication, multi-step forms, consistent help
- React/JSX examples primarily, with HTML-equivalent guidance; principles apply to Vue, Svelte, plain HTML, and other stacks

**Does not cover:**

- **Level AAA** — out of scope (see WCAG 2.2 spec for AAA criteria)
- **Legal or regulatory conformance** — ADA, Section 508, EN 301 549, AODA, UK Equality Act, European Accessibility Act, etc. Meeting WCAG is a common prerequisite; meeting *only* this skill's checklist is not sufficient evidence of conformance with any statute, regulation, or contract
- **Non-web surfaces** — native mobile apps (iOS/Android), PDFs, Microsoft Office documents, email templates, kiosks, EPUB, embedded firmware UIs
- **Live assistive-tech behavior** — NVDA, JAWS, VoiceOver macOS/iOS, TalkBack, Dragon, switch controls. These require manual testing, ideally with real users
- **Third-party and embedded content** — CMS-authored content, YouTube embeds, payment widgets, chat widgets, reCAPTCHA, analytics scripts, marketing overlays. The skill flags the pattern but cannot fix upstream issues
- **Cognitive-load and plain-language sufficiency** — real evaluation requires user research with the relevant cognitive disability communities
- **Accessibility documentation artifacts** — VPAT/ACR authoring, conformance reports, accessibility statements
- **Other normative specifications** — WAI-ARIA 1.2+ (referenced but not a substitute for following the spec itself), ATAG, UAAG, PDF/UA

## Related tools and skills

This skill occupies one specific slot: **auto-triggering, in-editor, WCAG 2.2 AA–indexed, multi-framework DO/DON'T reference that complements automated tooling**. For adjacent needs:

- **axe DevTools** (browser extension) — runtime audit of a rendered page. Complementary: run axe on staging, use this skill while authoring.
- **`@axe-core/playwright`**, **`@axe-core/react`**, **`jest-axe`** — axe integrated into your test suite. The skill recommends these; this repo does not supply them.
- **pa11y** — CLI one-shot audit. Complementary.
- **Manual screen-reader testing** — NVDA (Windows, free), JAWS (Windows, commercial), VoiceOver (macOS/iOS, built-in), TalkBack (Android, built-in). This cannot be replaced by tooling.
- **User testing with disabled users** — no tool or skill substitutes for this, especially for complex interactive components or cognitive-accessibility concerns.
- **Qualified accessibility auditors** — for VPAT/ACR authoring, conformance testing, regulated contexts, or anything with legal consequences. This skill is a development aid; it is not an audit.

Pointers, not endorsements. Pick the combination that fits your workflow.

## Contributing

**This repository is not accepting external pull requests.** It is solo-maintained, and the maintainer prefers to personally validate every change that ships to a skill whose purpose is teaching accessibility practices — a subtly wrong pattern can teach bad habits to every downstream user.

Issues, however, are welcome:

- **Bug reports**
- **Pattern suggestions** (please cite W3C WCAG, WAI-ARIA, MDN, or vendor documentation)
- **Corrections** to existing patterns

See [CONTRIBUTING.md](CONTRIBUTING.md) for details. The skill is Apache-2.0 licensed — you are free to fork and adapt it for your own team or stack.

## Attribution

This skill is a derivative work based on the **Web Content Accessibility Guidelines (WCAG) 2.2**, © W3C® (MIT, ERCIM, Keio, Beihang), licensed under the [W3C Document License](https://www.w3.org/copyright/document-license/). See [NOTICE](NOTICE) for full attribution.

The W3C and the Web Accessibility Initiative (WAI) do not endorse this skill. All interpretations, examples, and recommendations here are the author's, based on the public W3C material. Any errors are the author's alone and should not be attributed to W3C.

## License

This project is licensed under the **Apache License 2.0** — see [LICENSE](LICENSE) for details. The WCAG 2.2 source material this skill references is licensed separately under the W3C Document License — see [NOTICE](NOTICE).

## Reporting issues in this skill

If you find a *code example or recommendation in this skill that would introduce an accessibility regression if followed*, please open a [GitHub issue](https://github.com/sorryDawgMyBad/claude-skill-wcag-2.2-aa/issues) using the "Accessibility correction" template. For urgent corrections you would rather not disclose publicly, email the maintainer via the email on their GitHub profile.

For accessibility issues in *your own code*, this skill is not the right place — consult a qualified accessibility professional and, if relevant, disabled-user testing.
