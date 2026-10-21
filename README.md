# butler-skill-pearl-prl

Teaches a Butler agent to move its owner's money in and out of Pearl: buy native PRL with
USDC, and sell it back for USDC on Base.

- **Buy:** USDC → WPRL on Ethereum with `acp trade`, then WPRL → PRL into the owner's Pearl
  wallet with `bevo-wallet bridge-prl`.
- **Sell:** PRL → WPRL with `bevo-wallet bridge-wprl`, wait for it to land, then WPRL → USDC
  on Base with `acp trade --chain-out 8453`.

Two things shape how it works:

- **Butler's own commands do the bridging, never the skill.** PearlBridge, a custodial third
  party, carries PRL between Pearl and Ethereum. bevo-server builds every bridge call from
  contract addresses it pins, checks PearlBridge's quote, and pays only the owner's own
  Pearl wallet. The skill passes a wallet label and an amount, and holds no contract
  address, calldata or PRL address.
- **Each leg waits for the last to land.** Bridging out takes 6 Pearl confirmations, and a
  large amount can sit a day in PearlBridge's slow lane. The skill never bridges or sells a
  balance that has not arrived, and never re-runs a slow bridge: on the way out each bridge
  costs at least 4 PRL.

One Butler skill, published by the
[Butler skill hub](https://github.com/Virtual-Protocol/butler-skills): the hub's
`skills.json` lists it by name, repo and ref.

- `SKILL.md` — the playbook (see the hub's
  [SKILL_STANDARD.md](https://github.com/Virtual-Protocol/butler-skills/blob/main/SKILL_STANDARD.md))
- `references/pearl.md` — the facts the playbook points to: identifying WPRL, its 8
  decimals, fees, and how each bridge works
- `CHANGELOG.md` — one entry per version; every change bumps `version` in SKILL.md

Validate locally:

```sh
curl -sSLO https://virtual-protocol.github.io/butler-skills/tools/validate.py
python3 validate.py --standalone .
```
