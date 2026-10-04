# COMP terminal

A static Vite / React / TypeScript terminal for the attested application deployment. Wallet transport and ABI calls use viem. There is no backend, hosted font, WalletConnect project ID, or private credential. Use an injected EIP-1193 browser wallet. The application is an `evm_contracts` deployment with no trading pool; there is no invented swap or liquidity flow.

## Run and rebuild

From the repository root, with Node 22 or newer:

```sh
npm ci --prefix web
npm run --prefix web typecheck
npm run --prefix web build
npm run --prefix web check:export
npm run --prefix web test
npm exec --prefix web -- playwright install chromium
npm run --prefix web test:browser
npm run --prefix web preview
```

Dependencies are locked in `web/package-lock.json`. Installation needs npm access or a populated local cache; the build itself makes no network requests. On a worker with a read-only home, append `--cache /tmp/comp-npm-cache` to npm installation commands, and use `PLAYWRIGHT_BROWSERS_PATH=/tmp/comp-playwright` for both browser installation and tests. Dependencies, caches and browser binaries are not submitted.

`build` typechecks, writes the production export to repository-root `dist/`, then regenerates `dist/imd-deployment.json`. Never edit the export independently. The manifest script checks the pinned ABIs using local Git objects; preserve the deployed source commit in a build checkout. `test:browser` runs its own bounded HTTP server at a `/preview/` subpath and closes it afterwards. It uses mocked RPC/wallet responses, never a funded wallet. Its screenshots and machine-readable report are in `docs/frontend/`.

## Configuration and ABI provenance

`dist/imd-deployment.json` is the **only runtime deployment configuration**. The app fetches it relative to its entrypoint and fetches its referenced implementation ABIs. It verifies ABI asset SHA-256 and canonical Keccak before making contract calls. The build input `web/deployment-source.json` preserves the supplied handoff and chain table; it is not separately imported by the app. Runtime addresses of constructor-created dependencies come from vault getters; their ABI files are inventory-verified assets under `dist/abi/`. No dependency address is guessed.

The deployed source is `6c08b0fc122bc0e02016cf9c4f507e2dfaadfcc3`. All four handoff ABI arrays are raw, byte-identical copies of `docs/abi/<Contract>.json` from that commit. Six constructor-dependency ABI arrays also come from that commit. The additional `SwarmWorkOracle` array was generated from that same checkout with:

```sh
forge inspect src/SwarmWorkOracle.sol:SwarmWorkOracle abi --json --offline \
  --out test/scratch/abi-out --cache-path test/scratch/abi-cache \
  > web/public/abi/SwarmWorkOracle.json
```

`web/scripts/manifest.mjs` verifies the pinned commit exports, canonical sorted-key Keccak hashes, exact handoff contract set, optional pool key, and unchanged network/wallet-add-chain objects while the pinned inputs exist. It always verifies all exported asset bytes, including `index.html`, CSS, JavaScript and ABIs. The manifest excludes itself. The handoff has no `poolKey`, so none is invented. All files use relative hosting paths; serve `dist/` over HTTP(S), including from a gateway subdirectory.

## Controls and transaction flow

- **Redemption:** live zero-size fee and constant floor/cap, reserve IMD balance, `minCR + redemptionSpread` ceiling, on-chain size comparisons, fee-adjusted IMD output, exact reserve/position split, debt cancelled, candidate checks and a nonzero minimum. Full `redeem` simulation enforces backing/ratio guards. Quotes invalidate on edits, account changes and refreshed state; a late response cannot replace a newer quote.
- **Work:** discover `vault.oracle()`. For `SwarmWorkOracle`, show effective attested count, published tally, credited high-water mark, publication age/staleness, rate, earned/consumed/remaining rights and claimant. Explicitly identify the count as the swarm's published tally, not an on-chain proof. For the deployed **MockWorkOracle**, show faucet mode and wallet rights, with no invented task count or cumulative accounting. Operator grant controls are permission-gated.
- **Position:** exact IMD approval as its own transaction, deposit, borrow, repay, withdraw, plus an operator-only test collateral faucet. COMP burn paths do not ask for approvals.
- **Oracle:** primary/NHI/spot/USD values, ages, staleness and divergence; permission-checked reporter fallback. Signed attestation submission is relayer infrastructure, not a visitor action.
- **Keeper:** inspect candidate debt/ratio/mark, mark, mark for a beneficiary, clear recovered mark and liquidate inside the execution window.
- **Backing:** reserve value, secured collateral, backed/bad debt, work ceiling, registered assets, permissionless sync and operator withdrawal.
- **Governance:** current economics, pending kind/payload/timelock, permissionless apply, governor proposals (including zero COMP per task), cancel and index checkpoint. Contract-only treasury/oracle entrypoints are not presented as visitor actions.

The application verifies public RPC chain ID, nonempty code for handoff and linked contracts, reciprocal vault links and the handoff's feed links. Each snapshot uses a single block; public reads work before connection, wallet-specific values appear after connection. Polling runs every 15 seconds while visible; snapshots older than 45 seconds cannot authorize a transaction. An explicit refresh is available.

Actions check chain/account, simulate, show a native review dialog, simulate again, ask the wallet to sign, wait for one successful receipt and refresh. A synchronous lock prevents concurrent submissions while each action keeps its own label. Approval stays locked through receipt and refetch. Unknown-chain switching offers the exact supplied `wallet_addEthereumChain` parameters, then switches again. Failed reads, stale/divergent feeds and insufficient permissions are explained and gated. Full addresses are available in explorer links/titles and copy controls; address inputs accept validated `0x` addresses, not ENS names.

## Delivery and limitations

See [validation](../docs/frontend/VALIDATION.md), [design](../docs/DESIGN.md), [browser evidence](../docs/frontend/browser-results.json) and [live read evidence](../docs/frontend/live-state.json).

The supplied Git tree contains no previous frontend or design system, so exact visual and behavioral continuity with previous increments cannot be established. The implementation follows the requested single-screen, neutral monochrome, monospace, tabular-number, hairline direction. The requested root `DESIGN.md` conflicts with the explicit write scope; the design document is delivered at `docs/DESIGN.md` instead.

The attested handoff deploys MockWorkOracle, not SwarmWorkOracle. Both modes are implemented; attested mode is interaction-tested with fixture data. Reproduce the recorded live reads with `node web/scripts/live-state.mjs 11844083` (an archival-capable configured endpoint is needed). At observed block 11844083 the real USD price was zero/stale and reserve IMD was zero, so live redemption was unavailable. No transactions were broadcast. No previous frontend regression suite was available. Absolute social-card metadata awaits the publisher's domain; a favicon, title and description are included. Publication, pinning, naming and independent service checks are subsequent control-plane work.

Worker commit limitation: this workspace mounts `.git` read-only. The source/export are present, but local staging/commit failed creating `.git/index.lock`. See the validation record.
