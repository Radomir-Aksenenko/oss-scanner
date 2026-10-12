# Threat model

## What this project does and where untrusted input enters
- Obsidian is a layer-3 VPN transport protocol (Go) designed to resist DPI and active probing: framing and padding, CPS signatures, ChaCha20-Poly1305 with X25519 / ML-KEM key exchange, REALITY-style TLS camouflage, UDP multiplexing.
- The server (`cmd/server`) listens on the public Internet. All network input to it is untrusted: handshake packets, framed data, TLS camouflage/probe traffic, and DNS requests proxied for clients.
- The client (`cmd/client`, `pkg/mobile`, `bindings/c`) parses server responses and connection URIs (`obsidian/uri.go`), which are untrusted too.

## Components that matter most / least
- Most: `obsidian/` handshake, crypto, protocol/framing, transport, reality, udp, uri; `cmd/server` (rate limiting, revocation, DNS proxy, TUN routing, egress).
- Less: `cmd/probe`, `cmd/client` platform glue, `bindings/c`.
- Out of scope: `tools/tspu_testbed` (research scripts), `examples/`.

## How to exercise it
- `go test ./obsidian/... ./cmd/...` — unit tests, including handshake, protocol, tunnel and server tests.
- Binaries are built to `/usr/local/bin/obsidian-server`, `obsidian-client`, `obsidian-probe`.

## How you rate severity
- Remote unauthenticated crash, memory-safety-like issue, authentication/handshake bypass, key or session compromise, replay-protection bypass: high or critical.
- Distinguishing the server from a normal TLS service via active probing (breaks the anti-censorship goal): medium to high.
- Traffic leaking outside the tunnel, client-side issues triggered by a malicious server or URI: medium to high.
- Pure resource-exhaustion DoS: medium.

## Anything to leave alone
- Platform-specific code that cannot be built or run on Linux (Windows/macOS/iOS/Android glue) can be reviewed but does not need reproducers.
