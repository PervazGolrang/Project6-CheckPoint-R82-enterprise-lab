# IP Address Plan

This is a complete addressing scheme that use /31 for point-to-point links where possible, /32 for loopbacks, /30 for P2P where /31 isn't supported, and RFC 1918 private address space for all networks.

---

## Global Address Blocks

| Block        | Range           | Purpose                                |
|--------------|-----------------|----------------------------------------|
| Loopbacks    | 10.255.0.0/24   | Router-IDs and BGP peering             |
| PE-CE Links  | 192.168.0.0/24  | Service provider customer-facing links |
| Oslo LAN     | 10.10.0.0/16    | Branch A internal networks             |
| Bergen LAN   | 10.20.0.0/16    | Branch B internal networks             |
| Hub LAN      | 10.100.0.0/16   | Hub internal, DMZ, management          |
| VPN Tunnels  | 172.31.0.0/24   | IPsec overlay addressing               |
| Management   | 192.168.40.0/24 | Out-of-band management (home network)  |

---

## Loopback Addresses

| Device    | Loopback0     | Router-ID  | ASN  | Notes                   |
|-----------|---------------|------------|------|-------------------------|
| PE1       | 10.255.0.1/32 | 10.255.0.1 | 2000 | Service provider edge   |
| PE2       | 10.255.0.2/32 | 10.255.0.2 | 2000 | Service provider edge   |
| GW-HUB1   | 10.255.3.1/32 | 10.255.3.1 | 3481 | Check Point R82 cluster |
| GW-HUB2   | 10.255.3.2/32 | 10.255.3.2 | 3481 | Check Point R82 cluster |
| R2-H      | 10.255.3.3/32 | 10.255.3.3 | 3481 | Internal hub router     |
| GW-Oslo   | 10.255.1.1/32 | 10.255.1.1 | —    | Check Point R81.20      |
| R1-A      | 10.255.1.2/32 | 10.255.1.2 | —    | Branch A router         |
| GW-Bergen | 10.255.2.1/32 | 10.255.2.1 | —    | Check Point R81.20      |
| R3-B      | 10.255.2.2/32 | 10.255.2.2 | —    | Branch B router         |

---

### PE-CE Links (eBGP)

| Link          | Device A | Interface A  | IP A         | Device B  | Interface B | IP B         | Subnet          | Notes        |
|---------------|----------|--------------|--------------|-----------|-------------|--------------|-----------------|--------------|
| PE1 ↔ GW-HUB1 | PE1      | Gi0/0        | 192.168.0.9  | GW-HUB1   | eth0        | 192.168.0.10 | 192.168.0.8/30  | Primary path |
| PE2 ↔ GW-HUB1 | PE2      | Gi0/0        | 192.168.0.13 | GW-HUB1   | eth1        | 192.168.0.14 | 192.168.0.12/30 | Backup path  |
| PE1 ↔ GW-HUB2 | PE1      | Gi0/1        | 192.168.0.17 | GW-HUB2   | eth0        | 192.168.0.18 | 192.168.0.16/30 | Primary path |
| PE2 ↔ GW-HUB2 | PE2      | Gi0/1        | 192.168.0.21 | GW-HUB2   | eth1        | 192.168.0.22 | 192.168.0.20/30 | Backup path  |

---

## Hub Site (ASN 3481 - OSPF Area 0)

### ClusterXL Sync Network (Private)

| Link                   | Device A | Interface A | Device B | Interface B | Notes             |
|------------------------|----------|-------------|----------|-------------|-------------------|
| GW-HUB1 ↔ GW-HUB2 Sync | GW-HUB1  | eth2        | GW-HUB2  | eth2        | Bonded into bond1 |
| GW-HUB1 ↔ GW-HUB2 Sync | GW-HUB1  | eth3        | GW-HUB2  | eth3        | Bonded into bond1 |
| GW-HUB1 ↔ GW-HUB2 Sync | GW-HUB1  | eth5        | GW-HUB2  | eth5        | Bonded into bond1 |

### Internal Hub Links (OSPF)

| Link           | Device A | Interface A | IP A          | Device B | Interface B | IP B         | Subnet          | Notes        |
|----------------|----------|-------------|---------------|----------|-------------|--------------|-----------------|--------------|
| GW-HUB1 ↔ R2-H | GW-HUB1  | eth4        | 10.100.254.1  | R2-H     | Gi0/0       | 10.100.254.2 | 10.100.254.0/30 | OSPF Area 0  |
| GW-HUB2 ↔ R2-H | GW-HUB2  | eth4        | 10.100.254.5  | R2-H     | Gi0/1       | 10.100.254.6 | 10.100.254.4/30 | OSPF Area 0  |
| R2-H ↔ DMZ-SW  | R2-H     | Gi0/2       | 10.100.10.254 | DMZ-SW   | Gi0/0       | —            | 10.100.10.0/24  | DMZ subnet   |

### DMZ Services

| Device     | Interface | IP Address    | Subnet         | Gateway       | Services        |
|------------|-----------|---------------|----------------|---------------|-----------------|
| Cisco ISE  | eth0      | 10.100.10.10  | 10.100.10.0/24 | 10.100.10.254 | TACACS+, RADIUS |

---

## VPN Tunnels (Site-to-Site)

### Oslo to Hub

| Tunnel Name      | Local Device | Local Interface | Local IP   | Remote Device | Remote IP     | Tunnel Subnet   |
|------------------|--------------|-----------------|------------|---------------|---------------|-----------------|
| Oslo-Hub-Primary | GW-Oslo      | eth1            | 172.31.1.1 | GW-HUB1       | 172.31.1.2    | 172.31.1.0/30   |
| Oslo-Hub-Backup  | GW-Oslo      | eth2            | 172.31.1.5 | GW-HUB2       | 172.31.1.6    | 172.31.1.4/30   |

### Bergen to Hub

| Tunnel Name        | Local Device | Local Interface | Local IP   | Remote Device | Remote IP  | Tunnel Subnet  |
|--------------------|--------------|-----------------|------------|---------------|------------|----------------|
| Bergen-Hub-Primary | GW-Bergen    | eth1            | 172.31.2.1 | GW-HUB2       | 172.31.2.2 | 172.31.2.0/30  |

---

## Branch A - Oslo

### Internal Links

| Link           | Device A | Interface A | IP A        | Device B | Interface B | IP B         | Subnet         |
|----------------|----------|-------------|-------------|----------|-------------|--------------|----------------|
| GW-Oslo ↔ R1-A | GW-Oslo  | eth0        | 10.10.254.1 | R1-A     | Gi0/0       | 10.10.254.2  | 10.10.254.0/30 |

### Branch A LANs

| Network       | Gateway    | DHCP Pool        | Static Hosts          |
|---------------|------------|------------------|-----------------------|
| 10.10.10.0/24 | 10.10.10.1 | 10.10.10.100-199 | Desktop1: 10.10.10.10 |
| 10.10.99.0/24 | 10.10.99.1 | —                | —                     |

---

## Branch B - Bergen

### Internal Links

| Link              | Device A  | Interface A | IP A         | Device B | Interface B | IP B         | Subnet         |
|-------------------|-----------|-------------|--------------|----------|-------------|--------------|----------------|
| GW-Bergen ↔ R3-B  | GW-Bergen | eth0        | 10.20.254.1  | R3-B     | Gi0/0       | 10.20.254.2  | 10.20.254.0/30 |

### Branch B LANs

| Network       | Gateway    | DHCP Pool        | Static Hosts          |
|---------------|------------|------------------|-----------------------|
| 10.20.20.0/24 | 10.20.20.1 | 10.20.20.100-199 | Desktop2: 10.20.20.10 |
| 10.20.99.0/24 | 10.20.99.1 | —                | —                     |

---

## Out-of-Band Management (Physical Access)

| Device    | OOB Interface | IP Address       | Gateway      | Notes                   |
|-----------|---------------|------------------|--------------|-------------------------|
| GW-HUB1   | Mgmt          | 192.168.40.223   | 192.168.40.1 | Direct to home network  |
| GW-HUB2   | Mgmt          | 192.168.40.224   | 192.168.40.1 | Direct to home network  |
| GW-Oslo   | Mgmt          | 192.168.40.221   | 192.168.40.1 | Direct to home network  |
| GW-Bergen | Mgmt          | 192.168.40.222   | 192.168.40.1 | Direct to home network  |
| SMS       | Mgmt          | 192.168.40.250   | 192.168.40.1 | Direct to home network  |

---

## ClusterXL Virtual IPs (Cluster VIPs)

| Cluster Object | VIP            | Member 1 IP    | Member 2 IP    | Subnet          | Interface | Notes              |
|----------------|----------------|----------------|----------------|-----------------|-----------|--------------------|
| HUB-Cluster    | 10.100.254.10  | 10.100.254.1   | 10.100.254.5   | 10.100.254.0/24 | eth4      | Internal side      |
| HUB-Cluster    | 192.168.0.100  | 192.168.0.10   | 192.168.0.18   | 192.168.0.0/24  | eth0      | PE1 side (primary) |
| HUB-Cluster    | 192.168.0.101  | 192.168.0.14   | 192.168.0.22   | 192.168.0.0/24  | eth1      | PE2 side (backup)  |