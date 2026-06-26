# Unbound DNS Configuration

A production-grade [Unbound](https://www.nlnetlabs.nl/projects/unbound/) configuration for a
**recursive, caching, DNSSEC-validating** DNS resolver running on the loopback
interface — a common upstream setup for Pi-hole, AdGuard Home, or a single host.

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Platform: Debian](https://img.shields.io/badge/Platform-Debian-red.svg)
![Unbound](https://img.shields.io/badge/Unbound-1.13%2B-green.svg)

## Contents

- [Features](#features)
- [Real-world performance](#real-world-performance)
- [Requirements](#requirements)
- [Installation](#installation)
- [Verification](#verification)
- [Configuration notes](#configuration-notes)
- [Tuning the open-file limit](#tuning-the-open-file-limit)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Features

- **Recursive resolution** — resolves names directly from the root servers down, with no third-party upstream resolver required.
- **Caching** — local message, RRset, and infrastructure caches for fast repeat lookups; prefetching keeps popular records warm.
- **DNSSEC validation** — cryptographically verifies responses to detect tampering.
- **Security hardening** — refuses non-loopback clients, hides identity/version, and enables a range of `harden-*` protections.
- **Modest footprint** — runs comfortably on small servers and SBCs.

## Real-world performance

These figures are from a live deployment where this resolver sits behind
[AdGuard Home](https://adguard.com/adguard-home/overview.html) as its sole
upstream. Over a 90-day window, Unbound (`127.0.0.1:5335`) handled **656,096
upstream queries** — 100% of them — at an **18 ms average response time**.

![Unbound upstream query count and average response time over 90 days](docs/screenshots/unbound-upstream-stats.png)

## Requirements

- Unbound 1.13 or newer (`apt install unbound`)
- The DNSSEC root trust anchor (`/var/lib/unbound/root.key`)
- A root hints file (`/var/lib/unbound/root.hints`)

This configuration listens on `127.0.0.1:5335`, which is the convention for
running Unbound as the upstream resolver behind Pi-hole or AdGuard Home. Adjust
`interface` and `port` if you intend to use it differently.

> **Tested on:** Debian (stable), Intel Xeon E3-1246 v3, 8 GB RAM. The config is
> not hardware-specific — these are simply the values the cache sizes were tuned
> for. Lower the `*-cache-size` settings on memory-constrained hosts.

## Installation

1. **Install Unbound**

   ```bash
   sudo apt update && sudo apt install unbound
   ```

2. **Fetch the DNSSEC root anchor and root hints**

   ```bash
   sudo unbound-anchor -a /var/lib/unbound/root.key
   sudo curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
   ```

3. **Install the configuration**

   ```bash
   sudo cp unbound.conf /etc/unbound/unbound.conf
   ```

   Review `interface`, `port`, and the `access-control` rules before continuing.

4. **Validate, then restart**

   ```bash
   sudo unbound-checkconf /etc/unbound/unbound.conf
   sudo systemctl restart unbound
   sudo systemctl status unbound
   ```

## Verification

Confirm basic resolution:

```bash
dig @127.0.0.1 -p 5335 example.com
```

Confirm DNSSEC validation is working. A signed, valid domain should return the
`ad` (Authenticated Data) flag:

```bash
dig @127.0.0.1 -p 5335 dnssec.works +dnssec
```

A deliberately broken domain should fail with `SERVFAIL`:

```bash
dig @127.0.0.1 -p 5335 sigfail.verteiltesysteme.net +dnssec
```

## Configuration notes

A few choices in `unbound.conf` are deliberate and worth calling out:

- **`port: 5335`, loopback only** — intended as an upstream resolver behind a
  filtering DNS server. Only `127.0.0.0/8` is allowed; everything else is refused.
- **`do-ip6: no`** — outgoing IPv6 is disabled. Enable it if your host has
  working IPv6 connectivity.
- **`serve-expired: yes`** with a 30-day `serve-expired-ttl` — stale answers can
  be served while a fresh lookup happens in the background, improving resilience
  if upstream is briefly unreachable. Shorten the TTL if you prefer stricter freshness.
- **Cache sizes** are sized for ~8 GB RAM. Reduce `msg-cache-size`,
  `rrset-cache-size`, etc. proportionally on smaller machines.

## Tuning the open-file limit

With `outgoing-range: 16384` across 4 threads, Unbound opens a large number of
sockets. If the service fails to start with an "out of file descriptors" error,
raise the limit in a systemd override:

```bash
sudo systemctl edit unbound
```

```ini
[Service]
LimitNOFILE=65536
```

Then `sudo systemctl daemon-reload && sudo systemctl restart unbound`.

## Troubleshooting

| Symptom | Likely cause |
| --- | --- |
| `unbound-checkconf` errors | Syntax issue or a missing referenced file (`root.key`, `root.hints`) |
| Service won't start | Open-file limit too low (see above) or port `5335` already in use |
| Everything returns `SERVFAIL` | Missing/expired root anchor — re-run `unbound-anchor` |
| No `ad` flag on signed domains | DNSSEC validation not active; check `module-config` and the trust anchor path |

## License

Released under the [MIT License](LICENSE).

Built by SmogCheckFan, 2026
