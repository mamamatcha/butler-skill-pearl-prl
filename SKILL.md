---
name: pearl-prl
description: Move money in and out of Pearl - buy WPRL on Ethereum and bridge it to native PRL, or sell WPRL back to USDC - and say plainly that the way out is not automatic.
version: 0.1.0
metadata: {"butler":{"moneyMoving":true,"keywords":["prl","pearl","wprl","buy prl","sell prl","bridge prl","native prl","pearl wallet","pearlbridge","wrapped pearl","get prl","cash out prl"],"requires":{"bins":["acp","bevo-read"]}}}
---

## When to use

Your owner wants to hold **native PRL**, or wants their PRL back as cash:

- **In** - "buy me some PRL", "get PRL into my Pearl wallet": buy **WPRL** on Ethereum
  with USDC, then bridge it to native PRL.
- **Out** - "sell my PRL", "cash out of Pearl": sell **WPRL** back to USDC. The
  PRL → WPRL half is **not** something Butler can do; see below.
- **Check** - what they hold, on either side.

Not this skill: anything on Pearl itself once the PRL is there, and PRL → WPRL, which
has no Butler command and has to be done by hand on a third-party website.

## Before you start

Four things, and the first one decides whether to go ahead at all.

1. **The way out is not automatic, and they must hear it before any money moves.**
   Butler can take them *in* - buy and bridge - in one flow. Coming *back* needs them
   to go to **pearlbridge.xyz** themselves, connect a wallet, and bridge PRL → WPRL by
   hand before Butler can sell it. If that is not something they will do, do not start.

2. **PearlBridge is a third party holding the money mid-flight.** Not Virtuals, not
   Pearl. It takes **0.5% each way**, so a round trip costs about **1%** before any
   trading cost. Say this as a fee, not a detail.

3. **How much, and where it comes from.** Minimum is **$25** of USDC, on Ethereum.

4. **The Pearl wallet is one per owner and permanent.** It is created on the first
   deposit and cannot be swapped for another later. Nothing is sent to an address they
   chose - this is the wallet Butler makes for them.

Read what they actually hold before proposing anything, and resolve WPRL to an exact
contract rather than trusting any address written down anywhere:

```sh
bevo-read assets
bevo-read token-search WPRL
```

Show the owner the matches and have them pick, even when there is only one. The right
one is **Wrapped Pearl (WPRL) on Ethereum, 8 decimals**. Never guess a contract and
never paste one from notes - a wrong address here sends their money to a stranger.

## Procedure

### Buying in

1. [ADAPT] Say the whole shape back to them in one message, with the exit named first.
   `references/pearl.md` has the numbers. Something like:

   ```text
   $100 of USDC becomes about $99.50 of native PRL.
   PearlBridge takes 0.5% on the way in, and 0.5% again whenever you come back.
   Coming back is not automatic: you would go to pearlbridge.xyz yourself and
   bridge PRL to WPRL by hand before I can sell it.
   Your Pearl wallet is created once and is yours permanently.
   ```

   Get a clear yes to **that**, not to "buy some PRL". A yes to the amount is not a
   yes to the one-way exit.

2. [FIXED] Buy WPRL with their USDC. This is a money command and renders an approval
   card:

   ```sh
   acp trade --token-in usdc --amount-in <AMOUNT_USD> --token-out <WPRL_ADDRESS> --chain 1 --idempotency-key <KEY>:buy
   ```

   `<WPRL_ADDRESS>` is the contract the owner confirmed from `token-search`, never one
   written into this skill.

3. [FIXED] Confirm the WPRL actually arrived before bridging. Bridging a balance that
   is not there yet fails halfway:

   ```sh
   bevo-read token-balance <WPRL_ADDRESS>
   ```

4. [FIXED] Bridge it to native PRL. Second money command, second approval:

   ```sh
   acp wallet bridge-prl --amount <WPRL_AMOUNT> --idempotency-key <KEY>:bridge
   ```

   The Pearl wallet is created here if it does not exist. Do not run a separate
   setup step.

5. [ADAPT] Tell them what landed and where, and repeat the exit once, briefly.

### Selling out

1. [ADAPT] Say the part Butler cannot do, first and without softening:

   ```text
   I can sell WPRL for you. I cannot turn native PRL back into WPRL - that part
   is yours to do at pearlbridge.xyz, and it takes 0.5%.
   ```

   If they hold only native PRL, that is the whole answer: tell them the step, and
   offer to sell once the WPRL shows up. Do not start anything.

2. [FIXED] Once WPRL is in the wallet, check it is really there:

   ```sh
   bevo-read token-balance <WPRL_ADDRESS>
   ```

3. [FIXED] Sell it:

   ```sh
   acp trade --token-in <WPRL_ADDRESS> --amount-in <AMOUNT> --token-out usdc --chain 1 --idempotency-key <KEY>:sell
   ```

## Idempotency and retries

Reads are free - re-run `bevo-read` rather than trusting a balance from a minute ago.

Buying, bridging and selling are money commands and are **not** idempotent on their
own. One `--idempotency-key` per step, and if a step's outcome is unclear **do not
re-run it** - a retried money command can spend twice. Look it up instead:

```sh
bevo-read request <KEY>
```

Bridging twice is the expensive mistake here: the second bridge is a second 0.5% fee
on money that is already across.

## Failure handling

| Outcome | What to do |
| --- | --- |
| Owner will not use pearlbridge.xyz to come back | Do not buy. There is no other way out. |
| Under $25 | Say the minimum and stop. Do not round them up to it. |
| Not enough USDC on Ethereum | Say what they hold and where. Do not bridge USDC to cover it as a surprise. |
| Buy filled but bridge failed | The WPRL is theirs and safe in their wallet. Say that first, then offer to bridge again. Never re-run the buy. |
| Bridge outcome unclear | Look the key up. Never bridge again on a guess. |
| Owner holds only native PRL and wants out | Give them the pearlbridge.xyz step. Offer to sell when the WPRL arrives. |
| `bridge-prl` is not a command | Say the bridge is not available from Butler right now, and that their WPRL is untouched. |
| `token-search WPRL` returns nothing, or several | Ask the owner for the contract, or stop. Never pick one yourself. |

## Limits

- **One direction is automated.** In: buy and bridge. Out: selling WPRL only. PRL →
  WPRL is manual, on a third-party site.
- **PearlBridge is custodial and third-party**, 0.5% each way. Funds sit with it
  mid-flight. Neither Virtuals nor Butler controls it or can recover a stuck bridge.
- **WPRL and the bridge are both upgradeable contracts.** Whoever holds the upgrade
  key can change how either behaves. Disclose it if asked what the risk is.
- Ethereum mainnet only, $25 minimum.
- One Pearl wallet per owner, created once, permanent.
- Not advice. Move what they ask for; never suggest the size or call PRL a good buy.

## Say to the owner

Before buying, with the exit in the same breath as the price:

"$100 of USDC gets you about $99.50 of native PRL - PearlBridge takes 0.5%. Coming
back out, you would bridge PRL to WPRL yourself at pearlbridge.xyz, which costs
another 0.5%, and then I can sell it. Happy with that?"

After: "Done - about 36.4 PRL is in your Pearl wallet. It is one wallet, yours
permanently."

When they hold only native PRL and want cash: "I can sell WPRL, but I cannot turn
native PRL back into WPRL - that step is at pearlbridge.xyz and costs 0.5%. Once the
WPRL is in your wallet, say the word and I will sell it."
