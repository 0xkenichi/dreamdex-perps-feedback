# Perps developer page says the app runs SDK 0.32.1, but npm `latest` has been 0.33.0 since 9 Nov
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://app.testnet.dreamdex.io/docs/developers/perps-testnet
2. Read the intro: "Build against it with @somnia-chain/markets-sdk; this page uses version 0.32.1, the one the app runs."
3. Run `npm view @somnia-chain/markets-sdk version`. It returns `0.33.0`, published 9 Nov 2026.
4. Install the SDK the usual way (`npm install @somnia-chain/markets-sdk`, no version given). You get 0.33.0, not the version the page documents.

## Expected
Either the page is updated for 0.33.0, or it says to pin `@somnia-chain/markets-sdk@0.32.1` and whether 0.33.0 is supported for perps on Hideki. A developer should know which version their bot should run against the live app.

## Actual
The page names 0.32.1 as the app's version and gives no pinned install command. It doesn't mention 0.33.0. 0.33.0 changes the perps API surface the page describes. For example, it adds `quotePerpClose`, `getPerpPositionsPage`, `sinceBlock`/`toBlock` on the margin reads and `CreditReclaimed`. So a developer on default `latest` reads docs for a different version than the one they installed, and can't tell which one the app expects.

## Evidence
- Perps page text, 15 Nov 2026 08:06 UTC: "this page uses version 0.32.1, the one the app runs." There is no `npm install` command on the page and no mention of 0.33.
- npm registry, 15 Nov 2026: `latest` = 0.33.0 (published 2026-11-09).

## Environment
Docs at app.testnet.dreamdex.io (checked 15 Nov 2026), npm registry, `@somnia-chain/markets-sdk` 0.32.1 / 0.33.0, Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs check)
