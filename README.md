# Home OpenWrt edge lab — Phase 1

Hardened OpenWrt home edge: config, firewall, guest Wi-Fi, DNS filtering, SQM, and WireGuard — documented for IT support / service desk interview evidence.

**Status:** Phase 1 scaffold (hardware ordered; flash and harden notes to be filled as work is done).  
**Hardware:** GL.iNet GL-MT3000 (Beryl AX) as the edge router. Existing TP-Link Deco kept for coverage (not meshed with OpenWrt).  
**Owner:** George Lawrence — UK entry-level IT support / junior cyber portfolio project.

---

## Problem

Home Wi-Fi had coverage gaps and a consumer mesh that did not teach real router/security practice. Goal: replace the main edge with a supported OpenWrt device, learn production-like hardening on a budget (~£70–£89), and publish a clear write-up recruiters can skim.

## Why not a Pi as the edge (yet)

A Pi-as-router needs extra NICs/radios, risks SD-card wear, and fights Phase 1 affordability. The GL-MT3000 already runs OpenWrt-class firmware, has proper WAN+LAN, Wi-Fi 6, and MediaTek drivers that behave. Same skills (uci, firewall zones, LuCI, WireGuard) — chosen the way a service desk would push known-good hardware. Pi mesh nodes are **Phase 2** only after this README has screenshots.

## Architecture (Phase 1)

```text
ISP modem
    │
    ▼
GL-MT3000 (OpenWrt edge)  ← WAN admin locked; SSH key-only; firewall zones
    │
    ├── LAN / Ethernet ──► Deco (AP / wired backhaul — separate proprietary mesh)
    ├── OpenWrt SSID(s) ──► trusted clients
    └── Guest SSID ──────► isolated zone (no LAN printers/NAS)
```

Deco **cannot** peer with OpenWrt 802.11s. Coexistence story: keep phones on Deco while hardening, or migrate room-by-room onto OpenWrt SSIDs.

See [docs/architecture.md](docs/architecture.md).

## Bill of materials

| Item | Role | Approx. GBP | Phase |
|------|------|-------------|-------|
| GL.iNet GL-MT3000 (Beryl AX) | OpenWrt edge | ~£70–£89 (ordered Amazon UK ~£88.99, Nov 2026) | 1 — required |
| Existing TP-Link Deco | Coverage / wired AP | £0 (reuse) | 1 |
| Pi Zero 2 kit + MT7612U | Mesh / AP lab node | ~£67+ | 2 — deferred |

Links and notes: [docs/bom.md](docs/bom.md).

## Harden checklist

Work in order. Each step gets: goal → LuCI/uci path → verification → rollback.

| # | Step | Done? | Notes |
|---|------|-------|-------|
| 1 | Day-one baseline (WAN/LAN, root password, packages, first backup) | ☐ | |
| 2 | Lock remote admin (no WAN LuCI/SSH; HTTPS LuCI; SSH keys; no password auth) | ☐ | |
| 3 | Firewall literacy (wan / lan / guest zones + one allow + one reject with tests) | ☐ | |
| 4 | Guest Wi-Fi + client isolation | ☐ | |
| 5 | DNS filtering (AdGuard Home / https-dns-proxy / SmartDNS — pick one) | ☐ | |
| 6 | SQM cake on WAN + bufferbloat before/after | ☐ | |
| 7 | WireGuard (phone-in or site-to-site) + kill-switch notes | ☐ | |
| 8 | BanIP (or similar) + log review habit | ☐ | |
| 9 | sysupgrade backup / restore drill written up | ☐ | |

Detail: [docs/harden-checklist.md](docs/harden-checklist.md).

A George-reported session record (not an independent audit, and not a verification of the checklist above) is in [docs/beryl-ax-troubleshooting-and-hardening-record.md](docs/beryl-ax-troubleshooting-and-hardening-record.md).

## Interview STAR (short)

- **Situation:** Career switch into IT support; home lab needed real edge security practice, not just consumer mesh.
- **Task:** Deliver a stable, documented OpenWrt edge on a tight budget.
- **Action:** Chose supported MediaTek gear; segmented guest; DNS filter; SQM; VPN; documented rollback.
- **Result:** *(fill after go-live — stable edge, public README, measured bufferbloat improvement).*

## Phase 2 (explicitly not done)

Pi nodes + MT7612U, 802.11s / mesh11sd / batman-adv between OpenWrt devices only — only after Phase 1 is online with screenshots. 802.11ah / HaLow stays a separate project.

## Repo layout

```text
README.md                 ← you are here
docs/architecture.md
docs/bom.md
docs/harden-checklist.md
docs/troubleshooting.md
docs/beryl-ax-troubleshooting-and-hardening-record.md
config-examples/          ← sanitized uci snippets later (no secrets)
assets/                   ← screenshots / bufferbloat graphs later
LICENSE
```

## Safety

- Never commit passwords, WireGuard private keys, or full unredacted configs.
- Prefer screenshots and sanitized snippets.
- Recovery path if locked out of SSH/LuCI is written in the harden checklist before disabling password auth.

## Licence

MIT — see [LICENSE](LICENSE).
