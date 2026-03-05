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
# Virtual Machines
| VM Name | OS | Purpose | Network |
|---------|-----|---------|---------|
| dc-vm | Windows Server 2019 | Domain Controller | MST300-vnet1 |
| webserver-vm | Windows Server 2019 | IIS Web Server | MST300-vnet2 |
| client-vm | Windows 10 Pro | Client Workstation | MST300-vnet3 |

### Domain Configuration
- **Domain Name:** `studentID.MST300.com` (replace `studentID` with your actual student ID)
- **Domain Admin:** `studentID.admin`
- **Domain User:** `studentID`

# Subnetting Your Assigned Network

Before creating virtual networks, you need to properly subnet your assigned /24 network address space.

### Understanding the Requirements

You have been assigned a **/24 network** (e.g., `131.131.131.0/24`) and need to create **4 subnets** for:

1. **vnet1-subnet1** - Domain Controller subnet
2. **AzureBastionSubnet** - Azure Bastion (required name)
3. **vnet2-subnet1** - Webserver subnet
4. **vnet3-subnet1** - Client subnet

# Subnetting Strategy
Since you have a /24 network (256 IP addresses) and need 4 subnets, we'll use **/26 subnetting**.
**Why /26?**
- A /26 subnet provides 64 IP addresses (62 usable)
- 256 ÷ 4 = 64 addresses per subnet
- Each subnet gets equal space
- Simple and clean division

# Subnet Calculation Table

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

### Step-by-Step: How to Subnet YOUR Network

Follow these steps with YOUR assigned IP address:

**Step 1: Write down your assigned network**
```
Example: 131.131.131.0/24
Your network: ___.___.___.___ /24
```

**Step 2: Calculate your four /26 subnets**

Replace `x.x.x` with your first three octets:

```
Subnet 1 (vnet1-subnet1):        x.x.x.0/26
Subnet 2 (AzureBastionSubnet):   x.x.x.64/26
Subnet 3 (vnet2-subnet1):        x.x.x.128/26
Subnet 4 (vnet3-subnet1):        x.x.x.192/26
```

**Step 3: Fill in your subnet table**

| My Subnet Name | My Subnet Address | My VNet |
|----------------|-------------------|---------|
| vnet1-subnet1 | ___.___.___.___ /26 | MST300-vnet1 |
| AzureBastionSubnet | ___.___.___.___ /26 | MST300-vnet1 |
| vnet2-subnet1 | ___.___.___.___ /26 | MST300-vnet2 |
| vnet3-subnet1 | ___.___.___.___ /26 | MST300-vnet3 |

### Alternative: Using Private IP Ranges for vnet2 and vnet3

If you prefer (or if peering requires it), you can use private IP address ranges for vnet2 and vnet3:

**Option A: All subnets from your assigned /24 (Recommended)**
```
vnet1: Your assigned network (e.g., 131.131.131.0/26 and 131.131.131.64/26)
vnet2: Your assigned network (e.g., 131.131.131.128/26)
vnet3: Your assigned network (e.g., 131.131.131.192/26)
```

**Option B: Mix of assigned and private ranges**
```
vnet1: Your assigned network (e.g., 131.131.131.0/26 and 131.131.131.64/26)
vnet2: Private range (e.g., 10.0.0.0/24)
vnet3: Private range (e.g., 10.1.0.0/24)
```

**Recommendation:** Use **Option A** (all from your assigned /24) for simplicity and to demonstrate proper subnetting of your assigned address space.

# Example Completed Subnetting Plan

**Assigned Network:** `131.131.131.0/24`

| Purpose | Subnet | VNet | Notes |
|---------|--------|------|-------|
| Domain Controller | 131.131.131.0/26 | MST300-vnet1 | DC will get 131.131.131.4 |
| Azure Bastion | 131.131.131.64/26 | MST300-vnet1 | MUST be named AzureBastionSubnet |
| Webserver | 131.131.131.128/26 | MST300-vnet2 | Webserver will get 131.131.131.132 |
| Client | 131.131.131.192/26 | MST300-vnet3 | Client will get 131.131.131.196 |

---

