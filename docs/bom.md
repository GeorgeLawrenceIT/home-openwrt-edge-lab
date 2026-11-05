# Bill of materials

Prices checked around **5 Nov 2026** (Europe/London). Re-check before any further purchase.

## Phase 1 (required)

| Item | Why | Approx. price | Notes |
|------|-----|---------------|-------|
| [GL.iNet GL-MT3000 / Beryl AX](https://www.amazon.co.uk/GL-iNet-GL-MT3000-Portable-Wireless-Cascading/dp/B0BPSGJN7T) | OpenWrt-class edge, Wi-Fi 6, MediaTek | ~£88.99 Amazon UK (ordered) | Also listed on [GL.iNet UK store](https://store-uk.gl-inet.com/products/beryl-ax-gl-mt3000-wi-fi-6-travel-router) (~£69.74 when checked) |
| Existing TP-Link Deco | Room coverage | £0 | Keep separate from OpenWrt mesh |

**Phase 1 total:** one Beryl AX + reuse Deco.

## Phase 2 (deferred — do not buy until Phase 1 README has screenshots)

| Item | Why | Approx. price |
|------|-----|---------------|
| Raspberry Pi Zero 2 (or 4/5) starter kit | Mesh / AP lab node | ~£53+ |
| [MT7612U USB Wi-Fi](https://thepihut.com/products/dual-band-usb-wifi-adapter-for-raspberry-pi-5ghz-2-4ghz-mt7612u) | MediaTek radio for OpenWrt AP/mesh | ~£13.90 |

Avoid Realtek USB Wi-Fi for OpenWrt AP/mesh work.

## Explicitly skipped for Phase 1

- Pi as the main edge (extra NIC/radio cost and SD risk).
- Buying mesh nodes before the edge write-up exists.
