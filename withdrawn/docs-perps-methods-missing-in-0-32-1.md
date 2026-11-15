# Perps docs document quotePerpClose and getPerpPositionsPage, which SDK 0.32.1 does not have
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/perps (HTTP 200).
2. "Closing when the book is thin" says "client.quotePerpClose tells you how much that is before you send." It gives:
   ```ts
   const q = await client.quotePerpClose({ pool, marginBank, account });
   if (q.priceable && q.fillableSize < q.quantity) { … q.bandToFillAll … q.oppositeSideEmpty … }
   ```
   The same section adds: "On the exchange surface, exchange.quotePerpClose(symbol) quotes the connected wallet's position."
3. "Every account at once" says: "client.getPerpPositionsPage({ pool, minSize, includeFlat, cursor }) reads all accounts' positions with entryFundingIndex … Pass nextCursor back as cursor until it is null."
4. `npm i @somnia-chain/markets-sdk@0.32.1` (npm `latest`, and the version the dreamDEX docs say the app runs), paste the step 2 snippet and run `tsc`.

## Expected
Documented methods exist in the released SDK. If they are unreleased, the page says which version adds them.

## Actual
None of the three members exists in 0.32.1:
- `SomniaMarketsClient.quotePerpClose`
- `SomniaMarkets.quotePerpClose`
- `SomniaMarketsClient.getPerpPositionsPage`

The snippet fails to compile with "Property 'quotePerpClose' does not exist on type 'SomniaMarketsClient'". In plain JavaScript it fails at runtime with "client.quotePerpClose is not a function".

The docs site's own release-notes page lists 0.32.1 as the newest release, and does not mention these methods. They are also absent from the newest npm canary, `0.32.2-canary.38575926931.ge4550933d36d` (14 Oct 2026).

The nearest 0.32.1 calls do something different:
- `client.previewPerpClosePnl(...)` has no book-walk or band fields.
- `client.listPerpPositions(account, …)` reads one account, not every account with a cursor.

## Evidence
- `grep -rn "quotePerpClose\|getPerpPositionsPage" dist/ docs/ src/` over the unpacked `@somnia-chain/markets-sdk@0.32.1` tarball → no matches. The tarball's own `docs/PERPS.md` has neither name.
- The client declares related reads but not these: `dist/somniaMarketsClient.d.ts` L3544 `previewPerpClosePnl(p: { pool; marginBank; account; quantity?; price?; asMaker? }): Promise<PerpClosePreview>;` and L2743 `listPerpPositions(account: string, opts?: { pool?; includeFlat?; limit?; offset? })`.
- Canary check: `GET https://unpkg.com/@somnia-chain/markets-sdk@0.32.2-canary.38575926931.ge4550933d36d/dist/somniaMarketsClient.d.ts` → HTTP 200, 538,735 bytes, no `quotePerpClose` or `getPerpPositionsPage`.
- Release notes: https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/release-notes (HTTP 200). The first heading is `## 0.32.1`, and neither name appears on the page.
- I could not check for an existing report. The SDK repo the docs link to returns 404 (`https://api.github.com/repos/somnia-chain/somnia-markets` → 404).

## Environment
SDK version 0.32.1 (@somnia-chain/markets-sdk), network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review)

## Withdrawn: fixed (re-checked 15 Nov 2026)
Not submitted. `@somnia-chain/markets-sdk@0.33.0` adds `SomniaMarketsClient.quotePerpClose`, `SomniaMarkets.quotePerpClose` and `SomniaMarketsClient.getPerpPositionsPage` (`dist/somniaMarketsClient.d.ts` L3755 and L2904, `dist/unified/exchange.d.ts` L2490). The documented snippets compile with `tsc --strict` against 0.33.0, and the release notes list both methods under 0.33.0. Note: the dreamDEX perps developer page still says the app runs 0.32.1, where they are missing.
