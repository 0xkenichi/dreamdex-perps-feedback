# Perp contracts are unverified on the explorer, so perp transactions show raw selectors
Severity: normal
Area: contracts

## Steps to reproduce
1. Open the BTC-PERP pool on the explorer the docs give: https://explorer.testnet.somnia.network/tokyo/address/0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0?tab=contract
2. Note "NOT VERIFIED". The bytecode is a beacon proxy; reading `implementation()` (`0x5c60da1b`) on the beacon `0xb787f7d2f87d32c1505c110c4457ed36a3f6b002` returns `0x81f9eb88c95a52b4d3ea47a73ac394b8a547d443`.
3. Open the implementation: https://explorer.testnet.somnia.network/tokyo/address/0x81f9eb88c95a52b4d3ea47a73ac394b8a547d443?tab=contract , also "NOT VERIFIED".
4. Go back to the pool's Transactions tab.

## Expected
Verified source (or at least a verified ABI) for the perp pool implementation, MarginBank and USDso, and the pools detected as proxies. Developers can then read the contract on the explorer, see decoded method names and debug their own failed `placeOrderFor` transactions there.

## Actual
- The pool, its implementation, MarginBank `0x478769E1177C0773c43A593DCCc4Ad15F15F1394`, USDso `0xed7993A74dAe1e6656B704A424DcD8711aAf4F72` and PerpPoolFactory `0xE631fAEE547baeE07353E290f0562ea67d856560` all report `is_verified: false`, and `implementations: []` (the beacon proxy is not detected). Unchanged on 15 Nov 2026.
- On the BTC-PERP pool, recent transactions show raw selectors instead of names. On 15 Nov 2026, 21 of the latest 50 were `0xb5e6cdfd` (11) and `0xe37b444b` (10); only batch calls such as `amendOrders`, `placeOrders` and `cancelOrders` decode. On 29 Oct, `0x80054449` also appeared, including FAILED transactions developers would want to inspect.
- "Read contract" has no ABI to read with.

## Evidence
- Explorer API: `GET https://explorer.testnet.somnia.network/tokyo/api/v2/addresses/<address>` returns `{"is_verified": false, "is_contract": true, "implementations": []}` for each of the five addresses above (29 Oct 2026, and again on 15 Nov 2026).
- Read-only `eth_call` of `implementation()` (`0x5c60da1b`) on the beacon `0xb787f7d2f87d32c1505c110c4457ed36a3f6b002` via `https://api.hideki2.infra.testnet.somnia.network` still returns `0x81f9eb88c95a52b4d3ea47a73ac394b8a547d443` (15 Nov 2026).
- Pool page Transactions tab, 29 Oct 2026 ~03:15 UTC: rows with method `0x80054449` (including a FAILED one), `0xb5e6cdfd`, `0xe37b444b`.
- `GET .../api/v2/addresses/0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0/transactions`, 15 Nov 2026 ~07:58 UTC: of 50 rows, 29 `amendOrders`, 11 `0xb5e6cdfd`, 10 `0xe37b444b`.
- SDK comment in `dist/contractErrorsAbi.js` (0.32.1 and 0.33.0, L4-L5) says the perp error ABIs come "from the indexer's committed ABIs because the deployed perp contracts live on unmerged protocol branches", which is consistent with no public verified source.

## Environment
Explorer explorer.testnet.somnia.network/tokyo, Somnia Hideki/Tokyo testnet (chain 50383), SDK 0.33.0 (the dreamDEX perps page names 0.32.1 as the app's version), wallet address: n/a (read-only check). First checked 29 Oct 2026, re-checked 15 Nov 2026.
