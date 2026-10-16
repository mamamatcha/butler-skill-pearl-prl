---
name: pearl-prl
description: Move money in and out of Pearl - buy native PRL with USDC by way of WPRL on Ethereum and PearlBridge, or bridge PRL back to WPRL and sell it for USDC on Base.
version: 0.1.0
metadata: {"butler":{"moneyMoving":true,"keywords":["prl","pearl","wprl","buy prl","sell prl","bridge prl","native prl","pearl wallet","pearlbridge","wrapped pearl","get prl","cash out prl"],"requires":{"bins":["acp","bevo-read","bevo-wallet"]}}}
---

## When to use

Your owner wants to hold **native PRL**, or wants their PRL back as cash:

- **Buy** - "buy me some PRL", "get PRL into my Pearl wallet": buy **WPRL** on
  Ethereum with USDC, then bridge it into their Pearl wallet.
- **Sell** - "sell my PRL", "cash out of Pearl": bridge their PRL back to **WPRL**,
  wait for it to land, then sell it for USDC on Base.
- **Check** - what they hold, on either side.

Not this skill: sending PRL to someone (`bevo-send`), or anything on Pearl itself.

## Before you start

1. **PearlBridge is a third party holding the money mid-flight.** Not Virtuals, not
   Pearl. Custodial, upgradeable, and neither Butler nor Virtuals can recover a stuck
   bridge. Say so before the first bridge either way.

2. **The fees are not symmetric.** Into PRL: **0.5%**. Back out to WPRL: **0.5% with a
   4 PRL minimum** per bridge - on 100 PRL that is 4%, on 10 PRL it is 40%. Quote the
   fee in PRL, not as a percentage, whenever the amount is under 800 PRL.

3. **Trades are on Ethereum, $25 minimum** each way.

4. **The PRL lives in their Pearl wallet, and only there.** One per owner and
   permanent (a Privy limit). The bridge pays nowhere else, so never ask for an
   address.

Read what they hold before proposing anything. `bevo-wallet list` shows their Pearl
wallet's label, its `prl1p…` address and its PRL balance:

```sh
bevo-read assets
bevo-wallet list
```

Resolve WPRL to an exact contract rather than trusting a ticker:

```sh
bevo-read token-search WPRL
```

The right one is **Wrapped Pearl (WPRL) on Ethereum, 8 decimals**, and it must be the
WPRL contract in Butler's own `references/wallets.md`. A ticker search can rank an
impersonator first: if the two disagree, stop and ask the owner. Never guess a
contract and never paste one from notes.

## Procedure

### Buying PRL

1. [ADAPT] Say the whole shape back in one message. `references/pearl.md` has the
   numbers. Something like:

   ```text
   $100 of USDC buys WPRL on Ethereum, then PearlBridge turns it into native PRL
   in your Pearl wallet, keeping 0.5%. Coming back out costs 0.5% with a 4 PRL
   minimum, plus the trade back to USDC.
   ```

   Get a clear yes before moving anything.

2. [FIXED] Buy WPRL with their USDC. Money command, approval card:

   ```sh
   acp trade --token-in usdc --amount-in <AMOUNT_USD> --token-out <WPRL_ADDRESS> --chain-out 1 --idempotency-key <KEY>:buy
   ```

   `<WPRL_ADDRESS>` is the contract confirmed above, never one written into this
   skill. The chain flag is `--chain-out` when buying and `--chain-in` when selling -
   it names the side WPRL sits on.

3. [FIXED] Confirm the WPRL arrived before bridging. Bridging a balance that is not
   there yet fails:

   ```sh
   bevo-read token-balance <WPRL_ADDRESS>
   ```

4. [FIXED] Bridge it into their Pearl wallet. Money command, approval card:

   ```sh
   bevo-wallet bridge-prl --wallet <PEARL_LABEL> --amount <WPRL_AMOUNT> --idempotency-key <KEY>:bridge-in
   ```

   `<PEARL_LABEL>` is their Pearl wallet's label from `bevo-wallet list`.
   `<WPRL_AMOUNT>` is a plain decimal, at most 8 places - what step 3 showed, never a
   base-unit number. Butler builds the approve and the burn itself against
   PearlBridge's quote; the card shows what arrives after the fee.

5. [ADAPT] Tell them what is on its way and where. PRL lands some minutes after the
   card executes, 0.5% less than left.

### Selling PRL

1. [ADAPT] Say the whole shape back in one message, the fee in PRL:

   ```text
   PearlBridge turns your 500 PRL into about 496 WPRL on Ethereum (fee: 4 PRL),
   then I sell that for USDC on Base. The bridge needs 6 Pearl confirmations;
   a large amount can wait up to a day in PearlBridge's slow lane.
   ```

   Get a clear yes. If they already hold WPRL and want only that sold, skip to step 4.

2. [FIXED] Bridge PRL back to WPRL. Money command, approval card:

   ```sh
   bevo-wallet bridge-wprl --wallet <PEARL_LABEL> --amount <PRL_AMOUNT> --idempotency-key <KEY>:bridge-out
   ```

   `<PRL_AMOUNT>` is a plain decimal, at most 8 places, no more than the Pearl
   balance `bevo-wallet list` showed. The WPRL lands in Butler's Ethereum wallet.

3. [FIXED] Wait for the WPRL to land before selling. Check the bridge, then the
   balance; re-run these a few minutes apart until the WPRL is there:

   ```sh
   bevo-read request <KEY>:bridge-out
   bevo-read token-balance <WPRL_ADDRESS>
   ```

   **Never re-run the bridge because the WPRL is slow** - that is a second bridge and
   a second 4 PRL minimum. Tell the owner it is still in flight.

4. [FIXED] Sell the WPRL for USDC on Base. Money command, approval card:

   ```sh
   acp trade --token-in <WPRL_ADDRESS> --chain-in 1 --amount-in <WPRL_AMOUNT> --token-out usdc --chain-out 8453 --idempotency-key <KEY>:sell
   ```

   `<WPRL_AMOUNT>` is what step 3 showed arrived. One command sells on Ethereum and
   moves the USDC to Base; it blocks until the USDC lands, which can take a few
   minutes. Keep it running rather than re-issuing.

5. [ADAPT] Tell them how much USDC landed on Base.

## Idempotency and retries

Reads are free - re-run `bevo-read` rather than trusting a balance from a minute ago.

Trades and bridges are money commands and are **not** idempotent on their own. One
`--idempotency-key` per step, and if a step's outcome is unclear **do not re-run it**
- a retried money command can spend twice. Look it up instead:

```sh
bevo-read request <KEY>
```

A second bridge is the expensive mistake: another fee on money already across, and
on the way out another 4 PRL at least.

## Failure handling

| Outcome | What to do |
| --- | --- |
| Under $25 to buy, or WPRL worth under $25 to sell | Say the minimum and stop. Do not round them up to it. |
| Selling under ~40 PRL | The 4 PRL minimum is over 10%. Say the number and let them decide. |
| Not enough USDC on Ethereum | Say what they hold and where. Do not move USDC to cover it as a surprise. |
| Buy filled but bridge-in failed | The WPRL is theirs, in their Ethereum wallet. Say that first, then offer to bridge again. Never re-run the buy. |
| Bridge-out sent, WPRL not yet landed | Still in flight: 6 confirmations, or the slow lane. Check again later. Never bridge again. |
| Bridge-out failed | The PRL is still theirs, in their Pearl wallet. Say that first. |
| WPRL landed but the sale failed | The WPRL is theirs, in their Ethereum wallet. Offer to sell again. Never bridge again. |
| PearlBridge paused, unreachable, or past its daily limit | Nothing moved. Say so, and offer later or a smaller amount. |
| Bridge refuses its own quote as untrusted | Stop. A contract moved or was upgraded; do not work around it. |
| `token-search WPRL` disagrees with `references/wallets.md` | Ask the owner, or stop. Never pick one yourself. |

## Limits

- **PearlBridge is custodial and third-party.** Funds sit with it mid-flight.
- **WPRL and the bridge are both upgradeable contracts.** Whoever holds the upgrade
  key can change how either behaves. Disclose it if asked what the risk is.
- Trades on Ethereum only, $25 minimum. Sales settle as USDC on Base.
- One Pearl wallet per owner, permanent. A Pearl wallet cannot trade.
- Not advice. Move what they ask for; never suggest the size or call PRL a good buy.

## Say to the owner

Before buying: "$100 of USDC gets you about $99.50 of native PRL - PearlBridge takes
0.5%. Selling later costs 0.5% with a 4 PRL minimum on the bridge back. Go ahead?"

Before selling: "Your 500 PRL becomes about 496 WPRL after PearlBridge's 4 PRL fee,
then I sell it for USDC on Base. It can take a while to clear the bridge. Go ahead?"

While waiting: "The bridge has your PRL and is waiting on Pearl confirmations. I will
sell as soon as the WPRL lands - nothing for you to do."
