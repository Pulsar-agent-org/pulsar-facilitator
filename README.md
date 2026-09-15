# pulsar-facilitator

Pulsar is a pay-per-call payment layer for HTTP APIs, AI agents, and MCP servers,
settled in USDC on Stellar. This repository is the facilitator: a stateless
service that verifies and settles Pulsar payments on behalf of a tool provider,
so the provider does not have to run Stellar infrastructure itself.

## Status

Early and pre-release. Testnet only. Unaudited. This repository currently holds
the specification of the service and its planned work; it does not yet contain
runnable code. Do not use it for anything holding real value.

## The 402 handshake

A provider answers an unpaid request with `402 Payment Required` and a challenge:

    GET /tools/summarize
    <- 402 Payment Required
       WWW-Authenticate: Pulsar network="stellar:testnet", asset="USDC:GBBD...",
         amount="0.002", pay_to="GA7Q...", nonce="9f1c...", expires="1789..."

The caller pays the amount on Stellar with a memo derived from the nonce, then
repeats the request with a proof of payment:

    GET /tools/summarize
       Authorization: Pulsar tx="<hash>", nonce="9f1c..."
    <- 200 OK

The facilitator exists to run the verification and settlement steps of this
exchange as an HTTP service. A provider that does not want to talk to Stellar
directly calls the facilitator's `/verify` to check a proof, and can delegate
`/settle` to submit a transaction. The facilitator holds no protocol logic of
its own; it consumes `@pulsar/core`.

## Where this repository sits

Pulsar is three repositories, and dependencies point in one direction only:

    pulsar-spec  ->  pulsar-js  ->  pulsar-facilitator

- [pulsar-spec](https://github.com/Pulsar-agent-org/pulsar-spec): the normative
  protocol and the conformance suite.
- [pulsar-js](https://github.com/Pulsar-agent-org/pulsar-js): the TypeScript
  implementation, including `@pulsar/core`.
- pulsar-facilitator (this repository): the verify-and-settle service, built on
  `@pulsar/core`.

This repository depends on the other two. Nothing here may make them depend on
it, and this service must not re-implement header parsing or verification that
already lives in `@pulsar/core`.

## Planned surface

- `POST /verify`: takes a requirement and a payment proof, returns valid or
  invalid with a reason code from the spec's error table.
- `POST /settle`: submits a transaction on behalf of a caller.
- A health endpoint, structured JSON logs, and request tracing.
- Rate limiting by client IP and by provider key.
- A Dockerfile and a Compose file that bring the service up against Testnet in
  one command.

## Not yet runnable

There is no code to run yet. When the service exists, this section will carry the
exact commands to build the image and start it against Testnet. Until then, see
the open issues for the planned work and `CONTRIBUTING.md` to get involved.

## License

Apache-2.0. See `LICENSE`.
