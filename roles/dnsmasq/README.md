# dnsmasq

Split-DNS resolver for the tailnet, running on `hermes-agent` (the `oracle` ssh
alias) alongside [caddy](../caddy/README.md).

`muresine.top` has no public address records. Names under it resolve only
inside the tailnet, via this resolver, which Tailscale routes to through Split
DNS. Caddy still gets real Let's Encrypt certificates, because DNS-01 only
needs permission to write `TXT` records in the Cloudflare zone — an `A` record
was never part of that.

The zone is served as a wildcard: every name under `muresine.top` answers with
hermes-agent's tailnet address, where Caddy does the actual per-service
routing. **Adding a service means adding a vhost in `roles/caddy` and nothing
else** — no DNS record here, in Cloudflare, or anywhere.

## Why not a public record

Pointing a public record at a tailnet address looks like it works, but
`100.64.0.0/10` is the CGNAT range, and consumer routers with DNS rebinding
protection strip answers in that range. Any LAN client using the router as its
resolver gets an empty answer. Split DNS sidesteps that: the query never
reaches the router.

## Run

    ansible-playbook -i inventory playbooks/dnsmasq.yml

Then, once per tailnet, in the Tailscale admin console under **DNS → Split
DNS**: add `muresine.top` pointing at hermes-agent's tailnet address. Without
that, nothing routes here and the names will not resolve.

## Caveats

- The resolver binds to the tailnet address only. This host has a public
  interface too, and dnsmasq's default is to listen on all of them — an open
  resolver is an amplification vector, so the role asserts a tailnet address
  exists rather than falling back.
- `no-resolv` means this answers `muresine.top` and forwards nothing. That is
  deliberate; Split DNS sends it only that domain. Pointing a client's whole
  resolver at it will not work.
- The wildcard answers for names Caddy has no vhost for. Those resolve fine and
  then fail at TLS, which reads as a certificate error rather than a DNS error.
