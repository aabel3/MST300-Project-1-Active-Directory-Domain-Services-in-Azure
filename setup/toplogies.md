# Network Topology
```
MST300-project1-rg
├── MST300-vnet1 (Domain Controller Network)
│   ├── vnet1-subnet1 (Domain Controller)
│   └── AzureBastionSubnet (Bastion Host)
├── MST300-vnet2 (Webserver Network)
│   └── vnet2-subnet1 (Webserver)
└── MST300-vnet3 (Client Network)
    └── vnet3-subnet1 (Client)
```
# Virtual Machines Topology
| VM Name | OS | Purpose | Network |
|---------|-----|---------|---------|
| dc-vm | Windows Server 2019 | Domain Controller | MST300-vnet1 |
| webserver-vm | Windows Server 2019 | IIS Web Server | MST300-vnet2 |
| client-vm | Windows 10 Pro | Client Workstation | MST300-vnet3 |

# Domain Configuration
- **Domain Name:** `studentID.MST300.com` (replace `studentID` with your actual student ID)
- **Domain Admin:** `studentID.admin`
- **Domain User:** `studentID`

# Subnet Calculation Table Topology

| Subnet Name | Network Address | Usable Range | Broadcast | CIDR | Hosts |
|-------------|----------------|--------------|-----------|------|-------|
| vnet1-subnet1 | x.x.x.0/26 | x.x.x.1 - x.x.x.62 | x.x.x.63 | /26 | 62 |
| AzureBastionSubnet | x.x.x.64/26 | x.x.x.65 - x.x.x.126 | x.x.x.127 | /26 | 62 |
| vnet2-subnet1 | x.x.x.128/26 | x.x.x.129 - x.x.x.190 | x.x.x.191 | /26 | 62 |
| vnet3-subnet1 | x.x.x.192/26 | x.x.x.193 - x.x.x.254 | x.x.x.255 | /26 | 62 |

# Example with Actual IP (131.131.131.0/24)
If your assigned network is `131.131.131.0/24`, your subnets would be:
| Subnet Purpose | Subnet Address | Address Range | VNet |
|----------------|----------------|---------------|------|
| Domain Controller | 131.131.131.0/26 | 131.131.131.1 - 131.131.131.62 | MST300-vnet1 |
| Azure Bastion | 131.131.131.64/26 | 131.131.131.65 - 131.131.131.126 | MST300-vnet1 |
| Webserver | 131.131.131.128/26 | 131.131.131.129 - 131.131.131.190 | MST300-vnet2 |
| Client | 131.131.131.192/26 | 131.131.131.193 - 131.131.131.254 | MST300-vnet3 |

# Completed Subnetting Plan Topology (Example)

**Assigned Network (Example):** `131.131.131.0/24`

| Purpose | Subnet | VNet | Notes |
|---------|--------|------|-------|
| Domain Controller | 131.131.131.0/26 | MST300-vnet1 | DC will get 131.131.131.4 |
| Azure Bastion | 131.131.131.64/26 | MST300-vnet1 | MUST be named AzureBastionSubnet |
| Webserver | 131.131.131.128/26 | MST300-vnet2 | Webserver will get 131.131.131.132 |
| Client | 131.131.131.192/26 | MST300-vnet3 | Client will get 131.131.131.196 |
