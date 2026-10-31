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
- The pool, its implementation, MarginBank `0x478769E1177C0773c43A593DCCc4Ad15F15F1394` and USDso `0xed7993A74dAe1e6656B704A424DcD8711aAf4F72` all report `is_verified: false`, and `implementations: []` (the beacon proxy is not detected).
- On the BTC-PERP pool, most recent transactions show raw selectors instead of names: `0x80054449`, `0xb5e6cdfd`, `0xe37b444b`. Only `placeOrders` and `cancelOrders` decode. Some of these are FAILED transactions developers would want to inspect.
- "Read contract" has no ABI to read with.

## Evidence
- Explorer API: `GET https://explorer.testnet.somnia.network/tokyo/api/v2/addresses/<address>` returns `{"is_verified": false, "is_contract": true, "implementations": []}` for each of the four addresses above (29 Oct 2026).
- Pool page Transactions tab, 29 Oct 2026 ~03:15 UTC: rows with method `0x80054449` (including a FAILED one), `0xb5e6cdfd`, `0xe37b444b`.
- SDK comment in `dist/contractErrorsAbi.js` says the perp error ABIs come "from the indexer's committed ABIs because the deployed perp contracts live on unmerged protocol branches", which is consistent with no public verified source.

## Environment
Explorer explorer.testnet.somnia.network/tokyo, Somnia Hideki/Tokyo testnet (chain 50383), SDK 0.32.1, wallet address: n/a (read-only check)
