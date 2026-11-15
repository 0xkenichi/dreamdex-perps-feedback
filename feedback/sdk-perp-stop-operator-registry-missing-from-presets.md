# First placePerpStopOrder on a pool throws NotConfiguredError with SOMNIA_HIDEKI_TESTNET_ADDRESSES (the Hideki preset has no operatorPermissionsRegistry)
Severity: normal
Area: SDK

## Steps to reproduce
1. Create `new SomniaMarkets({ chain: hidekiTestnet, indexerUrl: "https://hideki-dev.smk.somnia.host/v1/graphql", addresses: SOMNIA_HIDEKI_TESTNET_ADDRESSES, privateKey })` with SDK 0.33.0 (first seen on 0.32.1).
2. Use an account that has not yet granted the pool's PerpStopOrderRegistry `placeOrderFor`, i.e. its first TP/SL on that market. (In 0.33.0 the grant is per pool, so this is the first TP/SL on every market the account trades.) Call `exchange.trader.placePerpStopOrder({ registry: market.stopRegistry, pool, isBid: false, quantity: 0n, triggerPrice, triggerOperator: 0, stopOrderType: 1 })` without `operatorRegistry`. This is the call shape the Perps guide's "Where the registry address comes from" section describes, and it names only `registry` and `pool`.

## Expected
The SDK resolves the OperatorPermissionsRegistry itself, as the parameter's doc comment promises: "Resolved from the live config (addresses.operatorPermissionsRegistry) if omitted". Otherwise the Perps guide should say that `operatorRegistry` must be passed and where to find it.

## Actual
`placePerpStopOrder` checks the registry against the factory, then reads `isOperatorAuthorized`. When that is false, it builds the grant (in 0.33.0 a per-pool `setOperatorApprovalForPool`) and throws `NotConfiguredError("operatorRegistry or addresses.operatorPermissionsRegistry", "a perp stop order")` before sending anything. `buildPlacePerpStopOrder` throws the same way unless `skipOperatorApproval: true`.

Re-checked on 15 Nov 2026 with 0.33.0: `SOMNIA_TESTNET_ADDRESSES`, `SOMNIA_MAINNET_ADDRESSES` and `SOMNIA_HIDEKI_TESTNET_ADDRESSES` still have no `operatorPermissionsRegistry`. The new `SOMNIA_TOKYO_ADDRESSES` (chain 50321) does carry one, which shows the fix is only a preset entry, but Hideki, where perps run, did not get it. The SDK's own JSDoc still says "no deployment manifest carries the key yet". The pool-based discovery `client.getOperatorPermissionsRegistry(pool)` is documented for SpotPools, and the perp stop path does not use it.

The Hideki address is published on the docs contracts page (OperatorPermissionsRegistry `0x6B2FbfeD328FF5F20C0BE7D3C3437613aFFE8D3a`). The dreamDEX bot kit hard-codes it for exactly this reason. The Perps guide still never mentions the `operatorRegistry` parameter.

Suggested fix: add `operatorPermissionsRegistry` to the Hideki (and Shannon) presets, or fall back to `getOperatorPermissionsRegistry(pool)`. Also document the parameter in the Perps guide.

## Evidence
- SDK 0.33.0 `dist/perp/stops.js` (0.32.1: L489-L493, L574-L582, L708-L712):
  - L496-L499: `const operatorRegistry = p.operatorRegistry ?? w.addresses().operatorPermissionsRegistry; if (!operatorRegistry) { throw new NotConfiguredError("operatorRegistry or addresses.operatorPermissionsRegistry", "a perp stop order"); }`
  - L600-L611: the grant (and so the throw) runs when `skipOperatorApproval !== true` and `isOperatorAuthorized` is false.
  - L737-L744: the build twin always calls `operatorGrantCall` unless skipped.
- SDK 0.33.0 `dist/trade.d.ts` L1503-L1507 (`PlacePerpStopOrderParams.operatorRegistry`): "Shared OperatorPermissionsRegistry where the one-time approval for `pool` is granted. Resolved from the live config (addresses.operatorPermissionsRegistry) if omitted." (0.32.1: L1443-L1447, same promise.)
- SDK 0.33.0 `dist/addresses.js`: `SOMNIA_TESTNET_ADDRESSES` (L10-L27), `SOMNIA_MAINNET_ADDRESSES` and `SOMNIA_HIDEKI_TESTNET_ADDRESSES` (L59-L74) have no `operatorPermissionsRegistry` key. Only `SOMNIA_TOKYO_ADDRESSES` has one, L92: `operatorPermissionsRegistry: "0xeC0066de3781BaC11E97490DD0E61AcbA9Fb3fD6"`. Runtime check: `"operatorPermissionsRegistry" in SOMNIA_HIDEKI_TESTNET_ADDRESSES` → `false`.
- SDK 0.33.0 `dist/somniaMarketsClient.d.ts` L5068-L5071 (0.32.1: L4829-L4833): "…the grant writes and the two grant reads need that address and otherwise throw NotConfiguredError, and no deployment manifest carries the key yet."
- Docs: https://somnia-platform-docs.vercel.app/markets/somnia-markets/contracts#hideki-testnet (HTTP 200) has the row `OperatorPermissionsRegistry | 0x6B2FbfeD328FF5F20C0BE7D3C3437613aFFE8D3a`. The Perps page (https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/perps, HTTP 200) has no occurrence of `operatorRegistry` or `OperatorPermissionsRegistry` (29 Oct and 15 Nov 2026). The dreamDEX perps developer page lists the same Hideki address in its Contracts table but does not say the SDK needs it passed.
- The bot kit works around it: https://github.com/somnia-chain/dreamdex-bot-kit/blob/4e75896896e024c03e2033d5920514f249998cd6/packages/perp-core/src/config.ts#L72-L86 ("It is NOT in the SDK's bundled address sets … so `placePerpStopOrder` throws `NotConfiguredError` unless it is passed explicitly").

## Environment
SDK version 0.33.0 (@somnia-chain/markets-sdk; also present in 0.32.1, which the dreamDEX perps page names as the app's version), network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review). First checked 29 Oct 2026, re-checked 15 Nov 2026.
