---
name: factory-network-upgrade
description: U6 Pro + UCG-Ultra + WireGuard project to fix plant Wi-Fi and retire Tailscale; site facts, reservation table, cutover sequence
metadata:
  type: project
---

Started 2026-09-09. **2026-09-15: UCG-Ultra (console GMS-Bangi, WAN 192.168.0.192 behind the SR1041F) + U6 Pro adopted and updated; test SSID GMS-SETUP live; WireGuard server GMS-Bangi-VPN (51820, 192.168.2.0/24) with phone peer 192.168.2.2 scanned; laptop peer, DDNS, port forward and the LAN cutover still pending.** UniFi console reachable at unifi.ui.com (Danial signs in himself). Full plan + DHCP reservation table + cutover steps live in the vault: `Documents/Dantec/DANOTTO/Projects/Factory Network & VPN Upgrade.md` (plain copy `Documents/Dantec/Factory-Network-Plan.md`).

Site facts found 2026-09-15: internet is TM Unifi with a real public IPv4 (1.32.46.x, NOT carrier NAT). Current router is the TM-supplied TP-Link **SR1041F** at 192.168.0.1 (admin user `tmadmin`; it does the TM PPPoE login itself (user `globalmanufacturin11@unifibiz`, VLAN 500, MTU 1492), WAN = public IP, the ZTE fibre box is a bridge; only LAN2 is cabled, so all wired devices hang off a switch on LAN2) (SSIDs `GMSADM@unifi_2.4G`/`_5G`, WPA2) plus a TP-Link extender at .112. DHCP pool ~.100–.199, nothing reserved, so line-PC Ethernet IPs drift. Line PC MACs (Ethernet): L1 .141 FC-9D-05-76-5C-9C, L2 .137 FC-9D-05-76-5C-9D, L3 .122 FC-9D-05-84-17-68, L4 .144 50-91-E3-71-52-44, L5 .121 FC-9D-05-76-5C-A3; Parts PC Ethernet .140 94-C6-91-6F-87-F6; robot .115 00-26-C6-86-0E-E8.

**Why:** the U6 Pro alone does not fix anything; the fault class is hand-set Wi-Fi + unreserved DHCP. The Pi WhatsApp bridge is NOT on the plant LAN (reached only via Tailscale) so retiring Tailscale needs the Pi to become a WireGuard peer of the plant gateway.

**How to apply:** Phase 1 = UCG behind the TM router (double NAT, TM LAN moved to 192.168.100.x, UCG owns 192.168.0.1/24, U6 Pro broadcasts the SAME SSIDs/password so no device is touched), done in an idle window. Phase 2 VLANs, Phase 3 Pi peer + Tailscale removal. Keep Tailscale two weeks after cutover. Related: [[remote-access-from-home]], [[nas-daiya-line-monitor]], [[whatsapp-bridge-pi]], [[pvs-db-failsafe]], [[pvs-deploy-when-line-idle]].
