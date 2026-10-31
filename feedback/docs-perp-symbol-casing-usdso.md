# Perps docs use symbol "BTC/USDSO:USDSO", but the SDK builds "BTC/USDso:USDso" and symbol lookup is case-sensitive
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/perps (HTTP 200) and go to "Quick start".
2. Run the sample against Hideki with SDK 0.32.1:
   ```ts
   await exchange.loadMarkets();
   await exchange.depositMargin("BTC/USDSO:USDSO", 1_000);
   await exchange.createOrder("BTC/USDSO:USDSO", "limit", "buy", 0.001, 62_000);
   await exchange.fetchFundingRate("BTC/USDSO:USDSO");
   ```

## Expected
The documented perp symbol resolves to the BTC perp.

## Actual
From the source, the first call throws `InvalidInputError: unknown symbol BTC/USDSO:USDSO — call loadMarkets() first`.

- **How the SDK builds the symbol.** A perp symbol is `${base}/${quote}:${quote}`, made from the indexer's `quoteSymbol`. The SDK keeps its case: its own comment says "venue token symbols are mixed-case identities ("USDso", …) and consumers key on them verbatim".
- **How it looks a symbol up.** `resolve()` does an exact `Map.get`, with no case folding.
- **The real `quoteSymbol`.** The Hideki indexer returns `"USDso"` for every perp market, so the real symbol is `BTC/USDso:USDso`.

The docs contradict themselves. The exchange page's multi-chain example uses `exchange.createOrder("BTC/USDso:USDso", …)`, while the Perps page and the exchange page's Symbols table use `BTC/USDSO:USDSO`.

Side note: the Perps page intro "Perps are live on testnet: BTC/USDSO:USDSO and ETH/USDSO:USDSO" is also out of date. The Hideki indexer returned 20 perp markets for a `limit: 20` query (LIT, BNB, DOGE, SOL, …, BTC, NEAR).

## Evidence
- Docs with the wrong casing:
  - Perps page, intro: "Perps are live on testnet: BTC/USDSO:USDSO and ETH/USDSO:USDSO".
  - Perps page, "Quick start": `exchange.depositMargin("BTC/USDSO:USDSO", 1_000)`, `exchange.createOrder("BTC/USDSO:USDSO", …)`, `exchange.fetchFundingRate("BTC/USDSO:USDSO")`.
  - https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/exchange (HTTP 200), "Symbols" table: `Perp | BTC/USDSO:USDSO | BTC/USDSO:USDSO`.
- Docs with the right casing: the same exchange page, "One exchange, several chains": `const order = await exchange.createOrder("BTC/USDso:USDso", "limit", "buy", 1, 60_000);`
- SDK 0.32.1 `dist/unified/symbols.js`:
  - L20-L24, `sanitizePart`: "Case is preserved — venue token symbols are mixed-case identities ("USDso", "USDC.e") and consumers key on them verbatim."
  - L57-L61: `if (m.marketType === "PERP") { … const quote = m.quoteSymbol ? sanitizePart(m.quoteSymbol) : …; return \`${base}/${quote}:${quote}\`; }`
  - L278-L280: `const market = this.bySymbol.get(marketSymbol); if (!market) throw new InvalidInputError(\`unknown symbol ${ref} — call loadMarkets() first\`);`
- Indexer (read-only): `POST https://hideki-dev.smk.somnia.host/v1/graphql` with `{ Market(where:{marketType:{_eq:"PERP"}}, limit:20){ baseSymbol quoteSymbol poolAddress } }` → HTTP 200. Every row has `"quoteSymbol":"USDso"`, e.g. `{"baseSymbol":"BTC","quoteSymbol":"USDso","poolAddress":"0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0"}`.
- The same uppercase strings ship in the 0.32.1 tarball: `docs/PERPS.md` L3 and L1197-L1200, `docs/EXCHANGE.md` L39, and the comment in `dist/unified/structs.d.ts` L503 `"BTC/USDSO:USDSO"`.
- The bot kit, maintained by the same team, uses `BTC/USDso:USDso`: https://github.com/somnia-chain/dreamdex-bot-kit/blob/25798660bbe4674a6f83a960b202852896e2fed4/packages/perp-core/src/markets.ts#L13 and #L46.

## Environment
SDK version 0.32.1 (@somnia-chain/markets-sdk), network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review)
