# SDK exchange page says Hideki has no hosted indexer; other pages list one, and it answers
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/exchange (HTTP 200) and find two passages:
   - The "One exchange, several chains" code: `// Hideki has no hosted indexer. Indexed reads need an indexer you operate.` followed by `{ chain: hidekiTestnet, indexerUrl: hidekiIndexerUrl, addresses: SOMNIA_HIDEKI_TESTNET_ADDRESSES }`.
   - "Reads across chains": "Chains with no indexer. A chain bound with no indexerUrl (Hideki has no hosted indexer) keeps the parts of each read that come from the chain."
2. Open https://somnia-platform-docs.vercel.app/markets/somnia-markets/contracts (HTTP 200), "Networks" table. It lists the Hideki testnet indexer as `https://hideki-dev.smk.somnia.host/v1/graphql`.
3. Open https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/reference/networks-and-endpoints (HTTP 200). It gives the same URL, marked "(development-grade)".
4. Send `POST https://hideki-dev.smk.somnia.host/v1/graphql` with body `{"query":"{__typename}"}`.

## Expected
The pages agree. The exchange page should point Hideki users at `https://hideki-dev.smk.somnia.host/v1/graphql`, with the development-grade caveat, instead of telling them to run their own indexer.

## Actual
The pages contradict each other, and the hosted indexer is up:
- Step 4 returns HTTP 200 with `{"data":{"__typename":"query_root"}}`.
- A `Market(where:{marketType:{_eq:"PERP"}})` query returns Hideki perp markets (BTC, SOL, HYPE, …).

A developer who follows the exchange page leaves Hideki with no `indexerUrl`, or sets up their own. The same page then explains what that costs: for `fetchOrders`, `getOrdersPage`, `getOrderHistoryPage`, `fetchMyTrades`, `fetchOpenStopOrders` and `watchUser`, "The chain is skipped". For perps on Hideki, where the app and Perps Arena run, that means no order or trade history and no account watch, with no error.

The 0.32.1 npm tarball has the same contradiction:
- `docs/EXCHANGE.md` L240 and L374 say there is no hosted indexer.
- `docs/reference/networks-and-endpoints.md` L40 and L49-L54 list `hideki-dev.smk.somnia.host` and say "It indexes the Hideki binary, perp and spot markets".

The docs-site release-notes page also still has older 0.32.0-rc entries saying "Hideki has no hosted indexer. Supply a consumer-operated indexerUrl…".

## Evidence
- Exchange page quotes as in step 1.
- Contracts page, Networks table row: `Hideki testnet | 50383 | https://api.hideki2.infra.testnet.somnia.network | wss://api.hideki2.infra.testnet.somnia.network/ws | https://hideki-dev.smk.somnia.host/v1/graphql`.
- Networks reference: `Hideki testnet (50383) | https://hideki-dev.smk.somnia.host/v1/graphql (development-grade) | SOMNIA_HIDEKI_TESTNET_ADDRESSES`.
- Indexer requests:
  - `curl -s -o /dev/null -w '%{http_code}' https://hideki-dev.smk.somnia.host/v1/graphql` → `200`.
  - The POST in step 4 → `{"data":{"__typename":"query_root"}} HTTP 200`.
- The dreamDEX bot kit uses this indexer by default for Hideki: https://github.com/somnia-chain/dreamdex-bot-kit/blob/25798660bbe4674a6f83a960b202852896e2fed4/packages/perp-core/src/config.ts#L92

## Environment
SDK version 0.32.1 (@somnia-chain/markets-sdk), network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review)
