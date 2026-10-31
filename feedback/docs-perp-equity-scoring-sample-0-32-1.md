# Perps "account equity for scoring" sample does not work on SDK 0.32.1: unknown margin event kind, and its block range is ignored
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://somnia-platform-docs.vercel.app/markets/somnia-markets/typescript/perps (HTTP 200) and go to "Account equity for scoring is one read". The text says: "Three indexed reads list the flows, and each takes the same inclusive sinceBlock / toBlock". The sample then calls:
   ```ts
   const range = { sinceBlock: start.asOfBlock + 1n, toBlock: end.asOfBlock };
   client.getMarginEvents(account, { kinds: ["Deposit", "Credited", "Withdraw", "CreditReclaimed"], ...range, ...page })
   client.listPerpMarginPulls({ account, ...range, ...page })
   client.listPerpMainFundingEvents({ account, kind: "Returned", ...range, ...page })
   ```
2. Install `@somnia-chain/markets-sdk@0.32.1` and compile the sample with `tsc`.
3. Remove `"CreditReclaimed"` so it compiles, then run it on an account that deposited before `start.asOfBlock`.

## Expected
The sample compiles. Each read returns only the flows in blocks `start.asOfBlock + 1` to `end.asOfBlock`, so `tradingGain` excludes deposits and withdrawals made in that window and nothing else.

## Actual
In 0.32.1:
1. **`"CreditReclaimed"` is rejected.** `MarginEventKind` is `"Deposit" | "Withdraw" | "Locked" | "Unlocked" | "Credited"`, so the literal fails to compile. In JS it throws `InvalidInputError: Margin event kinds must contain only Deposit, Withdraw, Locked, Unlocked, Credited; got CreditReclaimed` before any request is made.
2. **The block range is dropped.** None of the three reads takes `sinceBlock` or `toBlock`.
   - `GetMarginEventsOptions` is `{ kinds?, limit?, offset? }`, and `getMarginEvents` builds its filter from account and kinds only.
   - `listPerpMarginPulls` takes `{ account?, pool?, orderId?, source?, limit?, offset? }`.
   - `listPerpMainFundingEvents` takes `{ account?, payer?, kind?, limit?, offset? }`.

   The range comes in through a spread, and TypeScript does not flag extra properties from a spread. So once step 3 makes the sample compile, it subtracts every deposit, withdrawal and pull in the account's history, and `tradingGain` comes out silently wrong by those flows.
3. **No client-side workaround for margin events.** 0.32.1 `MarginEvent` rows carry no `blockNumber` (fields: id, account, kind, pool, amount, granter, timestamp, txHash), so the documented block-exact netting cannot be done for them in 0.32.1.

This is the documented procedure for competition scoring. On the SDK version the app runs, a developer following the site either cannot compile it or, after the obvious fix, gets wrong numbers with no error.

## Evidence
- Docs page quotes above. The flows table on the same page also lists "getMarginEvents | Withdraw, CreditReclaimed (voucher sweep) | out".
- SDK 0.32.1 `dist/perp/history.d.ts`:
  - L63: `export type MarginEventKind = "Deposit" | "Withdraw" | "Locked" | "Unlocked" | "Credited";`
  - L69-L95: `GetMarginEventsOptions` has only `kinds`, `limit`, `offset`.
  - L101-L122: `MarginEvent` has no block field.
- SDK 0.32.1 `dist/perp/history.js`:
  - L76-L82: `const where = { account: … }; if (opts.kinds != null) where.kind = …; … gqlRequest(MarginEventsQuery, { where, limit, offset })`
  - L86-L92 and L102-L111: the allowed kinds are Deposit/Withdraw/Locked/Unlocked/Credited, and anything else throws `InvalidInputError`.
- SDK 0.32.1 `dist/somniaMarketsClient.d.ts`:
  - L4446-L4453: `listPerpMarginPulls(opts?: { account?; pool?; orderId?; source?; limit?; offset? })`
  - L4464-L4470: `listPerpMainFundingEvents(opts?: { account?; payer?; kind?; limit?; offset? })`
- The 0.32.1 tarball's own `docs/PERPS.md` (L98-L105) describes the limitation the site leaves out. It uses `kinds: ["Deposit", "Credited", "Withdraw"]` and says: "These reads cannot be bounded to a block yet, so a flow close to either snapshot cannot be placed exactly." Its sample stops at `const change = end.totalEquity - start.totalEquity; // still includes deposits, withdrawals and credits`. The docs site describes an unreleased API in this section.

## Environment
SDK version 0.32.1 (@somnia-chain/markets-sdk), network Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs/code review)
