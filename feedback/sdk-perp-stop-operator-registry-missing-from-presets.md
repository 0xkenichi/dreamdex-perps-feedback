# First placePerpStopOrder on a pool throws NotConfiguredError with SOMNIA_HIDEKI_TESTNET_ADDRESSES (no operatorPermissionsRegistry in any preset)
Severity: normal
Area: SDK

## Steps to reproduce
1. Create `new SomniaMarkets({ chain: hidekiTestnet, indexerUrl: "https://hideki-dev.smk.somnia.host/v1/graphql", addresses: SOMNIA_HIDEKI_TESTNET_ADDRESSES, privateKey })` with SDK 0.32.1.
2. Use an account that has not yet granted the pool's PerpStopOrderRegistry `placeOrderFor`, i.e. its first TP/SL on that market. Call `exchange.trader.placePerpStopOrder({ registry: market.stopRegistry, pool, isBid: false, quantity: 0n, triggerPrice, triggerOperator: 0, stopOrderType: 1 })` without `operatorRegistry`. This is the call shape the Perps guide's "Where the registry address comes from" section describes, and it names only `registry`.

## Expected
The SDK resolves the OperatorPermissionsRegistry itself, as the parameter's doc comment promises: "Resolved from the live config (addresses.operatorPermissionsRegistry) if omitted". Otherwise the Perps guide should say that `operatorRegistry` must be passed and where to find it.

## Actual
`placePerpStopOrder` reads `isOperatorAuthorized`. When that is false, it builds the one-time grant and throws `NotConfiguredError("operatorRegistry or addresses.operatorPermissionsRegistry", "a perp stop order")` before sending anything. `buildPlacePerpStopOrder` throws the same way unless `skipOperatorApproval: true`.

None of the three presets (`SOMNIA_TESTNET_ADDRESSES`, `SOMNIA_MAINNET_ADDRESSES`, `SOMNIA_HIDEKI_TESTNET_ADDRESSES`) has `operatorPermissionsRegistry`. The SDK's own JSDoc says "no deployment manifest carries the key yet". The pool-based discovery `client.getOperatorPermissionsRegistry(pool)` is documented for SpotPools, and the perp stop path does not use it.

The Hideki address is published on the docs contracts page (OperatorPermissionsRegistry `0x6B2FbfeD328FF5F20C0BE7D3C3437613aFFE8D3a`). The dreamDEX bot kit hard-codes it for exactly this reason. The Perps guide never mentions the `operatorRegistry` parameter.

Suggested fix: add `operatorPermissionsRegistry` to the Hideki (and Shannon) presets, or fall back to `getOperatorPermissionsRegistry(pool)`. Also document the parameter in the Perps guide.

## Evidence
- SDK 0.32.1 `dist/perp/stops.js`:
  - L489-L493: `const operatorRegistry = p.operatorRegistry ?? w.addresses().operatorPermissionsRegistry; if (!operatorRegistry) { throw new NotConfiguredError("operatorRegistry or addresses.operatorPermissionsRegistry", "a perp stop order"); }`
  - L574-L582: the grant (and so the throw) runs when `skipOperatorApproval !== true` and `isOperatorAuthorized` is false.
  - L708-L712: the build twin always calls `operatorGrantCall` unless skipped.
- SDK 0.32.1 `dist/trade.d.ts` L1443-L1447 (`PlacePerpStopOrderParams.operatorRegistry`): "Shared OperatorPermissionsRegistry where the one-time global approval is granted. Resolved from the live config (addresses.operatorPermissionsRegistry) if omitted."
- SDK 0.32.1 `dist/addresses.js` L10-L74: none of the three presets has an `operatorPermissionsRegistry` key. `dist/config.d.ts` L88 declares the optional field.
- SDK 0.32.1 `dist/somniaMarketsClient.d.ts` L4829-L4833: "…the grant writes and the two grant reads need that address and otherwise throw NotConfiguredError, and no deployment manifest carries the key yet."
- Docs: https://somnia-platform-docs.vercel.app/markets/somnia-markets/contracts#hideki-testnet (HTTP 200) has the row `OperatorPermissionsRegistry | 0x6B2FbfeD328FF5F20C0BE7D3C3437613aFFE8D3a`. The Perps page (https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/perps, HTTP 200) has no occurrence of `operatorRegistry` or `OperatorPermissionsRegistry`.
- The bot kit works around it: https://github.com/somnia-chain/dreamdex-bot-kit/blob/25798660bbe4674a6f83a960b202852896e2fed4/packages/perp-core/src/config.ts#L72-L86 ("It is NOT in the SDK's bundled address sets … so `placePerpStopOrder` throws `NotConfiguredError` unless it is passed explicitly").

## Environment
SDK version 0.32.1 (@somnia-chain/markets-sdk), network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review)
