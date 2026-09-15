# Security policy

The project is unaudited and runs on the Stellar Testnet only. Do not use it with
real value.

## Reporting

Report privately. Do not open a public issue for a suspected vulnerability. Use
GitHub's private vulnerability reporting on this repository (the Security tab,
"Report a vulnerability"). If that is unavailable, email noevidence1017@gmail.com
and mark the subject as a security report.

Include the affected version or commit, a description of the flaw, and a concrete
reproduction. Expect an acknowledgement within a few days. Please allow time for
a fix before public disclosure.

## Sensitive surfaces

- The verification path. A `valid` verdict must follow only from confirmed
  on-chain settlement; any bug that returns valid for an unpaid or underpaid
  proof is critical.
- The settlement path and any signing key it uses. A key must come from the
  environment, never be logged, and never be committed.
- Replay handling inherited from `@pulsar/core`. The facilitator must not weaken
  the single-use nonce guarantee.
- The rate limiter, as a denial-of-service surface for a public instance.

## Scope

In scope: the service's verification, settlement, logging, and rate-limiting
code once it exists. Protocol-level issues belong in
[pulsar-spec](https://github.com/Pulsar-agent-org/pulsar-spec); implementation
issues in `@pulsar/core` belong in
[pulsar-js](https://github.com/Pulsar-agent-org/pulsar-js). Out of scope: mainnet
configuration, which is not supported.
