# ASN Plan

This document coiuntains the ASN allocation and BGP peering relationships for the Check Point R82 topology.

---

## ASN Assignments

| ASN   | Organization Type | Description                         | Devices                |
|-------|-------------------|-------------------------------------|------------------------|
| 2000  | Service Provider  | Upstream transit provider backbone  | PE1, PE2               |
| 3481  | Enterprise Hub    | Central security hub with HA        | GW-HUB1, GW-HUB2, R2-H |

Branch A and Branch B are VPN-only sites, no BGP used.

---

## BGP Peering

### eBGP Sessions (PE-CE)

| Local Router | Local AS | Remote Router | Remote AS | Neighbor IP  | Interface |
|--------------|----------|---------------|-----------|--------------|-----------|
| GW-HUB1      | 3481     | PE1           | 2000      | 192.168.0.9  | eth0      |
| GW-HUB1      | 3481     | PE2           | 2000      | 192.168.0.13 | eth1      |
| GW-HUB2      | 3481     | PE1           | 2000      | 192.168.0.17 | eth0      |
| GW-HUB2      | 3481     | PE2           | 2000      | 192.168.0.21 | eth1      |

---

## OSPF Areas

| Site | OSPF Area | Devices                | Purpose                            |
|------|-----------|------------------------|------------------------------------|
| Hub  | Area 0    | GW-HUB1, GW-HUB2, R2-H | Backbone area for internal routing |

**Redistribution:**
- **BGP to OSPF:** Default route (0.0.0.0/0) redistributed into OSPF at hub gateways so R2-H learns internet paths
- **OSPF to BGP:** Hub internal networks (10.100.0.0/16) redistributed into BGP for advertisement to PE routers
- **Static to OSPF:** Branch aggregates (10.10.0.0/16, 10.20.0.0/16) redistributed from VPN static routes