# Beryl AX Router Project: Troubleshooting and Hardening Record

*GL.iNet Beryl AX (GL-MT3000) home-lab deployment: double NAT, WireGuard (PiVPN) and remote printing*

- **Author:** George Lawrence
- **Record compiled:** 10 November 2026
- **Work period:** 6–7 November 2026
- **Status:** Phase 1 working; several follow-up items open (see Section 9)

> **Evidence convention: unless stated otherwise, every action and result in this record was reported by George during the work sessions. Screenshots, configuration exports and command output were not attached to this record, and the router settings have not been independently audited. Items marked 'Unverified' or 'Pending' have not been confirmed.**

## 1. Summary

I replaced the routing role in my home network with a GL.iNet Beryl AX (GL-MT3000) travel router running GL.iNet's OpenWrt-based firmware, while keeping the BT Smart Hub 2 as the DSL modem/router. Because the Smart Hub 2 connects over RJ11 DSL and could not act as a bridge for this setup, I chose a double-NAT design: BT Smart Hub 2 → Beryl AX → home LAN.

The main problems I worked through were:

- Getting my existing PiVPN (WireGuard) server working again behind two layers of NAT. This involved a stale default gateway on the Raspberry Pi, an out-of-date client endpoint, and an upstream block on the BT side.
- Printing to my home printer over the VPN from my Mac while on another network that uses the same 192.168.1.0/24 subnet.

As reported, the WireGuard VPN now works from mobile data and from local tests with the BT firewall on and the Beryl not in the BT DMZ. The printer is reachable over the VPN and a test print job was accepted. Physical printout has not been confirmed. I also reported completing a 12-item router hardening checklist on 6 Nov 2026. Those settings have not been independently audited.

## 2. Scope and safety approach

- Hardware: GL.iNet Beryl AX GL-MT3000. The admin interface reported OpenWrt 21.02-SNAPSHOT. The router was not reflashed or upgraded during this work. The exact GL.iNet firmware version was not recorded (Unverified).
- Isolated baseline first (6 Nov 2026): I configured the Beryl with its WAN disconnected and one computer on the LAN. The household internet connection and the existing TP-Link Deco system were left untouched until the baseline was ready.
- Private configuration backup saved before cutover. It is stored privately and not published.
- Change control: I only moved the internet feed to the Beryl WAN after the baseline was complete.
- Redaction: this record leaves out public IP addresses, WireGuard keys, Wi-Fi network names and passwords, device serials, MAC addresses and personal details. Private (RFC 1918) addresses are included because they explain the design.

## 3. Final topology (as reported)

```text
Internet (DSL, RJ11)
      |
[BT Smart Hub 2]  LAN 192.168.2.254   (BT firewall ON, Beryl NOT in DMZ)
      |   Port forward: UDP 51820 -> 192.168.2.114
      |
[Beryl AX GL-MT3000]  WAN 192.168.2.114 (DHCP reservation on BT hub)
                      LAN 192.168.1.1/24
      |   Port forward: UDP 51820 -> 192.168.1.110
      |   (No WAN port forward for DNS)
      |
      +-- Raspberry Pi 192.168.1.110  (PiVPN/WireGuard + Pi-hole/ad-blocking DNS)
      +-- HP Smart Tank 7000 printer 192.168.1.131
      +-- Other downstream LAN/Wi-Fi devices (incl. Deco)
```

Note: an earlier plan to re-address the BT hub to 192.168.8.1 was dropped. The addresses above are the final ones I reported.

## 4. Timeline and phases

| Phase / date | What I did (reported) | Outcome |
|---|---|---|
| Phase 0: Isolated baseline (6 Nov 2026) | WAN disconnected, one computer on the LAN. Confirmed the firmware shows OpenWrt 21.02-SNAPSHOT. Saved a private configuration backup. Removed SSH rules. Set DHCP reservations/static IPs for the Pi-hole DNS server and PiVPN. Added a WireGuard UDP 51820 forward to 192.168.1.110. No WAN forward for DNS. | Baseline ready; household network not affected |
| Phase 1: WAN cutover (6 Nov 2026) | Decided BT hub bridge mode was not workable (RJ11 DSL), so chose double NAT. Moved the internet feed to the Beryl WAN. Reserved Beryl WAN 192.168.2.114 on the BT hub. | LAN internet via Beryl; VPN not yet working |
| Phase 2: VPN troubleshooting (6 Nov 2026) | Fixed the Pi's stale gateway. Updated the client endpoint. Ran a packet capture. Set up double UDP forwarding. Tested the BT firewall and DMZ. | VPN working with BT firewall ON and no DMZ |
| Phase 3: Remote printing (recorded 7 Nov 2026) | Created a direct IPP queue. Fixed a subnet overlap with an explicit AllowedIPs route. | Printer reachable via VPN; physical output unconfirmed |
| Phase 4: Follow-up (open) | Working-state backup, controlled speed tests, hardening review, possible LAN renumbering, sanitised GitHub write-up. | Pending |

## 5. Troubleshooting log

| # | Symptom | Test / evidence (reported) | Finding | Action | Result |
|---|---|---|---|---|---|
| 1 | BT Smart Hub 2 could not be put in front of the Beryl as a simple modem | Checked the hub's connection type and options | Hub uses RJ11 DSL and cannot act as a bridge for this setup | Chose double NAT. Reserved Beryl WAN 192.168.2.114 on the hub | Working double-NAT topology |
| 2 | WireGuard client could not connect | Client logs showed the handshake never completed | Needed to check the server side and the network path | Investigated the Pi, then the network path | Led to items 3–6 |
| 3 | Pi had no internet after the cutover | Pi default route was stale (192.168.1.254, ARP INCOMPLETE). Ping to 192.168.1.1 worked; internet ping failed | Pi kept a gateway from the old network | Identified the DHCP client in use (see item 4) | Root cause on the Pi found |
| 4 | Unclear how to renew the Pi's DHCP lease | dhcpcd processes were running; dhcpcd.service not found; NetworkManager showed eth0 unmanaged; dhclient not installed | dhcpcd was managing eth0 | Ran sudo dhcpcd -n eth0 | Default gateway became 192.168.1.1; Pi could ping 1.1.1.1 |
| 5 | Client still pointed at the old address | Compared the client endpoint with the current public IP (value redacted) | Endpoint was out of date | Updated the WireGuard client endpoint | Endpoint corrected |
| 6 | Handshake still failing | Ran a targeted capture for UDP 51820 on the Pi (I approved the capture) | No packets reached the Pi, so the block was upstream | Confirmed the protocol is UDP, not TCP. Set up double forwarding: BT UDP 51820 → 192.168.2.114; Beryl UDP 51820 → 192.168.1.110 | Path set up correctly; BT side still in question |
| 7 | Traffic blocked at the BT hub | BT firewall off, or Beryl in BT DMZ → VPN worked | Something on the BT side was blocking the forwarded traffic | Turned the BT firewall back on and took the Beryl out of the DMZ, then retested | Later worked with the BT firewall ON and no DMZ, from mobile data and laptop/local tests. Exact root cause not proven |
| 8 | Printer not usable over VPN from the Mac | Over VPN: ping, printer web interface and port tests all worked | The Bonjour/dnssd queue does not suit WireGuard (no multicast discovery) | Created a direct IPP Everywhere queue (command in Section 7) | Queue created |
| 9 | Printer traffic leaving through the wrong interface | On the grandmother's network (also 192.168.1.0/24), the route to 192.168.1.131 used en0 even with AllowedIPs 0.0.0.0/0, ::/0 | Subnet overlap: the local LAN route took priority | Added an explicit 192.168.1.0/24 to the client AllowedIPs | Route used utun; printer ping worked; 20-page job accepted quickly. Physical output unconfirmed |
| 10 | Beryl Wi-Fi felt slower than the old Deco | Personal observation only | Not measured | None yet | Controlled wired/wireless comparison pending |

## 6. Router and Wi-Fi hardening

> **Status: checklist George reported complete on 6 Nov 2026; not independently audited. No item below has been validated.**

On 6 Nov 2026, before the WAN cutover, I first asked whether I could harden the router offline. I was told most core hardening could be done with the WAN unplugged. I then asked for a list of hardening steps and replied "okay thats all done" to the 12-item checklist below. This is my report of completion. The router configuration has not been reviewed or tested against each item.

### 6.1 Checklist reported complete (6 Nov 2026)

| # | Checklist item (as provided) | Status |
|---|---|---|
| 1 | Use a unique, password-manager-generated admin password, not reused elsewhere. | Reported complete; not audited |
| 2 | Disable WAN administration: LuCI/HTTP/HTTPS/SSH not exposed on the WAN; admin from the LAN only. | Reported complete; not audited |
| 3 | Disable SSH unless needed (use keys later if SSH is needed). | Reported complete; not audited |
| 4 | Keep vendor remote management/support disabled unless deliberately needed. | Reported complete; not audited |
| 5 | Disable UPnP if automatic inbound mappings are not needed; confirm the WireGuard rule remains. | Reported complete; not audited |
| 6 | Secure guest network kept separate from the LAN, WPA2/WPA3, unique password. | Reported complete; not audited |
| 7 | Review firewall forwarding: only the intended WireGuard forward; never forward 53/80/443/22 for convenience. | Reported complete; not audited |
| 8 | Reserve IPs for critical devices and record the private address, UDP port and gateway privately. | Reported complete; not audited |
| 9 | Turn off unused router services/add-ons (e.g. router VPN, ad-blocking, file sharing) to avoid overlapping controls. | Reported complete; not audited |
| 10 | Plan firmware updates deliberately; no flashing in that session. | Reported complete; not audited (firmware not upgraded) |
| 11 | Validate before and after the WAN cutover. | Reported complete; not audited (no validation records attached) |
| 12 | Record sanitised evidence; do not upload configuration archives that contain secrets. | Reported complete; not audited (GitHub write-up not yet committed) |

### 6.2 Items I specifically confirmed

Beyond the general "all done", I specifically confirmed the following on 6 Nov 2026. These are still my reports and have not been independently checked:

- Private configuration backup saved (taken before the cutover; stored privately, not published).
- SSH rules removed (relates to items 2–3; the exact rules were not recorded).
- Static IPs/DHCP reservations set for the Pi-hole DNS server and PiVPN (item 8).
- Port forwarding set up for PiVPN: WireGuard UDP 51820 → 192.168.1.110 (item 7). I later confirmed there is no WAN port forward for DNS.

Related final-state detail (reported): the BT Smart Hub 2 firewall is ON and the Beryl is not in the BT DMZ. The DMZ and firewall-off tests during VPN troubleshooting were temporary diagnostics.

Still not done or unverified: an independent review of every checklist item, a WAN-side exposure scan, a check for newer firmware, a fresh post-hardening backup, and a sanitised written record of the final Wi-Fi security settings.

## 7. Commands recorded

These are the commands I reported using. Output is not reproduced because it was not supplied for this record.

```text
# Renew the DHCP lease on the Raspberry Pi (dhcpcd managing eth0)
sudo dhcpcd -n eth0

# macOS: direct IPP Everywhere print queue to the home printer over WireGuard
sudo lpadmin -p HP_Tank_VPN -E -v ipp://192.168.1.131/ipp/print -m everywhere
```

WireGuard client change (keys and endpoint removed): AllowedIPs = 0.0.0.0/0, ::/0, plus an explicit 192.168.1.0/24 entry.

## 8. Validation and current state

| Item | Current state | Evidence level |
|---|---|---|
| Internet via Beryl (double NAT) | Working | George-reported |
| Pi default gateway 192.168.1.1; Pi internet reachability | Working (ping to 1.1.1.1) | George-reported command result |
| WireGuard from mobile data | Working, BT firewall ON, no DMZ | George-reported |
| WireGuard from laptop/local tests | Working | George-reported |
| Printer over VPN (ping, web UI, ports) | Working | George-reported |
| Printing via IPP queue | Job accepted (20 pages) | Physical output unconfirmed |
| Hardening checklist | Reported complete | George-reported (12-item list, 6 Nov 2026); not independently audited |
| Wi-Fi performance vs. Deco | Subjectively slower | No controlled measurement |
| Working-state backup | Not confirmed | Pending |

## 9. Outstanding actions

- Take and store a fresh private configuration backup of the working state (Beryl, plus Pi/PiVPN as appropriate).
- Review the hardening settings independently (e.g. a second pass through the admin UI and a WAN-side exposure check) and write the confirmed settings down.
- Run controlled speed tests: wired vs. wireless, Beryl vs. Deco, same device, location and time window.
- Confirm a physical printout from the VPN print queue.
- Permanent fix for the subnet overlap (proposed, not done): after checks, renumber the home LAN to an uncommon private subnet such as 192.168.73.0/24, then update DHCP, reservations, the Pi/Pi-hole, port forwarding and the VPN configuration.
- Investigate the BT-side block if it comes back. The cause is currently unknown.
- Publish a sanitised write-up to GitHub (not yet committed), with no keys, public IPs, network names or personal details.

## 10. Lessons learned

- Isolating the new router (WAN unplugged, one LAN client) let me build the configuration without disrupting the household.
- After a gateway change, check every static or long-lease device. The Pi held on to the old 192.168.1.254 gateway.
- Identify which network service actually manages an interface (dhcpcd, NetworkManager or dhclient) before trying to renew a lease.
- A targeted packet capture on the server showed quickly whether traffic was arriving, and so whether to look upstream or at the server.
- With double NAT, each layer needs its own forward, and the protocol must match (WireGuard uses UDP).
- Use DMZ or firewall-off tests only as short diagnostics, and revert them afterwards.
- Overlapping private subnets break VPN access to remote LAN devices. A specific route (192.168.1.0/24) beats a default route, but renumbering is the cleaner fix.
- Bonjour discovery does not cross a routed WireGuard tunnel. A direct IPP queue avoids the problem.

## 11. Evidence and limitations

- This record is based on what I reported in chat sessions on 6–7 November 2026. No screenshots, configuration exports or raw logs are attached.
- The hardening checklist in Section 6 is the list I reported completing. No item has been validated by configuration review, scan or screenshot.
- The root cause of the BT-side block was not proven. A state refresh on the hub is one possibility, not a conclusion.
- The GL.iNet firmware version was not recorded. 'OpenWrt 21.02-SNAPSHOT' is what the interface showed.
- Speed observations are subjective. Earlier home Wi-Fi speed-map readings are user-reported Wi-Fi measurements, not line-speed evidence, and are left out.
- Redacted from this record: public IP addresses, WireGuard keys, network names and passwords, serials, MAC addresses and personal details.
