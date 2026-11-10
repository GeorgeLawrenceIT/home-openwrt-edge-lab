# Troubleshooting notes

A George-reported record of the 6–7 November 2026 Beryl AX work is in [beryl-ax-troubleshooting-and-hardening-record.md](beryl-ax-troubleshooting-and-hardening-record.md). It is not an independent audit. This page stays the incident template.

Add real incidents here after go-live. Template:

## YYYY-MM-DD — short title

- **Symptom:**
- **Impact:**
- **Evidence:** (log lines, Luci status — redact secrets)
- **Cause:**
- **Fix:**
- **Prevention / doc update:**

---

## Known Phase 1 gotchas (pre-filled)

1. **Deco will not mesh with OpenWrt** — keep Deco as AP / wired backhaul only.  
2. **LAN subnet clash with ISP** — change OpenWrt LAN before blaming WAN.  
3. **Lockout after disabling SSH password** — keep a wired console/recovery plan written before the change.  
4. **DNS filter “site won’t load” tickets** — document allowlist / temporary bypass (classic service-desk skill).  
5. **SQM set too low** — hurts speed; size to measured upstream, not marketing rate.
