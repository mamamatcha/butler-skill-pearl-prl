# Changelog

## 0.1.1

- **`bevo-wallet bridge-wprl` has shipped** (virtuals-agent#194, #203; bevo-server#1105),
  so the sell path runs as written.
- **Bridging the whole Pearl balance uses `--all`.** Pearl's network fee is paid from
  the wallet on top of the amount, so `--amount` set to the full balance is refused as
  `pearl_balance_short`. The skill says to pass `--all`, and what to do on that refusal.
- **Every card ends with a hand-off.** A card from chat returns before anything runs,
  and nothing wakes the agent when the owner approves, so a buy used to stop at the
  WPRL with no word of the bridge. Each card step now tells the owner what comes
  next and to say when they've approved, and "Picking up a flow" resumes on their
  next message from the last card's `approvalStatus`, never re-filing a step. (An
  in-turn wait was tried and dropped: it holds the chat, queuing anything else the
  owner sends.)
- The buy leaves out `--chain-in`; the server finds the cash on any chain.
- README: drops the "not yet listed" note.

## 0.1.0

- First release. **Buy:** USDC → WPRL on Ethereum, then bridge to native PRL in the
  owner's Pearl wallet. **Sell:** bridge PRL back to WPRL, wait for it to land, then
  sell for USDC on Base.
- **Bridging in is `bevo-wallet bridge-prl`**, Butler's own command. One card;
  bevo-server builds the exact-amount approve and the `requestBurn` call against
  contracts it pins, checks PearlBridge's quote, and pays only the owner's own Pearl
  wallet. The skill never builds calldata or handles a PRL address.
- **Bridging out is `bevo-wallet bridge-wprl`**, written against a command that has
  not shipped yet; see README.
- Waits for each bridge to land before the next step, and never re-runs a slow bridge.
- Quotes the exit fee in PRL: 0.5% with a **4 PRL minimum**, which dominates under
  800 PRL. Discloses PearlBridge as a custodial third party, the 6-confirmation wait
  and 24-hour slow lane on the way out, and the permanent one-per-owner Pearl wallet.
- Records that WPRL has **8 decimals**, and that WPRL and PearlBridge are both
  EIP-1967 proxies, so neither is immutable.
- `acp trade` names the chain with `--chain-out` when buying and `--chain-in` when
  selling, as the shipped `index-rebalance` template does. A sale settles as USDC on
  Base (`--chain-out 8453`).
