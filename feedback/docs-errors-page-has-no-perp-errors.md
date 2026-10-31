# Errors page doesn't list perp errors, though the perps page sends developers there
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://app.testnet.dreamdex.io/docs/developers/perps-testnet
2. In "An IOC that fills nothing reverts", follow "The full list of order errors is on the Errors page" (goes to `/docs/developers/contracts/errors#order-rejection`).
3. Search that page for the perp errors the perps page itself names: `MaxPositionSizeExceeded`, `IsolatedMarketBlocked`, `InsufficientCollateral` (and their selectors `0x714b36d6`, `0x3a23d825`).

## Expected
The perp pool / MarginBank errors listed with selectors and meaning, as the SpotPool errors are, so a bot developer can decode a perp revert (for example a `placeOrderFor` that fails with status 0) from the docs.

## Actual
The Errors page covers SpotPool and SpotStopOrderRegistry only. Its sections are: decoding-a-revert, spotpool-errors, order-validation, order-rejection, funding-and-vault, gas, order-lifecycle, batch-and-amend, operators-and-authorization, builder-codes, spotstoporderregistry-errors, preflight-http-api-reverts. The word "perp" does not appear on it. `MaxPositionSizeExceeded`, `IsolatedMarketBlocked`, `InsufficientCollateral` and the paper-USDso (voucher) errors are absent; only `ImmediateOrCancelNoFill` and `AmendReplacementRejected` appear, because SpotPool shares them.

## Evidence
- Perps page text: "The full list of order errors is on the Errors page." Its pitfalls list names `MaxPositionSizeExceeded() (0x714b36d6)`, `IsolatedMarketBlocked` and `InsufficientCollateral() (0x3a23d825)`.
- Errors page, checked 29 Oct 2026: 0 matches for "perp", "Perp", "MarginBank", "voucher", "MaxPositionSizeExceeded", "IsolatedMarketBlocked", "InsufficientCollateral", "0x714b36d6", "0x3a23d825".
- The selectors on the perps page are correct (`keccak256` checked with viem `toFunctionSelector`), so the gap is only on the Errors page.

## Environment
Docs at app.testnet.dreamdex.io (29 Oct 2026), SDK 0.32.1, Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs check)
