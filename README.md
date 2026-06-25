# Unbound DNS Configuration

Production-grade Unbound DNS resolver with recursive resolution, caching, and DNSSEC validation.
Tested and running stable on modest hardware.

## Features

✅ **Recursive Resolution** — Queries upstream nameservers to resolve domains
✅ **Caching** — Stores results locally for fast repeat lookups
✅ **DNSSEC Validation** — Ensures DNS responses are authentic and tamper-proof
✅ Set-and-forget operation — Requires minimal maintenance
✅ Low resource footprint — Runs smoothly on modest hardware

## Hardware

- **CPU:** Intel Xeon E3-1246 v3
- **RAM:** 8 GB
- **OS:** Debian Stable

## Usage

1. Copy `unbound.conf` to your server
2. Adjust IP addresses and interfaces as needed
3. Restart Unbound
4. Done — No further maintenance needed

## Production Tested

Running stable in production. Zero issues with this configuration.

## License

Public Domain

---

Built by SmogCheckFan, 2026
