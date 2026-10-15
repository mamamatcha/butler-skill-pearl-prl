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
- **The Pearl wallet is one per owner and permanent** (a Privy limit).
- Ethereum mainnet only, $25 minimum.

All verified on-chain 15 Oct 2026; `references/pearl.md` carries the detail.

## Open question before this can be listed

The original notes called the bridge `bevo-wallet bridge-prl`. **`bevo-wallet` is not
a command the Butler container has** — the validator rejects it outright (its own are
`acp`, `app-checkout`, `bevo-automation`, `bevo-notify`, `bevo-read`, `bevo-rpc`,
`bevo-send`, `bevo-sms`, `bevo-x`). This skill uses `acp wallet bridge-prl`, which
passes validation because the `wallet` group's subcommands are not enumerated.

**That is not the same as the subcommand existing.** Someone with access to `acp-cli`
needs to confirm `wallet bridge-prl` is real, and correct the call if it is spelled
differently. Until then the bridge step is unverified — the buy and sell legs are
ordinary `acp trade` and are fine.

## Validating a change

```sh
curl -sSLO https://virtual-protocol.github.io/butler-skills/tools/validate.py
python3 validate.py --standalone .
```

## Releasing

Bump `version` in `SKILL.md`, add the `CHANGELOG.md` entry, merge to `main`. A
published `name@version` is immutable. Listing a skill is maintainer-only, and a
money-moving skill needs two maintainer reviews.
