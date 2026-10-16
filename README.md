# butler-skill-pearl-prl

The `pearl-prl` skill for [Butler](https://github.com/Virtual-Protocol/butler-skills).
For the humans maintaining this repo — never published to a butler. Only `SKILL.md`
and `references/**/*.md` reach one.

## What it does

Buys WPRL on Ethereum with USDC and bridges it to native PRL, or sells WPRL back to
USDC. Money-moving: every step is an `acp` money command with its own approval card
and idempotency key.

## The thing this skill exists to get right

**Three of the four legs are automated and one is not.** Butler can take an owner in
— buy, then bridge — in a single flow. Coming back requires them to visit
**pearlbridge.xyz** themselves and bridge PRL → WPRL by hand before Butler can sell.

So the skill is built to say the exit *before* the price, and to require a yes to the
one-way exit specifically, not just to the amount. An owner who deposits without
understanding that has been mis-sold however smooth the rest of the flow is.

## Facts worth knowing before changing anything

- **WPRL has 8 decimals, not 18.** An amount computed against 18 is wrong by 10^10.
- **WPRL and PearlBridge are both EIP-1967 proxies** — upgradeable, so neither should
  ever be described as immutable.
- **PearlBridge is custodial and third-party**, 0.5% each way (~1% round trip).
- **The Pearl wallet is one per owner and permanent** (a Privy limit), and its
  address is a **required argument** to the bridge.
- Ethereum mainnet only, $25 minimum.

All verified on-chain 15 Oct 2026; `references/pearl.md` carries the detail.

## How the bridge actually works

The original notes called it `bevo-wallet bridge-prl`. **That command does not
exist** — `bevo-wallet` is not a Butler container command at all, and the validator
rejects it outright.

It turns out none is needed. **The bridge is an ordinary Ethereum contract**, and
`acp wallet send-transaction` — a documented Butler money command — reaches it like
any other. Two transactions: `approve`, then the bridge call.

Everything about that call was read off **live bridge transactions on 16 Oct 2026**,
not from a doc:

| | |
| --- | --- |
| Selector | `0xd2f33ed6` (nine real calls that day) |
| Arguments | `(uint256 amount, string prlAddress)` |
| Amount units | WPRL's own **8 decimals** |
| Destination | native PRL bech32, `prl1…`, 63 chars — **an argument, not a default** |
| Allowance | required; the bridge pulls with `transferFrom` (five approvals that day) |
| Fee, measured | 10.00 WPRL bridged → 0.05 WPRL taken = **exactly 0.5%** |

The function's name is unknown and irrelevant — the selector and argument layout are
what build the transaction. `references/pearl.md` carries the encoding.

**Consequence for the flow:** the bridge sends to an address it is given. "Bridge it
to my Pearl wallet" is not something the contract understands, so the owner's native
PRL address must be known, and read back to them, before anything is signed.

## Validating a change

```sh
curl -sSLO https://virtual-protocol.github.io/butler-skills/tools/validate.py
python3 validate.py --standalone .
```

## Releasing

Bump `version` in `SKILL.md`, add the `CHANGELOG.md` entry, merge to `main`. A
published `name@version` is immutable. Listing a skill is maintainer-only, and a
money-moving skill needs two maintainer reviews.
