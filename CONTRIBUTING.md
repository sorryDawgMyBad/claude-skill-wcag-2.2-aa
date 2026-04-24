# Contributing

**This repository is not accepting external pull requests.**

This is a solo-maintained project, and the maintainer prefers to personally validate every change that ships — particularly important for a skill whose purpose is teaching accessibility practices. A subtly wrong pattern can teach bad habits to every downstream user, so each change needs a review the maintainer can stand behind.

## What you can do

### Issues are welcome

- **Bug reports** — use the *Bug report* issue template.
- **Pattern suggestions** — use the *New pattern suggestion* template. Describe the pattern, link to authoritative sources (W3C WCAG, WAI-ARIA, MDN, vendor documentation, accessibility expert blog posts with cited spec references). The maintainer will evaluate and, if adopted, implement it.
- **Accessibility corrections** — use the *Accessibility correction* template if a pattern or example in the skill is incorrect, would introduce a regression, or misrepresents WCAG. Please cite an authoritative source.

### Fork freely

The skill is Apache-2.0 licensed. Fork it, adapt it for your team, your stack, your component library, or your jurisdiction's accessibility standard, and use it however you like. The license does not require you to upstream changes.

## What happens to PRs

PRs opened against this repository will be closed without merge, politely, and the author will be asked to open an issue instead. This is not a comment on the quality of the contribution — it's a blanket policy so the maintainer is not in the position of accepting code they cannot confidently review.

## Code of Conduct

All interactions in issues must follow the [Code of Conduct](CODE_OF_CONDUCT.md). Be respectful, assume good intent, critique the work not the person.

## Reporting accessibility issues in the skill content

If you find a code example in this skill that would *introduce* an accessibility regression if followed (for example: a pattern that fails a WCAG success criterion, a misuse of ARIA that would break a screen reader, a focus-management example that would create a keyboard trap), please open an *Accessibility correction* issue. For sensitive disclosures, contact the maintainer via the email on their GitHub profile rather than filing publicly.

## What this project is not

- It is not a place to ask for accessibility audits of your own code or website.
- It is not a place to ask for legal or compliance advice. Consult qualified counsel and accessibility auditors for that.
- It is not a forum for general WCAG questions. The [W3C WAI mailing lists](https://www.w3.org/WAI/about/groups/) and [Stack Overflow's accessibility tag](https://stackoverflow.com/questions/tagged/accessibility) are better venues.