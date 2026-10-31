# perp-core passes Shannon's SOMNIA_TESTNET_ADDRESSES on Hideki; on SDK 0.32.1 that sends perp discovery to Shannon's PerpPoolFactory
Severity: normal
Area: SDK

## Steps to reproduce
1. In dreamdex-bot-kit `main`, use the default `NETWORK=hideki`. `packages/perp-core/src/config.ts` then binds chain 50383 with the Hideki RPC and indexer.
2. Note that `packages/perp-core/src/exchange.ts` builds `new SomniaMarkets({ ..., addresses: SOMNIA_TESTNET_ADDRESSES })` for every network. The comment above it says "The bundled testnet address set serves Hideki too".
3. Move perp-core to SDK 0.32.1, the version the app runs (see `botkit-perp-core-sdk-pin-excludes-0-32-1.md`), and run `npm run perp:doctor` or any perp-* bot with `NETWORK=hideki`.

## Expected
A Hideki binding uses `SOMNIA_HIDEKI_TESTNET_ADDRESSES`, which 0.32.1 exports for chain 50383. The SDK networks reference (https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/reference/networks-and-endpoints) pairs Hideki with that constant.

## Actual
In 0.32.1, `SOMNIA_TESTNET_ADDRESSES` carries three Shannon-only perp keys that 0.31.0 did not have:

| Key | Shannon value | Hideki value |
| --- | --- | --- |
| `perpPoolFactory` | `0x516d…a07b` | `0xE631…6560` |
| `marginBank` | `0xdd4A…5C4E` | `0x4787…1394` |
| `priceFeedScheduler` | `0x7ccA…D32B` | `0x1998…1D7F` |

The SDK uses a configured `perpPoolFactory` in preference to the one the bank reports. So on chain 50383, `loadMarkets()` reads the PerpPoolFactory at Shannon's address. The Shannon `marginBank` hint and `priceFeedScheduler` also reach the Hideki engine.

What follows is from reading the source; I did not run it. If that factory read fails, the SDK contains the error in `perpDiscoveryError`, and every indexed perp row is returned with `active: true`. perp-core's halted-market filter relies on `m.active`, so it would stop filtering restricted or unregistered pools. New pools that the indexer has not picked up yet would also not be added from the chain.

The comment was true for 0.31.0. That release's `SOMNIA_TESTNET_ADDRESSES` had none of these three keys, and there was no Hideki preset. It is false for the version the app runs.

Suggested fix: pick the preset by network, e.g. `hideki → SOMNIA_HIDEKI_TESTNET_ADDRESSES`.

No GitHub issue or branch covers this (checked 29 Oct 2026). I did not do any on-chain checks of what is deployed at the Shannon factory address on chain 50383.

## Evidence
- Bot kit:
  - https://github.com/somnia-chain/dreamdex-bot-kit/blob/25798660bbe4674a6f83a960b202852896e2fed4/packages/perp-core/src/exchange.ts#L37-L57: `The bundled testnet address set serves Hideki too` … `addresses: SOMNIA_TESTNET_ADDRESSES,`
  - https://github.com/somnia-chain/dreamdex-bot-kit/blob/25798660bbe4674a6f83a960b202852896e2fed4/packages/perp-core/src/config.ts#L88-L94: `hideki: { rpc: "https://api.hideki2.infra.testnet.somnia.network", … chainId: 50383 }`
  - https://github.com/somnia-chain/dreamdex-bot-kit/blob/25798660bbe4674a6f83a960b202852896e2fed4/packages/perp-core/src/markets.ts#L94: `.filter((m) => m.type === "swap" && (opts.includeHalted || m.active))`
- SDK 0.32.1 `dist/addresses.js`:
  - `SOMNIA_TESTNET_ADDRESSES`: L18 `marginBank: "0xdd4A14A2763FDa39b9759D2D4150DB0e0f085C4E"`, L23 `perpPoolFactory: "0x516d2cb6de41bE98444e887793946E65989Ba07b"`, L24 `priceFeedScheduler: "0x7ccAE31E45693475be1e4BAa2360D6997eB2D32B"`.
  - `SOMNIA_HIDEKI_TESTNET_ADDRESSES`: L66 `marginBank: "0x478769E1177C0773c43A593DCCc4Ad15F15F1394"`, L71 `perpPoolFactory: "0xE631fAEE547baeE07353E290f0562ea67d856560"`, L72 `priceFeedScheduler: "0x1998C88D54a240b8C671493f27f58eC416b81D7F"`.
  - Public copy: https://unpkg.com/@somnia-chain/markets-sdk@0.32.1/dist/addresses.js (HTTP 200).
- SDK 0.31.0 `dist/addresses.js` (https://unpkg.com/@somnia-chain/markets-sdk@0.31.0/dist/addresses.js, HTTP 200): `SOMNIA_TESTNET_ADDRESSES` has no `perpPoolFactory`, `marginBank` or `priceFeedScheduler`, and no Hideki constant exists.
- SDK 0.32.1 `dist/config.d.ts` L161-L170: "When set it takes precedence … `SOMNIA_TESTNET_ADDRESSES` and `SOMNIA_HIDEKI_TESTNET_ADDRESSES` set it … these presets do not use the bank's factory."
- SDK 0.32.1 `dist/unified/exchange.js`:
  - L1212: `const factory = engine.config.addresses?.perpPoolFactory;`
  - L1384: `let factory = engine.config.addresses?.perpPoolFactory;`
  - L1560: `active: status ? !status.restricted && status.registered !== false : true,`
- SDK 0.32.1 `dist/createClient.js`:
  - L1181-L1182: `listPerpPoolStatuses` / `listTradeablePerpPools` use `config.addresses?.marginBank`.
  - L851-L853: `priceFeed: config.addresses.priceFeedScheduler`.

## Environment
SDK version 0.32.1 (@somnia-chain/markets-sdk), network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review)
