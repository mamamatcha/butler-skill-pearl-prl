# Changelog

## 0.1.0

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
