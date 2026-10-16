# Changelog

## 0.1.0

- **The bridge is a contract call, not a missing Butler command.** The notes'
  `bevo-wallet bridge-prl` does not exist and nothing equivalent does; the bridge is
  reached with `acp wallet send-transaction` on Ethereum, like any other contract.
  Selector `0xd2f33ed6`, arguments `(uint256 amount, string prlAddress)`, amount in
  WPRL's 8-decimal units, preceded by an exact-amount `approve` — all read off live
  bridge transactions rather than from a doc, and the 0.5% fee verified against one
  (10.00 WPRL in, 0.05 WPRL taken).
- The destination PRL address is a **required argument**, so the skill now asks for
  it, validates its shape, and reads it back before signing. A wrong one is
  irreversible.

- Corrected the `acp trade` flags: the chain is named with `--chain-out` when buying
  and `--chain-in` when selling, not `--chain`. Taken from the shipped
  `index-rebalance` template rather than guessed; the first version would have failed
  at runtime.

- First release. Buys WPRL with USDC on Ethereum and bridges it to native PRL; sells
  WPRL back to USDC.
- **The one-way exit is said before the price**, and the flow needs a yes to it
  specifically: PRL → WPRL has no Butler command and is done by the owner at
  pearlbridge.xyz. A yes to the amount is not a yes to that.
- Discloses PearlBridge as a custodial third party taking 0.5% each way, the $25
  minimum, and the permanent one-per-owner Pearl wallet.
- Records that WPRL has **8 decimals**, and that WPRL and PearlBridge are both
  EIP-1967 proxies, so neither is immutable.
- Bridge is called as `acp wallet bridge-prl`; `bevo-wallet` from the original notes
  is not a container command and is rejected by the validator. The subcommand's
  spelling is unverified — see README.
