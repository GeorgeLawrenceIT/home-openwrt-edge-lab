# Architecture — Phase 1

## Logical layout

```text
[ISP modem / ONT]
        │
        │ Ethernet (WAN)
        ▼
┌───────────────────────────────┐
│  GL.iNet GL-MT3000 (Beryl AX) │
│  OpenWrt edge                 │
│  - Firewall: wan / lan / guest│
│  - SQM (cake) on WAN          │
│  - DNS filter on LAN          │
│  - WireGuard (optional peer)  │
└───────────────┬───────────────┘
                │ LAN Ethernet
        ┌───────┴────────┐
        ▼                ▼
   Deco unit(s)     Wired PC / lab host
   (AP mode /       OpenWrt SSIDs
    wired BH)       Guest SSID (isolated)
```

## Zones (target)

| Zone | Purpose | Forward |
|------|---------|---------|
| `wan` | Internet | Reject inbound; allow established |
| `lan` | Trusted home / management | To wan; not from guest |
| `guest` | Visitors / untrusted phones | To wan only; no lan |

## Deco coexistence

- Deco proprietary mesh does **not** join OpenWrt 802.11s.
- Preferred Phase 1: modem → OpenWrt → Ethernet into Deco as AP / wired backhaul.
- Write which SSIDs phones stay on during hardening; migrate gradually.

## What success looks like

1. One PC on OpenWrt LAN with internet.
2. No LuCI/SSH reachable from WAN.
3. Guest client cannot ping LAN management IP.
4. Bufferbloat grade improved after SQM (screenshot in `assets/`).
5. Backup `.tar` restore drill documented.
