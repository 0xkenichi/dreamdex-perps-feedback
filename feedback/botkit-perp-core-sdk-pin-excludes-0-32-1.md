# Bot kit perp-core pins markets-sdk ^0.31.0, so it can never install 0.32.1
Severity: normal
Area: SDK

## Steps to reproduce
1. Clone https://github.com/somnia-chain/dreamdex-bot-kit (default branch `main`, HEAD 2579866, 12 Oct 2026).
2. Open `packages/perp-core/package.json`. The dependency is `"@somnia-chain/markets-sdk": "^0.31.0"`.
3. Open `package-lock.json`. `packages/perp-core/node_modules/@somnia-chain/markets-sdk` resolves to `0.31.0`.
4. Run `npm install` as the README says, then `npm start -w perp-starter`. All four perp-* bots go through perp-core, so they run on SDK 0.31.0.

## Expected
The perp-* bots use SDK 0.32.1. The dreamDEX docs say that is the version the app runs, and it is npm `latest` (published 2026-10-13).

## Actual
For a 0.x version, npm reads `^0.31.0` as `>=0.31.0 <0.32.0`. No `npm install` or `npm update` can move perp-core to 0.32.x. The lockfile pins 0.31.0, published 2026-09-29.

The SDK's own migration guide (`docs/MIGRATING.md`, section "0.31.0 → 0.32.0", in the 0.32.1 tarball) indexes 22 breaking changes between the two versions. One of them makes a Hideki bot built on 0.32.1 behave differently, because `SOMNIA_TESTNET_ADDRESSES` changed contents. That is filed separately as `botkit-perp-hideki-uses-shannon-address-preset.md`.

I checked whether this is already fixed or reported. On 29 Oct 2026, all four remote branches that contain perp-core still have `^0.31.0`: `main`, `feat/perp-strategies`, `fix/perp-preflight-trading-key` and a dependabot branch. None of the 52 issues and PRs mentions the SDK version for perps. The open dependabot PR #51 only bumps vitest.

## Evidence
- https://github.com/somnia-chain/dreamdex-bot-kit/blob/25798660bbe4674a6f83a960b202852896e2fed4/packages/perp-core/package.json#L17: `"@somnia-chain/markets-sdk": "^0.31.0",`
- https://github.com/somnia-chain/dreamdex-bot-kit/blob/25798660bbe4674a6f83a960b202852896e2fed4/package-lock.json#L2575-L2577: `"packages/perp-core/node_modules/@somnia-chain/markets-sdk": { "version": "0.31.0", "resolved": ".../markets-sdk-0.31.0.tgz"`
- `GET https://registry.npmjs.org/@somnia-chain%2Fmarkets-sdk` → HTTP 200. Response has `dist-tags.latest = "0.32.1"`, time 0.31.0 = 2026-09-29T09:05Z, 0.32.0 = 2026-10-12T13:25Z, 0.32.1 = 2026-10-13T09:04Z. The kit's last commit (2026-10-12 10:34 UTC) predates 0.32.0.
- The CI `sdk-floor` check in `scripts/checks.mjs` (L208-L217) only reads `packages/ec-core/package.json`, so nothing in CI looks at the perp-core pin. ec-core is also behind, on `^0.30.0`, but it is not perps.

## Environment
SDK version 0.32.1 (@somnia-chain/markets-sdk), network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review)
