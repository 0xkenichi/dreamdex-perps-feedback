# dreamDEX Perps Builder Bounty: feedback track

Feedback on dreamDEX perps on the Somnia testnet (Hideki / Tokyo, chain 50383), for the [Perps Builder Bounty](https://dorahacks.io/hackathon/dreamdex-perps-bounty/detail).

Each item is one Markdown file in [`feedback/`](feedback/), in the format the bounty asks for. Every item was checked on 29 Oct 2026 against:

- the docs at app.testnet.dreamdex.io and the Somnia Markets SDK docs site
- the open-source [dreamDEX bot kit](https://github.com/somnia-chain/dreamdex-bot-kit) at commit 2579866
- `@somnia-chain/markets-sdk@0.32.1`, the version the perps developer page says the app runs
- the Hideki RPC and WebSocket, the indexer, and the explorer at explorer.testnet.somnia.network/tokyo

All checks were read-only: no orders, no transactions.

Things checked and found correct, so not reported:

- the network table and contract addresses (all seven hold code, and `getPerpSystemConfig` returns the same addresses)
- the base-decimals table and the market table: tick, minimum size, leverage and paper-USDso status for all 21 markets
- the error selectors quoted on the perps page
- funding settings (8-hour window, hourly settlement on every market)

## Items

| File | Area | Severity | In short |
|---|---|---|---|
| [docs-perp-equity-scoring-sample-0-32-1.md](feedback/docs-perp-equity-scoring-sample-0-32-1.md) | docs | normal | The "account equity for scoring" sample doesn't compile on 0.32.1, and the obvious fix gives a wrong trading gain with no error |
| [docs-perps-methods-missing-in-0-32-1.md](feedback/docs-perps-methods-missing-in-0-32-1.md) | docs | normal | `quotePerpClose` and `getPerpPositionsPage` are documented but not in 0.32.1 |
| [docs-perp-symbol-casing-usdso.md](feedback/docs-perp-symbol-casing-usdso.md) | docs | normal | Docs use `BTC/USDSO:USDSO`; the real symbol is `BTC/USDso:USDso`, so the documented calls throw "unknown symbol" |
| [sdk-perp-stop-operator-registry-missing-from-presets.md](feedback/sdk-perp-stop-operator-registry-missing-from-presets.md) | SDK | normal | First `placePerpStopOrder` on a pool throws `NotConfiguredError` with every bundled address set |
| [botkit-perp-core-sdk-pin-excludes-0-32-1.md](feedback/botkit-perp-core-sdk-pin-excludes-0-32-1.md) | SDK | normal | Bot kit perp-core pins `^0.31.0`, which can never resolve to 0.32.1 |
| [botkit-perp-hideki-uses-shannon-address-preset.md](feedback/botkit-perp-hideki-uses-shannon-address-preset.md) | SDK | normal | Bot kit passes Shannon's address preset on Hideki; on 0.32.1 that points perp discovery at Shannon's factory (shows up once the pin above is bumped) |
| [docs-hideki-indexer-said-not-hosted.md](feedback/docs-hideki-indexer-said-not-hosted.md) | docs | normal | Exchange page says Hideki has no hosted indexer; other pages list one and it answers |
| [docs-errors-page-has-no-perp-errors.md](feedback/docs-errors-page-has-no-perp-errors.md) | docs | normal | Perps page sends developers to an Errors page with no perp errors |
| [docs-api-pages-dont-say-perps-unsupported.md](feedback/docs-api-pages-dont-say-perps-unsupported.md) | docs | normal | HTTP API, WebSocket and CCXT pages don't say perps aren't served there |
| [docs-faq-order-rejected-spot-only.md](feedback/docs-faq-order-rejected-spot-only.md) | docs | normal | "My order was rejected" FAQ has no perp causes |
| [sdk-hideki-chain-missing-block-explorer.md](feedback/sdk-hideki-chain-missing-block-explorer.md) | SDK | normal | `hidekiTestnet` has no `blockExplorers` |
| [contracts-perp-contracts-unverified-on-explorer.md](feedback/contracts-perp-contracts-unverified-on-explorer.md) | contracts | normal | Perp contracts unverified on the explorer; methods show as raw selectors |
| [docs-markets-sdk-broken-links.md](feedback/docs-markets-sdk-broken-links.md) | docs | normal | 13 SDK-docs links go to a private repo (404); one anchor is malformed |

## Contact

Kenichi (Olanrewaju Animashaun) · X [@0xKenichi](https://x.com/0xKenichi) · [0xkenichi.github.io](https://0xkenichi.github.io)
