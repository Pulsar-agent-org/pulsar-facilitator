# Contributing to pulsar-facilitator

You do not need to know Stellar to contribute. The parts that touch a blockchain
are explained here, and everything runs on the Testnet with free test funds.

## What this repository is, and the other two

Pulsar lets an HTTP endpoint or an MCP tool charge a small amount per call. This
repository is the facilitator: a stateless service that verifies and settles
those payments so a provider does not have to run Stellar infrastructure.

Pulsar is three repositories in the `Pulsar-agent-org` organization, and they
depend on each other in one direction only:

    pulsar-spec  ->  pulsar-js  ->  pulsar-facilitator

- pulsar-spec: the normative protocol, the threat model, and the conformance
  vectors.
- pulsar-js: the TypeScript implementation, including `@pulsar/core`.
- pulsar-facilitator (here): the verify-and-settle service, built on
  `@pulsar/core`.

This repository depends on the other two. It must not re-implement header
parsing or verification; all protocol logic comes from `@pulsar/core`. Protocol
changes belong in pulsar-spec, not here.

## The 402 handshake in plain language

A Stellar payment moves an asset from one account (a `G...` address) to another
and can carry a small tag called a memo. Pulsar uses the memo to bind a payment
to one request.

    GET /tools/summarize
    <- 402 Payment Required
       WWW-Authenticate: Pulsar network="stellar:testnet", asset="USDC:GBBD...",
         amount="0.002", pay_to="GA7Q...", nonce="9f1c...", expires="1789..."

    The caller pays 0.002 USDC on Stellar, memo = hash(nonce), then retries:

    GET /tools/summarize
       Authorization: Pulsar tx="<hash>", nonce="9f1c..."
    <- 200 OK

The facilitator runs the verify step: given the requirement and the proof, it
checks the payment on-chain and returns valid or a reason code. It can also
settle a transaction on a caller's behalf. The nonce is single-use and expires.
Amounts are compared in stroops, the indivisible unit of a Stellar asset, never
as floating-point numbers.

## Repository map

    src/                  Service code (planned; not yet present).
    test/                 Tests (planned).
    docs/                 Deployment and operations documentation.
    .github/              Issue and pull request templates, and CI.

Generated directories such as `node_modules/` and `dist/` are never committed.

## Getting set up

Prerequisites:

- Node.js 20 or newer and pnpm 9 or newer (`npm install -g pnpm`).
- Docker and Docker Compose, for the container work.
- git and a GitHub account.
- No Stellar account and no funds up front. Test accounts are funded for free by
  a faucet called friendbot, on the Testnet only.

Fund a Testnet account:

1. Generate a keypair. Any Stellar SDK or the Stellar Laboratory can do this; it
   produces a public key starting with `G` and a secret starting with `S`.
2. Open `https://friendbot.stellar.org/?addr=<G...>` in a browser, or curl it.
   Friendbot creates and funds the account with test XLM.
3. Never commit a secret key. Configuration comes from environment variables and
   a committed `.env.example`.

The command that proves the setup works: once code lands, `pnpm install` then
`pnpm test`. There is no code yet, so there is nothing to run today; the open
issues track the work that will make this section real.

## Where to start

Issues carry a difficulty label: `good first issue`, `intermediate`, or
`advanced`. Area labels (`area:verify`, `area:settle`, `area:ops`, `area:docs`)
say which part of the service an issue touches.

Unclaimed work, easiest first:

1. Write the deployment guide for self-hosting the facilitator. Difficulty: good
   first issue. It orients every other contributor.
2. Add structured logging and request tracing behind a small interface.
   Difficulty: intermediate. Needed before anyone can debug a live instance.
3. Add rate limiting by client IP and by provider key. Difficulty: intermediate.
   Protects a public instance from abuse.
4. Implement `POST /verify` on top of `@pulsar/core`. Difficulty: intermediate.
   The core of the service.
5. Implement `POST /settle` and the Dockerfile and Compose file. Difficulty:
   intermediate.
6. Build the provider dashboard and the tool directory. Difficulty: advanced.
   The largest pieces of work that live in this repository.

Two cross-cutting pieces live in the other repositories but shape this one. The
largest single piece of the whole project is payment channels, specified in
pulsar-spec; the most urgent is a persistent nonce store in pulsar-js, because
the current in-memory store forgets consumed nonces on restart. A facilitator
built before those land inherits their limits.

Claim an issue by commenting on it before you start. For anything that changes
the service's external shape or its trust assumptions, open a discussion first
and wait for a maintainer to agree the approach.

## Rules that matter here

A reviewer will send a pull request back for any of these:

- Weakening replay protection. A nonce is single-use and expires. A change near
  verification must ship a test proving a replayed proof fails, not only that a
  fresh one passes.
- Re-implementing protocol logic. Header parsing, memo derivation, and
  verification come from `@pulsar/core`. Duplicating them here lets the two drift
  apart and breaks interoperability.
- A float on a payment path. Amounts are integer stroops end to end.
- Serving a `valid` verdict without confirming settlement on-chain. Fail closed:
  any error, timeout, or ambiguity returns invalid.
- Holding state the service is supposed to be without. The facilitator is
  stateless; persistent nonce or session state belongs behind a documented
  interface, not baked into a handler.

## Code style and CI

TypeScript, ESM, Node 20. Formatting is Prettier. When the service exists, CI
will run, and every contributor should run locally before pushing:

    pnpm install
    pnpm build
    pnpm test
    pnpm lint

Follow Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `chore:`).
Branch names are `type/short-description`, for example `feat/rate-limiting`.

## Pull request checklist

- [ ] The change is a single logical step with a Conventional Commit message.
- [ ] Tests pass, and a security-relevant change adds a test for the failure case.
- [ ] Protocol logic is used from `@pulsar/core`, not re-implemented.
- [ ] No secret keys, and no mainnet configuration, are introduced.
- [ ] Docs are updated when behavior or configuration changes.
- [ ] A dependent pull request in pulsar-js or pulsar-spec is linked here.

## Releases

Maintainers cut releases and tag them. A contributor should not edit the version,
create tags, or publish images. Propose a release in a discussion if you think
one is due.

## Security

Report vulnerabilities privately through GitHub's private vulnerability reporting
on this repository, never as a public issue. See `SECURITY.md`. The sensitive
surfaces here are the verification path, the settlement path and any signing key
it uses, and the rate limiter. The project is unaudited and Testnet only.

## Community

Design discussion happens in GitHub Discussions and in issues labeled for design.
Consistent, helpful triage of issues earns triage rights; a track record of
merged, well-tested pull requests earns commit rights. Protocol changes need a
written proposal in pulsar-spec and a maintainer sign-off before any behavior
changes here. Be precise and be kind; a review that pushes for a failure-case
test is doing its job.
