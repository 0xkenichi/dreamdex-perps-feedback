# SDK 0.33.0 doc comments still use perp symbol "BTC/USDSO:USDSO", including the new exchange.quotePerpClose example; symbol lookup is case-sensitive, so the copied call throws
Severity: normal
Area: SDK

## Steps to reproduce
1. `npm i @somnia-chain/markets-sdk@0.33.0` (npm `latest`, 9 Nov 2026).
2. In an editor, hover `exchange.quotePerpClose` or `exchange.fetchFundingRateHistory`, or open `dist/unified/exchange.d.ts`. The examples are:
   ```ts
   const q = await exchange.quotePerpClose("BTC/USDSO:USDSO");
   const page = await exchange.fetchFundingRateHistory("BTC/USDSO:USDSO", since, 100);
   ```
3. Run either example against Hideki after `await exchange.loadMarkets()`.

## Expected
The examples that ship with the SDK use the symbol the SDK actually builds, `BTC/USDso:USDso`, as the docs site now does.

## Actual
From the source, both calls throw `InvalidInputError: unknown symbol BTC/USDSO:USDSO — call loadMarkets() first`.

- **How the SDK builds the symbol.** A perp symbol is `${base}/${quote}:${quote}`, made from the indexer's `quoteSymbol`. The SDK keeps its case: its own comment says "venue token symbols are mixed-case identities ("USDso", …) and consumers key on them verbatim".
- **How it looks a symbol up.** `resolve()` does an exact `Map.get`, with no case folding.
- **The real `quoteSymbol`.** The Hideki indexer returns `"USDso"` for every perp market, so the real symbol is `BTC/USDso:USDso`.

What changed since the first report (29 Oct, SDK 0.32.1): the docs site is fixed. On 15 Nov 2026 the Perps guide (intro and Quick start) and the exchange page's Symbols table use `BTC/USDso:USDso`, and the tarball's `docs/*.md` no longer contain `USDSO`. What is still wrong is the copy that ships inside the package and shows in editors: the JSDoc examples above, the `UnifiedFundingRate.symbol` comment, and the symbol-grammar comment in `symbols.js`. `exchange.quotePerpClose` is new in 0.33.0, so its example is newly wrong, not left over.

## Evidence
- SDK 0.33.0, `grep -rn "USDSO" dist/`:
  - `dist/unified/exchange.d.ts` L2348: `const page = await exchange.fetchFundingRateHistory("BTC/USDSO:USDSO", since, 100);`
  - `dist/unified/exchange.d.ts` L2466: `const q = await exchange.quotePerpClose("BTC/USDSO:USDSO");`
  - `dist/unified/exchange.js` L4977 and L5220: the same two examples.
  - `dist/unified/structs.d.ts` L510 (`UnifiedFundingRate.symbol`): `/** The perp's tradable symbol, e.g. "BTC/USDSO:USDSO". */`
  - `dist/unified/symbols.js` L6: `//   perp          BTC/USDSO:USDSO           (settle-suffixed swap notation)`
- SDK 0.33.0 `dist/unified/symbols.js`:
  - L20-L24, `sanitizePart`: "Case is preserved — venue token symbols are mixed-case identities ("USDso", "USDC.e") and consumers key on them verbatim."
  - L278-L280: `const market = this.bySymbol.get(marketSymbol); if (!market) throw new InvalidInputError(\`unknown symbol ${ref} — call loadMarkets() first\`);`
- Indexer (read-only), 15 Nov 2026: `POST https://hideki-dev.smk.somnia.host/v1/graphql` with `{ Market(where:{marketType:{_eq:"PERP"}}, limit:30){ baseSymbol quoteSymbol } }` → HTTP 200, 30 rows, every `quoteSymbol` is `"USDso"`.
- Docs site, 15 Nov 2026 (now correct): https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/perps has "Perps are live on testnet: BTC/USDso:USDso and ETH/USDso:USDso" and `exchange.createOrder("BTC/USDso:USDso", …)`; https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/exchange Symbols table has `BTC/USDso:USDso`.
- The bot kit, maintained by the same team, uses `BTC/USDso:USDso`: https://github.com/somnia-chain/dreamdex-bot-kit/blob/4e75896896e024c03e2033d5920514f249998cd6/packages/perp-core/src/markets.ts#L13 and #L46.

## Environment
SDK version 0.33.0 (@somnia-chain/markets-sdk; the dreamDEX perps page names 0.32.1 as the app's version), network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review). First checked 29 Oct 2026 on 0.32.1, re-checked 15 Nov 2026.
