# Developers overview, HTTP API, WebSocket and CCXT pages don't say perps aren't available there
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://app.testnet.dreamdex.io/docs/developers (Developers > Overview). It presents dreamBot Builder, Contracts, HTTP API, WebSocket API and Libraries (CCXT) as "everything you need to integrate with dreamDEX programmatically", and says "existing CCXT bots work with minimal changes".
2. Open https://app.testnet.dreamdex.io/docs/developers/http-api . The Base URLs table lists Mainnet (chain 5031, `https://api.dreamdex.io/v0`) and Testnet "Somnia Shannon" (chain 50312, `https://stg.api.dreamdex.io/v0`).
3. Call `GET https://stg.api.dreamdex.io/v0/markets` and `GET https://api.dreamdex.io/v0/markets`.

## Expected
Since perps run on the Hideki testnet (chain 50383) and are reached only through `@somnia-chain/markets-sdk` and the indexer, the Developers overview and the HTTP API, WebSocket and CCXT pages should say so in one line ("Perps are not served by the HTTP/WebSocket API or CCXT yet; see Perps on Testnet for developers"). Bot builders arriving from the API pages, or with an existing CCXT perps bot, then know where to go.

## Actual
- `stg.api.dreamdex.io/v0/markets` returns 3 spot markets (`WBTC:USDso`, `WETH:USDso`, `SOMI:USDso`); mainnet returns 4 spot markets (`USDC.e:USDso`, `WBTC:USDso`, `WETH:USDso`, `SOMI:USDso`). No perp markets on either (15 Nov 2026).
- The HTTP API base-URL table has no row for chain 50383, where the perps developer page says perps run.
- The Developers overview, HTTP API overview, Market Data, Trading, WebSocket overview, Real-Time Feed, Libraries, CCXT, Contracts overview and Contract Addresses pages contain no mention of perps beyond the sidebar navigation. The perps developer page only implies the SDK route ("Build against it with @somnia-chain/markets-sdk").
- Since the first check, Contract Addresses has gained a section "Testnet (Somnia Tokyo Testnet, chain ID 50383)". It lists spot tokens, spot pools and the SpotPoolRegistry for that chain, but no perp contracts and no link to the perps page, so a developer who looks up chain 50383 there still sees no sign of perps.

## Evidence
- API responses, 29 Oct 2026 and 15 Nov 2026: testnet `/v0/markets` → 3 items, none containing "PERP"; mainnet → 4 items, none containing "PERP".
- Page text checks, 29 Oct 2026 and 15 Nov 2026: on each page listed above, "perp" occurs only in the shared sidebar and footer, never in the page body. Pages checked on 15 Nov: `/docs/developers`, `/docs/developers/http-api`, `/http-api/market-data`, `/http-api/trading`, `/websocket-api`, `/websocket-api/real-time-feed`, `/libraries`, `/libraries/ccxt`, `/contracts`, `/contracts/contract-addresses`.
- HTTP API overview, Base URLs table: "Testnet (Somnia Shannon) 50312 https://stg.api.dreamdex.io/v0". Perps developer page, Network table: "Chain ID 50383".

## Environment
Docs at app.testnet.dreamdex.io (first checked 29 Oct 2026, re-checked 15 Nov 2026), HTTP API stg.api.dreamdex.io/v0 and api.dreamdex.io/v0, SDK 0.33.0 (the dreamDEX perps page names 0.32.1 as the app's version), Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs and public API check)
