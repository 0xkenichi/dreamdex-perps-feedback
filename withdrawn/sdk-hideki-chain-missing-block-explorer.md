# hidekiTestnet chain definition has no blockExplorers
Severity: normal
Area: SDK

## Steps to reproduce
1. Install `@somnia-chain/markets-sdk@0.32.1` (the version the perps developer page says the app runs).
2. `import { hidekiTestnet } from "@somnia-chain/markets-sdk/chains"`
3. `console.log(hidekiTestnet.blockExplorers)`

## Expected
An explorer entry like the other Somnia chains in the same file, pointing at the explorer the docs give for perps testnet (`https://explorer.testnet.somnia.network/tokyo`). Tools that read `chain.blockExplorers` (viem helpers, wagmi, RainbowKit, wallet "view on explorer" links) then work for perps builders.

## Actual
`undefined`. In `dist/chains/index.d.ts`, the mainnet chain (id 5031) and Somnia Testnet (id 50312) both declare a `blockExplorers` object, but Hideki Testnet (id 50383) only has an optional, unset one. Bots and apps built on `hidekiTestnet` get no explorer links for perp transactions.

## Evidence
- Runtime output with 0.32.1: `chain 50383 {"default":{"http":["https://api.hideki2.infra.testnet.somnia.network"],"webSocket":["wss://api.hideki2.infra.testnet.somnia.network/ws"]}} undefined` (last value is `hidekiTestnet.blockExplorers`).
- `dist/chains/index.d.ts` in the published 0.32.1 package: mainnet (`id: 5031`, line 53) declares `blockExplorers: { ... name: "Somnia Explorer" }` (lines 38–40) and Somnia Testnet (`id: 50312`, line 102) declares `name: "Somnia Testnet Explorer"` (lines 87–89). Hideki Testnet (`id: 50383; name: "Hideki Testnet"`, line 213) only has the optional `blockExplorers?:` type (line 193), and the runtime value is unset.
- Docs list the explorer for this chain: https://app.testnet.dreamdex.io/docs/developers/perps-testnet (Network table, "Explorer explorer.testnet.somnia.network/tokyo"). That URL loads and shows the BTC-PERP pool `0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0`.

## Environment
SDK 0.32.1 (@somnia-chain/markets-sdk), Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (read-only check)

## Withdrawn: fixed (re-checked 15 Nov 2026)
Not submitted. With `@somnia-chain/markets-sdk@0.33.0`, `hidekiTestnet.blockExplorers` is `{"default":{"name":"Somnia Explorer","url":"https://explorer.testnet.somnia.network/tokyo"}}` at runtime.
