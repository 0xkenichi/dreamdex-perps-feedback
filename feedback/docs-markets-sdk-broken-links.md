# Markets SDK docs link to private GitHub repos (HTTP 404), and one cross-page anchor is wrong
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript (HTTP 200).
2. Click "SDK-API-002" in the Install section ("These extra subpaths are existing migration debt against SDK-API-002").
3. Click "canary section of the release process" in the same section.
4. On any Markets page, including the Perps guide (https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/perps), click "Edit this page".
5. On https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/exchange, under createOrder params, click "ENGINE.md § Errors".

## Expected
Links open the referenced page or section.

## Actual
Steps 2-4 go to `github.com/somnia-chain/somnia-markets/...` or `github.com/somnia-chain/somnia-platform/...`. Both repos return 404 to anyone outside the org:
- `https://api.github.com/repos/somnia-chain/somnia-markets` → 404
- `https://api.github.com/repos/somnia-chain/somnia-platform` → 404

So the rule a reader is pointed to (SDK-API-002) and the canary release process cannot be read.

The npm package's `repository` field (`git+https://github.com/somnia-chain/somnia-markets.git`, directory `packages/sdk`) points at the same 404 repo.

Step 5 links to `/markets/somnia-markets/typescript/engine#errors--branch-on-the-type-never-the-message`, with a double hyphen. The target heading's id is `errors-branch-on-the-type-never-the-message`, so the browser loads the page and does not scroll to the section.

## Evidence
Every link below was requested with `curl -L` on 29 Oct 2026, and every one returned HTTP 404:

| Page | Link text | URL |
| --- | --- | --- |
| /typescript | SDK-API-002 | https://github.com/somnia-chain/somnia-markets/blob/main/packages/sdk/RULES.md#sdk-api-002--the-package-has-two-approved-entrypoints |
| /typescript | canary section of the release process | https://github.com/somnia-chain/somnia-markets/blob/main/packages/sdk/RELEASE_PROCESS.md#3a-canary-publication-from-main |
| /typescript | Edit this page | https://github.com/somnia-chain/somnia-markets/blob/main/packages/sdk/README.md |
| /typescript/perps | Edit this page | https://github.com/somnia-chain/somnia-markets/blob/main/packages/sdk/docs/PERPS.md |
| /typescript/exchange | Edit this page | https://github.com/somnia-chain/somnia-markets/blob/main/packages/sdk/docs/EXCHANGE.md |
| /typescript/spot | Edit this page | …/packages/sdk/docs/SPOT.md |
| /typescript/binary | Edit this page | …/packages/sdk/docs/BINARY.md |
| /typescript/architecture | Edit this page | …/packages/sdk/docs/ARCHITECTURE.md |
| /typescript/release-notes | Edit this page | …/packages/sdk/CHANGELOG.md |
| /markets/somnia-markets | Edit this page | https://github.com/somnia-chain/somnia-markets/blob/main/platform/docs/index.md |
| /contracts | Edit this page | …/platform/docs/contracts.md |
| /market-making | Edit this page | …/platform/docs/market-making.md |
| /markets | Edit this page | https://github.com/somnia-chain/somnia-platform/blob/main/apps/docs/content/markets/index.md |

- The engine page source has `<h2 id="errors-branch-on-the-type-never-the-message">Errors — branch on the type, never the message`. The exchange page's link is `href="/markets/somnia-markets/typescript/engine#errors--branch-on-the-type-never-the-message"`.
- The other 70 internal and external links on the 10 Markets pages I fetched returned 200. The two `npmjs.com` links returned 403, which is bot protection rather than a broken link, so they are not included here.

## Environment
SDK version 0.32.1 (@somnia-chain/markets-sdk), network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review)
