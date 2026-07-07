# React (Web) — presentation style pattern

Every frontend adaptation **must** include visual design tokens and per-view style guidance — not only routes and hooks.

## Sources (priority)

1. Epic / ADR design-token or visual-fidelity requirements
2. IR `presentation` views and state patterns (loading, empty, error, success)
3. Reference renders or story descriptions in uploads
4. Sensible defaults documented in `meta.assumptions` when epic is silent

## Required outputs

- `Presentation/PresentationStyle.json` — machine-readable tokens + component/view styles
- `Presentation/PresentationStyle.md` — human-readable style guide with CSS variable table

## Token catalog (minimum)

- Colors: primary, secondary, background, surface, text, semantic (error/success/warning)
- Typography: font families (include RTL alternate when epic mentions bidirectional layout)
- Spacing scale and border radius
- Elevation / shadow for cards and modals

## Per-view styling

For each page/route in `PresentationDesign.json`:

- Layout pattern (stack, grid, carousel, modal)
- Which tokens apply to header, body, CTA, list rows
- State-specific visuals: skeleton/loading, empty illustration placeholder, error banner, success confirmation

## Component style bindings

Map shared components (`StatefulPageShell`, `ConfirmationModal`, etc.) to:

- CSS custom properties or token references
- Interactive states: default, hover, focus, disabled, loading
- Minimum touch target size (44px) for mobile-first web

## CSS strategy

Document in `cssStrategy`:

- Prefer **design tokens as CSS variables** (`--color-primary`, `--space-md`)
- Name global token file (e.g. `tokens.css`)
- Note whether CSS Modules, styled-components, or utility classes are assumed — default to token + CSS Modules when epic is silent

## Do not

- Invent brand colors without assumption note
- Skip style artifacts because epic lacked a Figma link — infer from functional UI states
- Put raw CSS in consolidated deliverables — keep in `PresentationStyle.md` under `_internal/`
