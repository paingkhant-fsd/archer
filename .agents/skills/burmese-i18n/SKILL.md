---
name: burmese-i18n
description: Design, implement, or review Burmese localization in multilingual software, especially translation choices, locale-aware typography, untracked letter spacing, vertical spacing, wrapping, and language switches. Use for Burmese UI copy or layouts; do not use for general translation that does not involve Burmese.
---

# Burmese I18n

Build a multilingual interface in which Burmese is treated as its own typographic and linguistic mode, not as English text substituted into an English layout.

## Translation decisions

- Translate for clarity and natural Burmese usage rather than word-for-word equivalence.
- Do not translate every term. Keep a familiar English term when Burmese users commonly recognize it more readily in English, especially technical vocabulary, product or brand names, role names such as `Freelancer`, API and protocol names, file formats, URLs, code identifiers, and well-known UI terms.
- Prefer Burmese for explanatory copy, validation, instructions, navigation, and actions when the Burmese wording is natural and unambiguous.
- Do not invent awkward transliterations merely to eliminate English. Intentional mixed-language copy is valid.
- Follow an existing product glossary or established catalog choices. When no glossary exists, make a small, consistent term list as part of the implementation instead of translating the same concept differently across screens.
- Preserve placeholders, interpolation variables, markup, currency codes, identifiers, and plural/count behavior. Verify catalog key parity, but do not treat an intentionally retained English value as a missing translation.
- Leave user-authored or server-authored content in its original language unless the product explicitly provides translated content. Never imply that untranslated content was machine-translated.

## Language-aware typography

- Set a valid `lang` attribute (`my` or a more specific locale such as `my-MM`) on the document or localized subtree, and update it when the user changes language.
- Select typography with `:lang(my)`, `[lang="my"]`, locale classes, or language-specific design tokens. Do not force Burmese and Latin text through one identical typography rule set.
- Use a font stack with verified Myanmar glyph support. Preserve sensible fallbacks; do not assume the Latin brand font contains usable Burmese glyphs.
- Let the font determine line boxes with `line-height: normal` when it renders well. If explicit tuning is required, use a Burmese-only, unitless line-height with more vertical room than the Latin setting. Do not apply a cramped fixed line-height globally.
- Avoid fixed text heights and single-line assumptions. Prefer `min-height`, block padding, content-driven sizing, and wrapping so stacked marks are not clipped.
- Never apply custom `letter-spacing` to Burmese text. Burmese must use `letter-spacing: normal`; neutralize inherited tracking, design tokens, and utility classes that introduce positive or negative letter spacing. Do not use Latin-specific all-caps transformations. Adjust font choice, weight, size, line height, or layout spacing instead of tracking when Burmese typography needs refinement.
- Give controls, cards, navigation, alerts, and form fields enough vertical padding for Burmese text. A shared component may keep the same structure while its typography and dimensions respond to the active language.

Use language tokens where they fit the codebase, for example:

```css
:root:lang(en) {
  --font-ui: "Inter", sans-serif;
  --line-ui: 1.45;
}

:root:lang(my) {
  --font-ui: "Noto Sans Myanmar", "Myanmar Text", sans-serif;
  --line-ui: normal;
  --control-block-padding: 0.75rem;
  letter-spacing: normal;
}

body {
  font-family: var(--font-ui);
  line-height: var(--line-ui);
}
```

Treat this as a pattern, not a mandatory font or exact spacing value. Match the project's design system and verify the actual font rendering.

## Implementation workflow

1. Inspect the existing locale state, translation catalogs, font loading, responsive styles, and components most likely to constrain text. Search for inherited or component-level `letter-spacing` and tracking utilities that can reach Burmese text.
2. Identify the product vocabulary that should remain English and keep it consistent across the Burmese catalog.
3. Connect the selected locale to both translation lookup and a DOM `lang` attribute. Persist the selection only through the project's established preference mechanism.
4. Add Burmese-specific typography tokens or selectors at the owning design-system layer. Fix reusable components instead of adding screen-by-screen clipping workarounds.
5. Keep formatting locale-aware for dates, times, numbers, and relative time while preserving explicit currency codes required by the product.
6. Review real Burmese copy at narrow and wide widths. Check headings, buttons, selects, inputs, errors, cards, tables, navigation, dialogs, and empty/loading states.

## Verification

- Confirm switching language updates copy, the DOM language, typography, and persisted preference without reloading when the product supports live switching.
- Check for clipped vowel signs or stacked marks, overlapping lines, truncated labels, broken focus outlines, and controls that no longer meet touch-target requirements.
- Confirm Burmese elements have a computed `letter-spacing` of `normal`, including headings, labels, buttons, navigation, badges, and mixed-language components.
- Test wrapping with longer realistic Burmese strings, browser zoom, and the project's supported mobile breakpoints. Do not validate layout using only short sample words.
- Confirm English-retained terms are deliberate and consistent, not accidental untranslated leftovers.
- Run the repository's narrow type, lint, build, and UI checks. Report any visual validation that could not be performed.
