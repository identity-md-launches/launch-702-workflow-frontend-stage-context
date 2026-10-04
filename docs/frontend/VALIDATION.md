# Worker validation — COMP terminal

Date: 2026-10-04. This is worker evidence, not an independent certification or a publication result.

## Scope and consequential assumptions

Frontend source/configuration/lockfile are under `web/`; the static export prepared for submission is under `dist/`; documentation and actual browser screenshots are under `docs/`. Solidity, root configuration, libraries and deployment files remain untouched.

Two assignment conditions cannot be established from the supplied inputs:

1. There is no previous frontend or design system in the pinned checkout or local history for `web/`, `dist/`, or root `DESIGN.md`. Historical palette/layout/motion and prior-pane regression equivalence are therefore **unverified**. The requested neutral, monospace, hairline, single-viewport direction is implemented; all seven panes have functional controls.
2. Root `DESIGN.md` lies outside the explicit allowed write paths. The complete implemented-design record is at `docs/DESIGN.md`.

The actual handoff uses MockWorkOracle, despite the assignment's description of an attested work oracle. The terminal discovers the linked oracle, labels faucet mode honestly and exposes all requested attested fields when SwarmWorkOracle is present. Attested mode is verified with ABI-encoded fixtures; it is not falsely attributed to this live deployment.

The launch is application contracts only, so Uniswap swaps, quotes, approvals and liquidity are **not applicable**. The supplied network block is preserved unchanged; no extra router/pool is added. No real transaction was broadcast, no signing authority was used, and no publication/deployment action was performed.

## Executed checks

| Check | Result |
| --- | --- |
| `npm ci --prefix web --offline --cache /tmp/comp-npm-cache` | Passed from the supplied lockfile using the worker's populated npm cache; no cache is submitted |
| `npm run --prefix web build` | Passed after final source corrections; includes strict TypeScript check and production export/manifest generation |
| `npm run --prefix web test` | Four tests passed; malformed/oversized/overprecision amounts, allowed zero governance value, reserve/position/mixed payouts, 200 varied rounding/conservation cases, fee direction, zero price/output, address and custom-error handling |
| `PLAYWRIGHT_BROWSERS_PATH=/tmp/comp-playwright npm run --prefix web test:browser` | 24 interaction/render/security checks passed against the actual production export under `/preview/` |
| `npm run --prefix web check:export` | Four handoff canonical ABI hashes match; all 16 asset hashes and exact configuration match |
| `node web/scripts/live-check.mjs` | All three configured public endpoints returned chain ID 11155111 and nonempty code for all four handoff contracts |
| `node web/scripts/live-state.mjs` | Read-only getters decoded at explicit block 11844083; evidence includes linked addresses, fee/ceiling/reserve and actual work-oracle mode |

Vite emits its advisory warning about a roughly 550 kB uncompressed JavaScript chunk. The complete static export, including all ABIs and manifest, is 685,779 bytes, comfortably within the asset/count and response-budget limits. There are no remote font/image assets, dependency archives, caches or node_modules in the submission. Exact packaging size is recorded in `packaging.json`.

`web/tests/browser.mjs` starts and closes its own foreground preview and Chromium process. The assigned browser connector returned `Transport closed`; the local Playwright/Chromium route was used successfully instead. No browser checks are inferred solely from source.

## Interaction evidence

See `browser-results.json` for the timestamp, Chromium version and each assertion. Public RPC methods and the EIP-1193 wallet were intercepted with ABI-encoded fixture responses; these are not on-chain executions.

Checked: missing wallet; rejected connection and retry; wrong chain; 4902 add-chain with exact handoff parameters; reserve-only, position-only and mixed redemption; size-dependent fee reads; candidate requirement and ceiling rejection; quote invalidation; simulated backing-guard rejection; stale-feed gating; both oracle modes; task age/rate/rights; exact approval as a separate transaction; rejected signing and retry; receipt and allowance refresh; deposit/borrow/repay/withdraw; work mint; keeper inspect/mark/clear/liquidate; reserve sync; governance apply/propose; reporter fallback; redemption transaction encoding and receipt flow; RPC failure/recovery; missing contract code; ABI tampering and retry.

A keyboard-only path edits the redemption amount, tabs through slippage/candidate, quotes with Enter, opens review with Enter, cancels with Escape and restores focus to Review redemption. A focused slippage field was visually inspected in `keyboard-focus.png`. Native dialog focus behavior is used; a screen-reader session was not performed.

## Better Interface — six-domain review

The pinned workflow and all six core domains were read before implementation. Its documentation method was applied to final source values. Each row is a scoped review, not a claim of universal compliance.

| Domain | Coverage and observations | Limits |
| --- | --- | --- |
| Accessibility | Checked native labels/buttons/select/details/dialog, skip link, focusable scroll regions, error associations and live statuses; visible keyboard outline inspected; keyboard redemption flow and focus return passed; desktop/320px Axe WCAG A/AA scan found zero violations | No screen-reader session, native-device touch, native 200% zoom, or comprehensive assistive-technology matrix |
| Layout | Checked DOM order, bounded pane overflow and progressive disclosure; rendered 1440×900, 1280×800, 900×900, 390×844, 320×740; document/body dimensions exactly match each viewport; every mobile pane reachable | No historical frontend to compare; RTL and localized copy expansion not tested (English-only interface) |
| Writing | Checked action verbs, token units, exact transaction consequences, recovery messages, fee retention and reserve-first explanation; published-tally/no-proof disclosure and faucet limitations are explicit | No editorial localization review; no claim that COMP's market price equals its debt unit |
| Typography | Checked system monospace stack, numeric alignment, wrapping, full precision in quotes and 16px mobile inputs; inspected saved renders for clipped page content and full quote scrolling | System font varies by platform; screen-reader pronunciation and non-Linux font rendering untested |
| Colors | Neutral-only source and computed CSS; measured main text/pane 15.60:1, secondary/pane 7.61:1, input boundary/background 4.22:1; visible focus inspected | Dark-only by brief; no alternate theme or wide-gamut palette; not every hover/disabled contrast pair separately measured |
| UI | Checked hover/focus/disabled/loading/error/empty/review/confirmed states; 150ms named color transitions with documented easing; reduced-motion computes zero duration; pane-only scrolling and exact approval flow | No 10%-speed animation-panel replay; native wallet extension screens were mocked |

No marketing imagery, new hue, gradient, glow or mascot was introduced. Actual screenshots were opened and inspected at both required desktop sizes, 320px and in the keyboard-focused quote state. Other saved viewport screenshots were measured automatically; they are available for inspection.

## Findings, disposition and rechecks

| Finding | Source | Disposition and evidence |
| --- | --- | --- |
| Medium: asynchronous quote could complete after its input/snapshot changed | `web/src/Redemption.tsx:45`, `:125`, `:211` | Generation invalidation discards the old result; input-change invalidation is browser-tested. Delayed-response interleaving was reviewed in source, not separately latency-injected. |
| Medium: validation errors lacked an explicit input association | `web/src/actions.tsx:94`, `:140`; `web/src/Redemption.tsx:232` | Added feedback IDs/`aria-describedby`, field-specific invalid state and focus on parsing failure. Final Axe and keyboard checks passed; a screen-reader announcement was not tested. |
| Medium: the live handoff would mislead if attested metrics were assumed | `web/src/state.ts:192`; `web/src/Panes.tsx:151` | Probe linked oracle, distinguish attested/faucet/unknown modes, retain missing values as unavailable. Both modes tested; live `deployer()` confirms faucet mode. |
| Medium: operator-only test IMD mint needed an explicit role gate | `web/src/state.ts:123`; `web/src/Panes.tsx:123` | Read MockIMD deployer and disable faucet control for other wallets; write simulation still enforces authorization. Source review and final suite passed. |
| Low: the governor must be able to propose zero COMP per task | `web/src/actions.tsx:110`; `web/src/Panes.tsx:727` | Added a nonnegative decimal field kind for governed rates/ceilings; ordinary transfer amounts still reject zero. Unit assertion covers the zero-value parser. |
| Low: clipped pane content needed a visible overflow cue | `web/src/actions.tsx:177`; `web/src/style.css:238` | Added a neutral vertical scroll cue, keyboard focus and native thin scrollbars. Rendered desktop/mobile and pane-selection checks passed. |

During validation the harness initially failed to parse minified three-digit CSS hex values and expected an internal RPC error instead of the user-facing message. Those harness expectations were corrected; the final measured contrasts and recovery assertions above are from successful reruns. No palette change was needed.

## Live observations and untested behavior

`live-check.json` records read-only chain/code checks. `live-state.json` records block 11844083 on the first configured endpoint. At that block, reserve IMD was zero, the current fee/floor/cap were 50/50/500 bps, minCR was 200%, ceiling was 250%, and the USD feed returned zero with stale status. The linked MockWorkOracle's operator getter answered; its governed COMP-per-task setting was 0.01. These are dated observations, not seeded demo metrics.

Reproduce the explicit-block calls with `node web/scripts/live-state.mjs 11844083` where the configured endpoint retains that history. Live wallet signing, actual redemption/approval/liquidation/governance execution, attestation publication, browser-to-public-RPC reliability/CORS, wallet extension compatibility, reorg handling beyond one confirmation and control-plane publication checks remain untested. The site conservatively refuses writes without verified fresh prerequisites and always simulates before requesting signing.

## Completion

The useful frontend source/export and worker validation are complete within the permitted paths. The requested workspace Git commit could not be created: `git add` and `git commit` both failed with `Unable to create .git/index.lock: Read-only file system`. No permission escalation or metadata workaround was attempted; the real workspace files remain uncommitted. An isolated scratch clone is used solely to measure an equivalent full submission bundle, not to alter the protected repository metadata. Historical design/regression equivalence and a root-level DESIGN.md remain unmet for the input/scope reasons stated above. An attested tally cannot be shown as live on a faucet deployment; its truthful alternative and supported attested mode are delivered. No claim of overall workflow completion or independent approval is made.
