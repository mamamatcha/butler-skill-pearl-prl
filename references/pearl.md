# Pearl, WPRL and the bridge: the facts

Verified on Ethereum mainnet, 15 Oct 2026. Re-check before relying on any of it.

## Identifying the right token

**No contract address is written in this skill, deliberately.** A skill that carries
an address is one edit away from sending an owner's money somewhere else, so the
address always comes from `bevo-read token-search WPRL` and the owner confirms it.

Check the match against these properties instead:

```text
name      Wrapped Pearl
symbol    WPRL
decimals  8
chain     Ethereum mainnet (chain id 1), not Base, not an L2
supply    ~3.37M WPRL
```

If the match does not have all of those, it is not the token. Ask the owner rather
than picking.

## WPRL has 8 decimals, not 18

```text
name     Wrapped Pearl
symbol   WPRL
decimals 8
supply   ~3,373,543 WPRL
```

**This is the easiest thing to get wrong here.** Almost every ERC-20 a Butler touches
has 18 decimals; WPRL has 8. An amount computed against 18 is wrong by a factor of
10^10 - not a rounding error, a different number entirely. Pass human amounts to `acp
trade` and let it do the conversion; never hand-build base units.

## Both contracts are upgradeable

Both the WPRL token and the PearlBridge contract are **EIP-1967 proxies**, not fixed
code: each delegates to an implementation address that is stored, not hard-wired.

Whoever holds the upgrade key can change what either contract does, at any time,
without the token address changing. That is normal for bridges and worth saying
plainly when an owner asks what the risk is. It is not a reason to refuse - it is a
reason not to describe either as immutable.

## Calling the bridge

The bridge is an ordinary Ethereum contract. There is no Butler "bridge" feature and
none is needed - `acp wallet send-transaction` reaches it like any other contract.

**Everything below was read off live bridge transactions on 16 Oct 2026**, not from a
doc. Nine owners used it that day.

Two transactions, in this order:

### 1. Allowance

The bridge pulls the WPRL with `transferFrom`, so it needs an allowance first -
confirmed by five `Approval` events granted to it that day.

```text
to    the WPRL token
call  approve(spender = the bridge, amount = exactly what is being bridged)
```

**Never an unlimited allowance.** This is a third-party custodial contract that can be
upgraded; an infinite approval on it is a standing invitation.

### 2. The bridge call

```text
to        the bridge contract
selector  0xd2f33ed6
args      (uint256 amount, string prlAddress)
```

Standard ABI encoding, five 32-byte words in the observed calls:

```text
[0] amount        WPRL in its own 8-decimal units  (0x3b9aca00 = 1,000,000,000 = 10.00 WPRL)
[1] 0x40          offset to the string
[2] 0x3f          string length, 63
[3] prl1praqglvllkn007qjsdp4n0l2m4ax      the destination, as UTF-8 bytes
[4] htc6tpg23xnn0c4f0lklwnh8s9x0hls
```

The function's *name* is unknown and does not matter - the selector and the argument
layout are what build the transaction.

### What that call cost, measured

One observed transaction bridged **10.00 WPRL** and moved **0.05 WPRL** to the bridge
as its cut: **exactly 0.5%**, matching the published fee. What lands on the Pearl side
is the remainder.

### The destination is an argument, not a default

`prlAddress` is a **native PRL bech32 address**: starts `prl1`, 63 characters. The
bridge sends there and nowhere else, and it cannot be undone.

```text
prl1praqglvllkn007qjsdp4n0l2m4axhtc6tpg23xnn0c4f0lklwnh8s9x0hls
```

So "bridge it to my Pearl wallet" is not something the contract understands. The
owner's address has to be known, and read back to them, before anything is signed.

## The asymmetry, which is the whole product risk

| Leg | Who does it | Cost |
| --- | --- | --- |
| USDC → WPRL | Butler, `acp trade` | swap slippage |
| WPRL → native PRL | Butler, `approve` + `send-transaction` to the bridge | 0.5% |
| **native PRL → WPRL** | **the owner, by hand at pearlbridge.xyz** | **0.5%** |
| WPRL → USDC | Butler, `acp trade` | swap slippage |

Three of the four legs are one tap. The third is a website the owner has to visit
themselves. An owner who does not understand that before depositing has been
mis-sold, however good the rest of the flow is.

**Round trip is about 1% in bridge fees alone**, plus slippage both ways on a token
with a small float.

## Costs

- **PearlBridge: 0.5% each direction.** Minimum 4 PRL on the way back.
- **Minimum deposit $25.** This is the product's floor, not a gas floor.
- **Ethereum gas is currently negligible** - around 0.07 gwei, so a buy plus a bridge
  is cents. Do not quote that as a permanent fact: gas spikes, and at 30 gwei the same
  two transactions are dollars. Quote the bridge fee, which is fixed, and let gas be
  whatever the approval card says.

## The Pearl wallet

One per owner, created on the first bridge, and **permanent** - a Privy limit, not a
policy that can be waived. It is not an address the owner picks. Say this before the
first deposit, not after.

## Coming back out, step by step

What to tell an owner who wants their money back:

1. Go to **pearlbridge.xyz** and connect the wallet holding the native PRL.
2. Bridge PRL → WPRL. Costs 0.5%, minimum 4 PRL.
3. Wait for the WPRL to land on Ethereum.
4. Come back and ask Butler to sell it.

Butler cannot do steps 1-3. Do not imply otherwise, and do not offer to "start" the
withdrawal - there is nothing to start.
