# Segmented Home Lab

A segmented network built on a Cisco 1921 router and a Cisco 2960 switch. It separates servers, clients and IoT devices into three VLANs, routes between them with an access control list, and carries a two-host Proxmox cluster that runs an Active Directory domain.

![Segmented network architecture](infographic-segmented-network.png)

## Design

| VLAN | Name | Subnet | Purpose |
|---|---|---|---|
| 10 | SERVERS | 192.168.x.x/24 (static) | Domain controller, Proxmox hosts, file, print and backup servers |
| 20 | CLIENTS | 10.x.20.x/24 (DHCP) | Client machines, resolves the domain through the domain controller |
| 30 | IOT | 10.x.30.x/24 (DHCP) | Wireless access point and IoT devices, internet only |
| 999 | PARKING | none | Unused switch ports, shut down |

- **Routing:** the 1921 routes between VLANs using 802.1Q subinterfaces on one trunk.
- **Access control:** an extended ACL on the IoT subinterface denies IoT traffic to the SERVERS and CLIENTS networks and permits everything else.
- **Addressing:** the router is the DHCP server for CLIENTS and IOT. SERVERS use static addresses.
- **Internet:** the 1921 translates all three subnets through its WAN address (NAT overload).

## Stages

| Stage | Scope | Result |
|---|---|---|
| 1 | Build the switch and router offline | VLANs, trunk, subinterfaces, ACL, DHCP and NAT configured |
| 2 | Core connectivity | Trunk up, internet reachable, laptop receives DHCP on VLAN 20 |
| 3 | Proxmox host 1 and the domain controller on VLAN 10 | Host and domain controller reachable, zeroday.local resolves |
| 4 | Proxmox host 2 and the servers on VLAN 10 | Three servers reachable; file share opens and the Finance share is denied |
| 5 | TP-Link AX-1800 as an access point | IoT network on VLAN 30, isolated from the lab networks |

The full configurations, step-by-step procedures, test tables and troubleshooting for every stage are in the build guide:

- [Segmented_Lab_Stage1_to_Stage5.docx](Segmented_Lab_Stage1_to_Stage5.docx)

## Troubleshooting highlights

- **Proxmox web UI unreachable from another VLAN:** the node firewall needs rules for the source subnet of that VLAN (10.x.20.x/24 for CLIENTS).
- **Domain controller has no gateway or internet:** the VM-level firewall on the NIC dropped ARP and ICMP. Turning it off on the NIC fixed the path.
- **Access point not routing:** the AP must be in Access Point mode with a static management address, or it runs its own DHCP and NAT.

## Addressing note

The addresses in this repository are placeholders. Replace them with your own plan before you build. Keep your real addresses in a private copy of the build guide.

## Requirements

- Cisco 1921 router and Cisco 2960 switch (or Packet Tracer for a simulation)
- A TP-Link AX-1800, or another access point with an Access Point mode
- Two Proxmox VE hosts
- Windows Server 2025 evaluation for the domain and server VMs

## Demonstrated skills

- **Cisco IOS:** VLANs, 802.1Q trunking, router-on-a-stick, DHCP, NAT overload, extended ACLs
- **Proxmox VE:** VLAN-aware networking, host and VM firewall troubleshooting
- **Active Directory:** domain services, DNS, server joins, OU design
- **Troubleshooting:** layer-by-layer diagnosis of ARP, routing and firewall issues
- **Documentation:** step-by-step build guides with test tables and rollback plans

## Related projects

- **[Active Directory lab (AD-WS-Lab)](https://github.com/zdhomelab/AD-WS-Lab):** Windows Server Active Directory lab covering domain services, user and group management, and Group Policy.
- **[AD server lab (ad-server-lab)](https://github.com/zdhomelab/ad-server-lab):** Windows Server domain lab with a file server using AGDLP permissions, a GPO-deployed printer, and a System State backup with an AD Recycle Bin restore test.
- **[Pi-hole DNS lab (Pi-Hole-DNS-Lab)](https://github.com/zdhomelab/Pi-Hole-DNS-Lab):** Network-wide DNS filtering and monitoring with Pi-hole, including custom local records and client configuration.
- **[Wazuh SIEM detection lab (Wazuh-SIEM-Detection-Lab)](https://github.com/zdhomelab/Wazuh-SIEM-Detection-Lab):** Security monitoring with Wazuh, covering agent deployment, log collection, and detection rules with alerts for suspicious activity.

## Disclaimer

These are personal lab environments built for learning and testing. They are not production configurations. Addresses, host names and credentials in the documentation are placeholders. Do not apply any of these configurations to a live network without testing and review.
