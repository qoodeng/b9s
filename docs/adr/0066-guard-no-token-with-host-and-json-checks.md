---
type: ADR
id: "0066"
title: "Guard --no-token with a loopback Host and JSON writes"
status: active
date: 2026-11-05
---

## Context

ADR 0020 keeps `--no-token` for local development, on a loopback listen
address only. The listen address says nothing about who sent a request:

- A web page in any browser on the machine can POST `text/plain` to
  `127.0.0.1:7979` without a preflight, and bd runs the write.
- DNS rebinding lets a page read every issue under its own name.
- `tailscale serve` proxies from loopback, so a tailnet device reaches the
  server as if it were local. ADR 0020 records this for the peer address.

The user guide also sends people to `--no-token` for the Memory preview, so it
is not only used by developers of b9s.

## Decision

**Without pairing, `b9s web` answers only requests whose `Host` is a loopback
name, and refuses a request that changes state unless it is
`application/json`.** A foreign `Host` gets 403 and any other content type 415.

Public mode (ADR 0029) keeps answering any `Host`: it is served through a proxy
and offers the same access to everyone by design.

## Consequences

- The browser app needs no change: it already sends JSON from the same origin.
- `--no-token` behind `tailscale serve` or any other proxy stops working. Pair
  the browser instead, as ADR 0020 requires for that route.
- A cross-site request can still reach the read endpoints, but the browser
  gives the page no access to the response without CORS headers.
