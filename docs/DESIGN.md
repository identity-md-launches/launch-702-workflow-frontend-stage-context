# COMP terminal design

## Overview

A compact, single-screen terminal for COMP borrowers, redeemers, work claimants and operators. Seven independently scrollable panes expose on-chain state and explicit transaction controls. Redemption occupies a full-height column; position, work, oracle, keeper, backing and governance remain visible alongside it on desktop.

The supplied checkout contained no prior `web/`, `dist/` or frontend design document, including in the local Git path history. The brief's monochrome, monospace, tabular-figure and hairline constraints guided this implementation. Exact preservation of absent historical tokens, layout and motion cannot be certified. This document lives under the authorized `docs/` scope because root `DESIGN.md` is not writable under the assignment.

## Colors

Canonical definitions are in `web/src/style.css:1`. The app is intentionally dark-only; there is no theme toggle.

| Token | Value | Purpose |
| --- | --- | --- |
| `--bg` | `#111111` | Page and input background |
| `--surface` | `#161616` | Pane and dialog surface |
| `--raised` | `#202020` | Notice and hover fill |
| `--text` | `#eeeeee` | Main text, primary action fill |
| `--muted` | `#a8a8a8` | Labels, supporting text, unavailable state |
| `--rule` | `#3a3a3a` | Structural hairlines |
| `--control` | `#777777` | Input/button boundaries and scrollbar |
| `--focus` | `#eeeeee` | Two-pixel keyboard outline |

There is no chromatic accent, gradient, glow or mascot. Status meaning is carried by words. Measured exported CSS pairs: main text/pane 15.60:1, secondary text/pane 7.61:1, input boundary/input background 4.22:1. Structural rules are not used as text or required control boundaries. Forced-colors mode uses system `Highlight` and `ButtonText`.

## Typography

System stack: `SFMono-Regular`, `Consolas`, `Liberation Mono`, `monospace`. There are no downloaded fonts, and the exact platform face varies. The root requests 13px with unitless 1.5 line height and `font-variant-numeric: tabular-nums`. Dense metric labels are 11px; auxiliary chrome is 10px. Pane headings are 13px/600; page title 18px/600; hero figures 27px with 1.4 line height. Monospace bold availability follows the host's font stack.

Supporting paragraphs use 1.6 line height. Long IDs and errors wrap; address links expose full addresses in their title and destination. At 760px and below, metrics/body notes become 12px, hero figures 32px, and inputs/selects become 16px to avoid mobile input zoom. No numeric formatting changes the exact amount used in a transaction: full output/minimum values appear in the quote and review.

## Layout

Source: `web/src/style.css`, `.terminal`, `.grid`, `.pane`, `.pane-body`.

- `100dvh` flex shell, no body/page overflow, 20px desktop inline padding. Header, state strip and footer surround a grid with `min-height: 0`.
- Above 1100px: four equal `minmax(0,1fr)` columns, two rows. Redemption spans both rows of column two. All other panes each occupy one cell.
- 761–1100px: three columns and three rows, 12px shell padding; redemption spans all three rows in the middle.
- At 760px and below: native “View pane” select exposes one full-height pane at a time, retaining all seven panes and their state. Body remains fixed; the selected pane scrolls.
- Header of each pane remains outside its scroll area. A vertical scroll cue and native thin scrollbar indicate more content. Pane bodies use `overscroll-behavior: contain` and are keyboard-focusable.
- Primary spacing values: 6–8px inside compact controls/rows, 10–14px pane padding and control groups, 16px grid separation, 20–24px shell/dialog spacing. Controls remain in document flow inside their pane.

Browser measurements confirm document and body dimensions equal their viewport at 1440×900, 1280×800, 900×900, 390×844 and 320×740. Mobile pane switching was exercised for every pane. Native 200% zoom and physical-device behavior were not tested.

## Elevation & Depth

Flat surfaces, without shadows. One-pixel rules communicate pane divisions, disclosures and control boundaries. Notices use a two-pixel leading rule and a slightly lighter neutral fill. The native transaction dialog is the only modal layer, with a black 75% backdrop and a one-pixel control border.

## Shapes

Square-cornered panes, inputs, buttons and meters; no pill or card radius system. The small status mark is a square. Chart-free size comparisons use native meters in the same neutral ramp.

## Components

- `Pane` / `Row` in `web/src/actions.tsx`: indexed title, supporting tag, bounded scroll body; aligned metric labels and right-aligned values. Use these for further protocol state.
- `Action` / `ActionForm`: permission and network gates, action-specific processing label, persistent error text and explanatory disabled state. Field kinds distinguish addresses, positive decimal amounts, nonnegative decimal amounts and integers.
- `AddressLink`: explorer destination, full-address title and copy button. A copy failure does not falsely show success. ENS resolution is not implemented.
- `Redemption` in `web/src/Redemption.tsx`: fee/ceiling metrics, three size comparisons, exact input/slippage/candidate form, simulation-backed quote and source breakdown. Input, snapshot and account changes invalidate the quote. A generation counter discards late responses.
- Work and previous-functionality panes live in `web/src/Panes.tsx`. Native `details/summary` disclose operator and advanced controls, preserving keyboard behavior.
- The native `dialog` in `web/src/App.tsx` reviews the consequence, amount, target, network and wallet before signing. Native modal focus containment and Escape/cancel are retained; cancellation is disabled during submission.

Focus uses a two-pixel outline with a three-pixel offset. Desktop buttons are at least 34px high (compact copy controls 26px); mobile inputs are 44px and primary pane buttons 40px. Errors and statuses use live regions; state changes also have visible words.

Only background, text and border color transitions are used: `--fast: 150ms` with `--ease: cubic-bezier(.2,0,0,1)`. They exist only under `prefers-reduced-motion: no-preference`; reduced-motion computes to zero-duration transitions. There are no entrance, looping, scaling or scroll animations. These are documented implementation choices, not recovered historical values.

## Do's and Don'ts

Start additional protocol content with `Pane` and `Row`, retain the neutral tokens, keep addresses in the runtime deployment configuration, and route writes through the simulation/review/receipt flow. Put secondary/operator actions behind a labeled disclosure. Keep transaction amounts exact even when summary metrics are abbreviated.

Do not add a new hue, gradient, glow, external font, imaginary live metric or decorative illustration. Do not introduce page scrolling or a second runtime address map. Preserve the explicit distinction between published work tallies, available rights and testnet faucet credits.

Design guidance: Jakub Krehel's Better Interface, MIT, pinned commit `267330e1adfc66a718fb65fa6918c1f06d0a689e`. Documentation method: Paul Bakaus's Impeccable, Apache-2.0, commit `9d715cc4f5564a990ca8345abfdd5df6dc9b41c8`. See `docs/frontend/GUIDE-LICENSES.txt` and the six-domain validation record.
