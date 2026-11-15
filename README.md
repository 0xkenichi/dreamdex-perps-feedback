# dreamDEX Perps Builder Bounty: feedback track

Feedback on dreamDEX perps on the Somnia testnet (Hideki / Tokyo, chain 50383), for the [Perps Builder Bounty](https://dorahacks.io/hackathon/dreamdex-perps-bounty/detail).

Each item is one Markdown file in [`feedback/`](feedback/), in the format the bounty asks for. Every item was first checked on 29 Oct 2026 and re-checked on 15 Nov 2026 against:

- the docs at app.testnet.dreamdex.io and the Somnia Markets SDK docs site, as served on 15 Nov 2026
- the open-source [dreamDEX bot kit](https://github.com/somnia-chain/dreamdex-bot-kit), `main` at commit 4e75896 (30 Oct 2026; first check at 2579866). perp-core, the lockfile and the CI checks are unchanged between the two
- `@somnia-chain/markets-sdk@0.33.0`, npm `latest` since 9 Nov 2026. The dreamDEX perps developer page still says the app runs 0.32.1, which was the version of the first check; every item below also reproduces on 0.32.1
- the Hideki RPC, the indexer, and the explorer at explorer.testnet.somnia.network/tokyo

All checks were read-only: no wallets, no keys, no orders, no transactions.

Four items from the first check were fixed by 15 Nov 2026 and moved to [`withdrawn/`](withdrawn/). They are not submitted. Two more were partly fixed; their files now describe only what is still wrong.

Things checked on 29 Oct and found correct, so not reported:

- the network table and contract addresses (all seven hold code, and `getPerpSystemConfig` returns the same addresses)
- the base-decimals table and the market table: tick, minimum size, leverage and paper-USDso status for all 21 markets
- the error selectors quoted on the perps page
- funding settings (8-hour window, hourly settlement on every market)

## Items

| File | Area | Severity | In short |
|---|---|---|---|
| [sdk-perp-stop-operator-registry-missing-from-presets.md](feedback/sdk-perp-stop-operator-registry-missing-from-presets.md) | SDK | normal | First `placePerpStopOrder` on a Hideki pool throws `NotConfiguredError`: the Hideki preset still has no `operatorPermissionsRegistry` in 0.33.0 |
| [botkit-perp-core-sdk-pin-excludes-0-32-1.md](feedback/botkit-perp-core-sdk-pin-excludes-0-32-1.md) | SDK | normal | Bot kit perp-core pins `^0.31.0`, which can never resolve to 0.32.1 or 0.33.0 |
| [botkit-perp-hideki-uses-shannon-address-preset.md](feedback/botkit-perp-hideki-uses-shannon-address-preset.md) | SDK | normal | Bot kit passes Shannon's address preset on Hideki; on 0.32.1 and 0.33.0 that points perp discovery at Shannon's factory (shows up once the pin above is bumped) |
| [docs-perp-symbol-casing-usdso.md](feedback/docs-perp-symbol-casing-usdso.md) | SDK | normal | Docs site fixed, but 0.33.0's own doc comments, including the new `exchange.quotePerpClose` example, still use `BTC/USDSO:USDSO`, which throws "unknown symbol" |
| [docs-errors-page-has-no-perp-errors.md](feedback/docs-errors-page-has-no-perp-errors.md) | docs | normal | Perps page sends developers to an Errors page with no perp errors |
| [docs-api-pages-dont-say-perps-unsupported.md](feedback/docs-api-pages-dont-say-perps-unsupported.md) | docs | normal | Developers overview, HTTP API, WebSocket and CCXT pages don't say perps aren't served there |
| [docs-faq-order-rejected-spot-only.md](feedback/docs-faq-order-rejected-spot-only.md) | docs | normal | "My order was rejected" FAQ has no perp causes |
| [contracts-perp-contracts-unverified-on-explorer.md](feedback/contracts-perp-contracts-unverified-on-explorer.md) | contracts | normal | Perp contracts unverified on the explorer; methods show as raw selectors |
| [docs-markets-sdk-broken-links.md](feedback/docs-markets-sdk-broken-links.md) | docs | normal | Two SDK-docs links and the npm `repository` field still go to a private repo (404) |

### Withdrawn (fixed by 15 Nov 2026, not submitted)

| File | Fixed by |
|---|---|
| [docs-perp-equity-scoring-sample-0-32-1.md](withdrawn/docs-perp-equity-scoring-sample-0-32-1.md) | 0.33.0 adds `CreditReclaimed`, block ranges and `blockNumber`; the docs sample compiles |
| [docs-perps-methods-missing-in-0-32-1.md](withdrawn/docs-perps-methods-missing-in-0-32-1.md) | 0.33.0 adds `quotePerpClose` and `getPerpPositionsPage` |
| [docs-hideki-indexer-said-not-hosted.md](withdrawn/docs-hideki-indexer-said-not-hosted.md) | Exchange page now uses the hosted Hideki indexer |
| [sdk-hideki-chain-missing-block-explorer.md](withdrawn/sdk-hideki-chain-missing-block-explorer.md) | 0.33.0 `hidekiTestnet` has `blockExplorers` |

## Contact

Kenichi (Olanrewaju Animashaun) · X [@0xKenichi](https://x.com/0xKenichi) · [0xkenichi.github.io](https://0xkenichi.github.io)
