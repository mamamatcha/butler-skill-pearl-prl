# Pearl, WPRL and the bridge: the facts

Verified on Ethereum mainnet, 15 Oct 2026. Re-check before relying on any of it.

## Identifying the right token

**No contract address is written in this skill, deliberately.** A skill that carries
an address is one edit away from sending an owner's money somewhere else. The address
comes from `bevo-read token-search WPRL`, and must match the WPRL contract Butler's own
`references/wallets.md` names - a ticker search can rank an impersonator first.

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

`bevo-wallet bridge-prl --wallet <Pearl wallet> --amount <wprl>` is the whole call.
The skill never builds calldata, never names the bridge contract, and never handles a
PRL address. bevo-server does all of that, and refuses rather than improvises:

- It pins the WPRL and bridge contracts itself, and files nothing unless PearlBridge's
  quote names those same contracts on chain 1. A moved or upgraded contract stops the
  bridge instead of routing WPRL somewhere new.
- It refuses when PearlBridge is paused, past its daily limit, or unreachable, and
  when the quote's fee and net do not add up to the amount.
- It pays only one of the owner's own Pearl wallets, and only an address PearlBridge
  itself accepts (Taproot, past its OFAC screen).
- It files one card: an exact-amount `approve` and the burn, together. Never an
  unlimited allowance.

For a reader checking it on-chain: the burn is `requestBurn(uint256 grossAmount, string
pearlAddress)`, selector `0xd2f33ed6`, amount in WPRL's 8-decimal units. Observed on
16 Oct 2026: **10.00 WPRL** in, **0.05 WPRL** kept by the bridge, **exactly 0.5%**.

## Bridging back: PRL → WPRL

`bevo-wallet bridge-wprl --wallet <Pearl wallet> --amount <prl>`. PearlBridge's side
of it, from its own API (`api.pearlbridge.xyz/v1`, read 16 Oct 2026):

- PRL is sent from the Pearl wallet to a **deposit address PearlBridge derives for
  Butler's Ethereum wallet** (`/v1/deposit-address?ethAddress=`). PearlBridge mints
  WPRL to that Ethereum wallet once the deposit has **6 Pearl confirmations**.
- Fee: **0.5%, minimum 4 PRL, per deposit** (`depositFeeBps` 50,
  `depositFeeMinGrains` 400000000).
- Two lanes. The fast lane has its own daily cap; past it a deposit takes the slow
  lane and waits **24 hours** (`slowMintDelaySeconds` 86400).
- `/v1/mints/{pearlTxid}` follows a deposit to the mint.

## The four legs

| Leg | Command | Cost |
| --- | --- | --- |
| USDC → WPRL | `acp trade --chain-out 1` | swap slippage |
| WPRL → native PRL | `bevo-wallet bridge-prl` | 0.5% |
| native PRL → WPRL | `bevo-wallet bridge-wprl` | 0.5%, **minimum 4 PRL** |
| WPRL → USDC on Base | `acp trade --chain-in 1 --chain-out 8453` | swap slippage + Ethereum → Base bridge |

**Round trip is at least 1% in bridge fees**, more on a small amount because of the
4 PRL minimum, plus slippage both ways on a token with a small float.

## Costs

- **PearlBridge: 0.5% in; 0.5% with a 4 PRL minimum out.** Under 800 PRL the minimum
  is what applies: 4% on 100 PRL, 40% on 10 PRL.
- **Minimum trade $25**, each way. This is the product's floor, not a gas floor.
- **Ethereum gas is currently negligible** - around 0.07 gwei, so a buy plus a bridge
  is cents. Do not quote that as a permanent fact: gas spikes, and at 30 gwei the same
  two transactions are dollars. Quote the bridge fee, which is fixed, and let gas be
  whatever the approval card says.

## The Pearl wallet

One per owner, set up by Butler, and **permanent** - a Privy limit, not a policy that
can be waived. It is not an address the owner picks, and the bridge pays nowhere else.
`bevo-wallet list` shows its label, `prl1p…` address and PRL balance. A Pearl wallet
holds PRL only and cannot trade. Say this before the first deposit, not after.
