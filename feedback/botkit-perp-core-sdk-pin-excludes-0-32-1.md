# Bot kit perp-core pins markets-sdk ^0.31.0, so it can never install 0.32.1 or 0.33.0
Severity: normal
Area: SDK

## Steps to reproduce
1. Clone https://github.com/somnia-chain/dreamdex-bot-kit (default branch `main`, HEAD 4e75896, 30 Oct 2026; first seen at 2579866, 12 Oct 2026).
2. Open `packages/perp-core/package.json`. The dependency is `"@somnia-chain/markets-sdk": "^0.31.0"`.
3. Open `package-lock.json`. `packages/perp-core/node_modules/@somnia-chain/markets-sdk` resolves to `0.31.0`.
4. Run `npm install` as the README says, then `npm start -w perp-starter`. All four perp-* bots go through perp-core, so they run on SDK 0.31.0.

Re-checked on 15 Nov 2026: the two commits since 2579866 only touch `examples/`, so steps 2-4 give the same result on the current HEAD.

## Expected
The perp-* bots use the SDK the app runs. The dreamDEX perps developer page still says "this page uses version 0.32.1, the one the app runs", and npm `latest` is now 0.33.0 (published 2026-11-09).

## Actual
For a 0.x version, npm reads `^0.31.0` as `>=0.31.0 <0.32.0`. No `npm install` or `npm update` can move perp-core to 0.32.x or 0.33.x. The lockfile pins 0.31.0, published 2026-09-29.

The SDK's own migration guide (`docs/MIGRATING.md`, section "0.31.0 → 0.32.0", shipped in both the 0.32.1 and 0.33.0 tarballs) indexes 22 breaking changes between 0.31 and 0.32, and the 0.33.0 release notes add more marked "BREAKING". One of them makes a Hideki bot on the newer SDK behave differently, because `SOMNIA_TESTNET_ADDRESSES` changed contents. That is filed separately as `botkit-perp-hideki-uses-shannon-address-preset.md`.

I checked whether this is already fixed or reported. On 15 Nov 2026, every remote branch that contains perp-core still has `^0.31.0`: `main`, `feat/perp-strategies`, `fix/perp-preflight-trading-key` and the dependabot branch `npm_and_yarn-b49447145f`. None of the 54 issues and PRs mentions the SDK version for perps. The open dependabot PR #51 only bumps vitest.

## Evidence
- https://github.com/somnia-chain/dreamdex-bot-kit/blob/4e75896896e024c03e2033d5920514f249998cd6/packages/perp-core/package.json#L17: `"@somnia-chain/markets-sdk": "^0.31.0",`
- https://github.com/somnia-chain/dreamdex-bot-kit/blob/4e75896896e024c03e2033d5920514f249998cd6/package-lock.json#L2575-L2577: `"packages/perp-core/node_modules/@somnia-chain/markets-sdk": { "version": "0.31.0", "resolved": ".../markets-sdk-0.31.0.tgz"`
- `npm view @somnia-chain/markets-sdk dist-tags time` (15 Nov 2026): `latest = 0.33.0`; 0.31.0 = 2026-09-29T09:05Z, 0.32.0 = 2026-10-12T13:25Z, 0.32.1 = 2026-10-13T09:04Z, 0.33.0 = 2026-11-09T15:28Z.
- `git diff --stat 2579866 4e75896 -- packages/ scripts/ package-lock.json` is empty: perp-core and the lockfile have not changed since the first check.
- The CI `sdk-floor` check in `scripts/checks.mjs` (L208-L218) only reads `packages/ec-core/package.json`, so nothing in CI looks at the perp-core pin. ec-core is also behind, on `^0.30.0`, but it is not perps.

## Environment
SDK version 0.33.0 (@somnia-chain/markets-sdk; the dreamDEX perps page names 0.32.1 as the app's version), bot kit `main` at 4e75896, network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review). First checked 29 Oct 2026, re-checked 15 Nov 2026.
