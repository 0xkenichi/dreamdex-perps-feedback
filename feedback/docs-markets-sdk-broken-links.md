# Markets SDK docs still link to a private GitHub repo (HTTP 404) for SDK-API-002 and the canary release process, and the npm package points there too
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript (HTTP 200).
2. Click "SDK-API-002" in the Install section ("These extra subpaths are existing migration debt against SDK-API-002").
3. Click "canary section of the release process" in the same section.
4. Open https://www.npmjs.com/package/@somnia-chain/markets-sdk (or run `npm view @somnia-chain/markets-sdk@0.33.0 repository`) and follow the Repository link.

## Expected
Links open the referenced page or section.

## Actual
Steps 2-4 go to `github.com/somnia-chain/somnia-markets/...`. That repo returns 404 to anyone outside the org:
- `https://api.github.com/repos/somnia-chain/somnia-markets` → 404

So the rule a reader is pointed to (SDK-API-002) and the canary release process cannot be read, and the npm package's `repository` field in 0.33.0 (`git+https://github.com/somnia-chain/somnia-markets.git`, directory `packages/sdk`) still points at the same 404 repo.

What changed since the first report (29 Oct 2026): the "Edit this page" links to the private `somnia-markets` and `somnia-platform` repos are gone from every Markets page, and the exchange page's "ENGINE.md § Errors" link now uses the correct anchor `#errors-branch-on-the-type-never-the-message`. Only the two links above and the npm `repository` field are still broken.

## Evidence
Both links were requested with `curl -L` on 15 Nov 2026 and returned HTTP 404 (also on 29 Oct 2026):

| Page | Link text | URL |
| --- | --- | --- |
| /typescript | SDK-API-002 | https://github.com/somnia-chain/somnia-markets/blob/main/packages/sdk/RULES.md#sdk-api-002--the-package-has-two-approved-entrypoints |
| /typescript | canary section of the release process | https://github.com/somnia-chain/somnia-markets/blob/main/packages/sdk/RELEASE_PROCESS.md#3a-canary-publication-from-main |

- `@somnia-chain/markets-sdk@0.33.0` `package.json`: `"repository": { "type": "git", "url": "git+https://github.com/somnia-chain/somnia-markets.git", "directory": "packages/sdk" }`.
- Link check, 15 Nov 2026: all 302 unique links on 11 Markets pages (`/markets`, `/markets/somnia-markets`, `/contracts`, `/market-making`, and `/typescript`, `/perps`, `/exchange`, `/spot`, `/binary`, `/architecture`, `/release-notes` under `/typescript`) were requested. No "Edit this page" link appears on these pages or on `/engine`, `/reference/networks-and-endpoints`, `/reference/symbols` and `/migrating`. Only the two links above returned 404. The `npmjs.com` links returned 403, which is bot protection rather than a broken link, so they are not included here.

## Environment
SDK version 0.33.0 (@somnia-chain/markets-sdk), docs site somnia-platform-docs.vercel.app, network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review). First checked 29 Oct 2026, re-checked 15 Nov 2026.
