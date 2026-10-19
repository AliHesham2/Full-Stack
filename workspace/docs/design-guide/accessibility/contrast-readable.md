# Contrast and Readability

## WCAG 2.2 Contrast Requirements (Current Standard)

| Element | Minimum Ratio |
|---------|:---:|
| Normal text (< 18pt / 14pt bold) | 4.5:1 |
| Large text (≥ 18pt or ≥ 14pt bold) | 3:1 |
| UI components and graphical objects | 3:1 |
| Focus indicators | 3:1 |

### What Counts as "UI Components and Graphical Objects"
This includes: button backgrounds, input borders, icons that convey information, toggle tracks, slider tracks, and any visual element the user needs to perceive to operate the interface.

This does NOT include: purely decorative elements, text that is part of a logo.

## Practical Contrast Checking

### Colors That Commonly Fail
- Light gray (#999, #aaa) on white — FAILS for text
- Medium gray (#bbb, #ccc) on white — FAILS for everything
- White text on light backgrounds (blue-100, green-100)
- Colored text on colored backgrounds without checking

### How to Check
- Browser DevTools: Color picker shows contrast ratio
- WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
- Automated: axe-core, Lighthouse

## WCAG 3.0 and APCA (Do Not Rely On Either Yet)

### Accurate Status as of August 2026

- WCAG 3.0 is a W3C Working Draft (latest: March 2026) — years from completion (~2030 at the earliest per working-group participants).
- **The final contrast algorithm is undetermined.** APCA was the leading candidate but is
  **no longer in the draft**. This is directly checkable in the published draft series — APCA
  is named 11 times in each of the three 2021 Working Drafts (21 Jan, 8 Jun, 7 Dec 2021),
  along with the "Visual Contrast of Text" guideline built on it, and appears **zero times**
  in every draft from **24 July 2023** onward, up to and including the current
  `WD-wcag-3.0-20260303`. The current draft leaves contrast as an unfilled placeholder
  (`@@[non-text-contrast]`). APCA continues only as independent draft guidance from its
  author, outside the W3C process.
- The proposed conformance model has included Bronze/Silver/Gold tiers instead of A/AA/AAA, but this remains unsettled.

### Why APCA Is Still Interesting (Not for Compliance)

- Accounts for **perceptual contrast** rather than mathematical luminance ratio
- Considers **font size and weight** (smaller/lighter text needs more contrast)
- Uses a numeric score (e.g., Lc 60) instead of a ratio
- In practice, colors passing APCA for a given size/weight exceed WCAG 2 requirements in most cases — useful as a stricter design target, never as a compliance claim

| APCA Score (approx.) | Typical Use |
|----------------------|-------------|
| Lc 90+ | Preferred body text |
| Lc 75–90 | Body text |
| Lc 60–75 | Large text only |
| Below Lc 60 | Decorative/large display only |

**Bottom line:** Design for WCAG 2.2 AA now. Revisit when WCAG 3 nears finalization.

## Beyond Contrast: Readability

### Font Size
- Body text: 16px (1rem) recommended for reading-heavy content; 13–14px acceptable as the base in dense expert tools (dashboards, IDEs) — never below 13px.
- Large text for WCAG purposes: 18pt (24px) or 14pt bold (18.67px bold)

### Line Height
- Body text: 1.5 minimum, 1.5–1.6 recommended
- Headings: 1.1–1.3

### Text Spacing Override
Users must be able to override text spacing per WCAG 2.1 Success Criterion 1.4.12. Test that your layout doesn't break when:
- Line height: 1.5
- Paragraph spacing: 2em
- Letter spacing: 0.12em
- Word spacing: 0.16em

## References
- WCAG 2.2 Understanding docs: https://www.w3.org/WAI/WCAG22/Understanding/
- WCAG 3.0 Working Draft (March 2026): https://www.w3.org/TR/wcag-3.0/
- APCA (independent draft guidance): https://git.apcacontrast.com
- Adrian Roselli, "WCAG3 Contrast as of April 2026": http://adrianroselli.com/2026/04/wcag3-contrast-as-of-april-2026.html
- WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
