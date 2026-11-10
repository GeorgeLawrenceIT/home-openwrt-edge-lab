# Harden checklist — Phase 1

This page is a planning template. A separate George-reported session note, which does not verify these items, is in [beryl-ax-troubleshooting-and-hardening-record.md](beryl-ax-troubleshooting-and-hardening-record.md).

Fill **Done**, **Verified how**, and **Rollback** as each step is completed. Prefer commands or LuCI paths you actually used.

## 1. Day-one baseline

- [ ] WAN to modem, one PC on LAN  
- [ ] LAN subnet changed if it clashes with ISP  
- [ ] Strong root password set  
- [ ] Packages updated  
- [ ] First `sysupgrade -b` / LuCI backup saved offline  

**Verified how:**  
**Rollback:**  

## 2. Lock remote admin

- [ ] LuCI and SSH **not** listening / allowed on WAN  
- [ ] Prefer HTTPS for LuCI on LAN  
- [ ] SSH public key installed; password login disabled  
- [ ] Recovery path written **before** disabling password auth  

**Verified how:** (e.g. WAN-side scan / phone on mobile data)  
**Rollback:**  

## 3. Firewall literacy

- [ ] Document wan / lan / guest default input/output/forward  
- [ ] One intentional accept rule + one reject rule  
- [ ] Before/after ping or curl evidence  

**Verified how:**  
**Rollback:**  

## 4. Guest network

- [ ] Separate SSID + interface + firewall zone  
- [ ] Guest cannot reach LAN printers/NAS/management IP  
- [ ] Client isolation on  

**Verified how:**  
**Rollback:**  

## 5. DNS filtering

Pick one: AdGuard Home / https-dns-proxy (DoH) / SmartDNS.

- [ ] Filter active for LAN clients  
- [ ] Blocked-domain test recorded  
- [ ] Bypass procedure for troubleshooting documented  

**Verified how:**  
**Rollback:**  

## 6. SQM / performance

- [ ] cake on WAN sized to real upstream  
- [ ] Bufferbloat test before and after (screenshots in `assets/`)  

**Verified how:**  
**Rollback:**  

## 7. VPN (WireGuard)

- [ ] Peer for phone or site-to-site  
- [ ] Key hygiene notes (private keys never in git)  
- [ ] AllowedIPs and tunnel-down behaviour documented  
- [ ] Optional: policy routing for selected clients  

**Verified how:**  
**Rollback:**  

## 8. Threat hygiene

- [ ] BanIP (or similar) with a small list set  
- [ ] Logging enabled; one practice log review  

**Verified how:**  
**Rollback:**  

## 9. Backup / restore drill

- [ ] sysupgrade backup taken after harden baseline  
- [ ] Restore drill performed (lab or documented dry-run)  
- [ ] Steps written so a stranger could follow  

**Verified how:**  
**Rollback:**  

## Incident log (fill one)

When something breaks, add a short entry: symptom → log lines → fix → prevention. That is strong helpdesk evidence.
