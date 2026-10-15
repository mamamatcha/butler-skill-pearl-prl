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

## The asymmetry, which is the whole product risk

| Leg | Who does it | Cost |
| --- | --- | --- |
| USDC → WPRL | Butler, `acp trade` | swap slippage |
| WPRL → native PRL | Butler, `acp wallet bridge-prl` | 0.5% |
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
