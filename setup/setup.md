# MST300 Project 1: Active Directory Domain Services in Azure


# Project Overview
This project implements a complete Active Directory Domain Services environment in Microsoft Azure, featuring:
- A domain controller managing authentication and DNS
- A web server running IIS
- A client workstation
- Three isolated virtual networks connected via peering
- Secure access through Azure Bastion

# Required Knowledge
- Basic understanding of Windows Server
- Familiarity with Active Directory concepts
- Basic networking knowledge (IP addressing, DNS, subnetting)
- Azure Portal navigation

### Required Resources
- Azure for Students account (CloudLab)
- Assigned network address space from instructor (/24 network)
- Student ID (for naming conventions)

### Tools Needed
- Web browser (Chrome, Edge, or Firefox)
- Screen recording software (for project submission)

---

## Architecture

### Network Topology
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

### Virtual Machines
| VM Name | OS | Purpose | Network |
|---------|-----|---------|---------|
| dc-vm | Windows Server 2019 | Domain Controller | MST300-vnet1 |
| webserver-vm | Windows Server 2019 | IIS Web Server | MST300-vnet2 |
| client-vm | Windows 10 Pro | Client Workstation | MST300-vnet3 |

### Domain Configuration
- **Domain Name:** `studentID.MST300.com` (replace `studentID` with your actual student ID)
- **Domain Admin:** `studentID.admin`
- **Domain User:** `studentID`

---

## Subnetting Your Assigned Network

Before creating virtual networks, you need to properly subnet your assigned /24 network address space.

### Understanding the Requirements

You have been assigned a **/24 network** (e.g., `131.131.131.0/24`) and need to create **4 subnets** for:

1. **vnet1-subnet1** - Domain Controller subnet
2. **AzureBastionSubnet** - Azure Bastion (required name)
3. **vnet2-subnet1** - Webserver subnet
4. **vnet3-subnet1** - Client subnet

### Subnetting Strategy

Since you have a /24 network (256 IP addresses) and need 4 subnets, we'll use **/26 subnetting**.

**Why /26?**
- A /26 subnet provides 64 IP addresses (62 usable)
- 256 ÷ 4 = 64 addresses per subnet
- Each subnet gets equal space
- Simple and clean division

### Subnet Calculation Table

| Subnet Name | Network Address | Usable Range | Broadcast | CIDR | Hosts |
|-------------|----------------|--------------|-----------|------|-------|
| vnet1-subnet1 | x.x.x.0/26 | x.x.x.1 - x.x.x.62 | x.x.x.63 | /26 | 62 |
| AzureBastionSubnet | x.x.x.64/26 | x.x.x.65 - x.x.x.126 | x.x.x.127 | /26 | 62 |
| vnet2-subnet1 | x.x.x.128/26 | x.x.x.129 - x.x.x.190 | x.x.x.191 | /26 | 62 |
| vnet3-subnet1 | x.x.x.192/26 | x.x.x.193 - x.x.x.254 | x.x.x.255 | /26 | 62 |

### Example with Actual IP (131.131.131.0/24)

If your assigned network is `131.131.131.0/24`, your subnets would be:

| Subnet Purpose | Subnet Address | Address Range | VNet |
|----------------|----------------|---------------|------|
| Domain Controller | 131.131.131.0/26 | 131.131.131.1 - 131.131.131.62 | MST300-vnet1 |
| Azure Bastion | 131.131.131.64/26 | 131.131.131.65 - 131.131.131.126 | MST300-vnet1 |
| Webserver | 131.131.131.128/26 | 131.131.131.129 - 131.131.131.190 | MST300-vnet2 |
| Client | 131.131.131.192/26 | 131.131.131.193 - 131.131.131.254 | MST300-vnet3 |

### Quick Subnetting Reference

**CIDR Notation Cheat Sheet:**
```
/24 = 255.255.255.0   = 256 addresses (254 usable)
/25 = 255.255.255.128 = 128 addresses (126 usable)
/26 = 255.255.255.192 = 64 addresses  (62 usable)
/27 = 255.255.255.224 = 32 addresses  (30 usable)
/28 = 255.255.255.240 = 16 addresses  (14 usable)
```

**How to Calculate /26 Subnets:**
```
Starting network: x.x.x.0/24

Subnet 1: x.x.x.0/26
  → Start: x.x.x.0
  → End: x.x.x.63
  → Jump to next: Add 64

Subnet 2: x.x.x.64/26
  → Start: x.x.x.64
  → End: x.x.x.127
  → Jump to next: Add 64

Subnet 3: x.x.x.128/26
  → Start: x.x.x.128
  → End: x.x.x.191
  → Jump to next: Add 64

Subnet 4: x.x.x.192/26
  → Start: x.x.x.192
  → End: x.x.x.255
  → Last subnet
```

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

### Verification Checklist

Before proceeding to Part 1, verify:

- [ ] You have your assigned /24 network address
- [ ] You've calculated four /26 subnets
- [ ] Your subnets don't overlap
- [ ] You've written down all subnet addresses
- [ ] You understand which subnet goes in which VNet

### Example Completed Subnetting Plan

**Assigned Network:** `131.131.131.0/24`

| Purpose | Subnet | VNet | Notes |
|---------|--------|------|-------|
| Domain Controller | 131.131.131.0/26 | MST300-vnet1 | DC will get 131.131.131.4 |
| Azure Bastion | 131.131.131.64/26 | MST300-vnet1 | MUST be named AzureBastionSubnet |
| Webserver | 131.131.131.128/26 | MST300-vnet2 | Webserver will get 131.131.131.132 |
| Client | 131.131.131.192/26 | MST300-vnet3 | Client will get 131.131.131.196 |

---

## Part 1: Resource Group Setup

A resource group is a logical container for Azure resources.

### Steps:

1. **Login to Azure Portal**
   - Go to https://portal.azure.com
   - Sign in with your CloudLab credentials (ODL account)

2. **Create Resource Group**
   - Click **"Resource groups"** from the left menu
   - Click **"+ Create"**
   - Fill in the details:
     - **Subscription:** Azure for Students
     - **Resource group name:** `MST300-project1-rg`
     - **Region:** (US) East US (or your assigned region)
   - Click **"Review + create"**
   - Click **"Create"**

### Verification:
✅ Resource group appears in your resource groups list

---

## Part 2: Virtual Networks Configuration

Virtual Networks (VNets) provide isolated network environments for your VMs.

### Important: Use Your Calculated Subnets

Refer to your subnetting plan from the previous section. You should have:
- Subnet 1: x.x.x.0/26 for vnet1-subnet1
- Subnet 2: x.x.x.64/26 for AzureBastionSubnet
- Subnet 3: x.x.x.128/26 for vnet2-subnet1
- Subnet 4: x.x.x.192/26 for vnet3-subnet1

### VNet 1: Domain Controller Network

1. **Navigate to Virtual Networks**
   - Search for "Virtual Networks" in the top search bar
   - Click **"+ Create"**

2. **Basics Tab**
   - **Subscription:** Azure for Students
   - **Resource group:** MST300-project1-rg
   - **Virtual network name:** `MST300-vnet1`
   - **Region:** (US) East US
   - Click **"Next: IP Addresses"**

3. **IP Addresses Tab**
   - **IPv4 address space:** Enter your assigned /24 network
     - Example: `131.131.131.0/24`
     - **Use YOUR assigned network here!**
   
   - Azure will automatically create a default subnet - **DELETE IT**
   - Click the trash icon to delete the default subnet
   
   **Create First Subnet:**
   - Click **"+ Add subnet"**
   - **Subnet name:** `vnet1-subnet1`
   - **Subnet address range:** Your first /26 subnet
     - Example: `131.131.131.0/26`
     - **Use YOUR calculated subnet 1 here!**
   - Click **"Add"**
   
   **Create Second Subnet (for Bastion):**
   - Click **"+ Add subnet"**
   - **Subnet name:** `AzureBastionSubnet` (MUST be exactly this name - no spaces, capital letters matter!)
   - **Subnet address range:** Your second /26 subnet
     - Example: `131.131.131.64/26`
     - **Use YOUR calculated subnet 2 here!**
   - Click **"Add"**

4. **Review + Create**
   - Verify both subnets are listed
   - Verify address spaces don't overlap
   - Click **"Review + create"**
   - Click **"Create"**

### VNet 2: Webserver Network

**Using your assigned address space:**

1. **Create VNet**
   - Click **"+ Create"** virtual network
   - **Name:** `MST300-vnet2`
   - **Region:** (US) East US

2. **IP Addresses Tab**
   - **IPv4 address space:** Your third /26 subnet AS /26
     - Example: `131.131.131.128/26`
     - **Use YOUR calculated subnet 3 here!**
   
   - Delete default subnet if present
   - Click **"+ Add subnet"**
   - **Subnet name:** `vnet2-subnet1`
   - **Subnet address range:** Same as VNet address space
     - Example: `131.131.131.128/26`
   - Click **"Add"**

3. **Review + Create**

### VNet 3: Client Network

**Using your assigned address space:**

1. **Create VNet**
   - **Name:** `MST300-vnet3`
   - **Region:** (US) East US

2. **IP Addresses Tab**
   - **IPv4 address space:** Your fourth /26 subnet AS /26
     - Example: `131.131.131.192/26`
     - **Use YOUR calculated subnet 4 here!**
   
   - Delete default subnet if present
   - Click **"+ Add subnet"**
   - **Subnet name:** `vnet3-subnet1`
   - **Subnet address range:** Same as VNet address space
     - Example: `131.131.131.192/26`
   - Click **"Add"**

3. **Review + Create**

### Your Final Network Layout Should Look Like:

**Example using 131.131.131.0/24:**

```
MST300-vnet1 (131.131.131.0/24)
├── vnet1-subnet1 (131.131.131.0/26)       ← Domain Controller here
└── AzureBastionSubnet (131.131.131.64/26) ← Bastion here

MST300-vnet2 (131.131.131.128/26)
└── vnet2-subnet1 (131.131.131.128/26)     ← Webserver here

MST300-vnet3 (131.131.131.192/26)
└── vnet3-subnet1 (131.131.131.192/26)     ← Client here
```

### Verification:
✅ Three virtual networks created  
✅ MST300-vnet1 has 2 subnets (vnet1-subnet1 and AzureBastionSubnet)  
✅ MST300-vnet2 has 1 subnet (vnet2-subnet1)  
✅ MST300-vnet3 has 1 subnet (vnet3-subnet1)  
✅ All subnets use your assigned address space  
✅ No overlapping IP addresses  
✅ All subnets are /26 (64 addresses each)

---

## Part 3: Azure Bastion Setup

Azure Bastion provides secure RDP/SSH access without exposing VMs to the public internet.

### Steps:

1. **Create Bastion Host**
   - Search for "Bastions" in Azure Portal
   - Click **"+ Create"**

2. **Configure Bastion**
   - **Subscription:** Azure for Students
   - **Resource group:** MST300-project1-rg
   - **Name:** `MST300-BastionHost`
   - **Region:** (US) East US (must match VNet region)
   - **Tier:** Basic
   - **Virtual network:** MST300-vnet1
   - **Subnet:** AzureBastionSubnet (should auto-select)

3. **Public IP Address**
   - **Public IP address:** Create new
   - **Public IP address name:** `MST300-BastionIP`
   - **Public IP address SKU:** Standard
   - **Assignment:** Static

4. **Review + Create**
   - Click **"Review + create"**
   - Click **"Create"**
   - ⏱️ **Wait 10-15 minutes** for deployment

### Verification:
✅ Bastion host shows "Running" status  
✅ Public IP is assigned

---

## Part 4: Domain Controller VM

The Domain Controller manages authentication, user accounts, and DNS.

### Steps:

1. **Create Virtual Machine**
   - Search for "Virtual machines"
   - Click **"+ Create"** → **"Azure virtual machine"**

2. **Basics Tab**
   - **Subscription:** Azure for Students
   - **Resource group:** MST300-project1-rg
   - **Virtual machine name:** `dc-vm`
   - **Region:** (US) East US
   - **Availability options:** No infrastructure redundancy required
   - **Security type:** Standard
   - **Image:** Windows Server 2019 Datacenter - Gen2
   - **Size:** Standard_B2s (IMPORTANT: Don't use B1s - it may crash during DC promotion)
   - **Username:** `azureuser`
   - **Password:** [Create a strong password - save it!]
   - **Public inbound ports:** None
   - **Licensing:** Check "I confirm I have an eligible Windows Server license"

3. **Disks Tab**
   - **OS disk type:** Standard HDD
   - Click **"Next: Networking"**

4. **Networking Tab**
   - **Virtual network:** MST300-vnet1
   - **Subnet:** vnet1-subnet1
   - **Public IP:** None
   - **NIC network security group:** Basic
   - **Public inbound ports:** None
   - Click **"Review + create"**

5. **Create**
   - Click **"Create"**
   - ⏱️ Wait 3-5 minutes for deployment

### Verification:
✅ VM shows "Running" status  
✅ No public IP assigned  
✅ Connected to MST300-vnet1

---

## Part 5: Active Directory Domain Services

Configure the Domain Controller to provide Active Directory services.

### Part 5.1: Install AD DS Role

1. **Connect to dc-vm**
   - Go to dc-vm in Azure Portal
   - Click **"Connect"** → **"Bastion"**
   - **Username:** `azureuser`
   - **Password:** [your password]
   - Click **"Connect"**

2. **Open Server Manager**
   - Server Manager should open automatically
   - If not, click Start → Server Manager

3. **Add Roles and Features**
   - Click **"Add roles and features"**
   - Click **"Next"** (Before you begin)
   - **Installation Type:** Role-based or feature-based installation
   - Click **"Next"**
   - **Server Selection:** Select dc-vm
   - Click **"Next"**

4. **Select Server Roles**
   - Check ☑️ **"Active Directory Domain Services"**
   - Click **"Add Features"** (when prompted)
   - Click **"Next"**

5. **Features**
   - Keep defaults
   - Click **"Next"**

6. **AD DS**
   - Click **"Next"**

7. **Confirmation**
   - Click **"Install"**
   - ⏱️ Wait 5-10 minutes
   - Click **"Close"**

### Part 5.2: Promote to Domain Controller

1. **Start Promotion**
   - In Server Manager, click the **flag icon** with yellow warning
   - Click **"Promote this server to a domain controller"**

2. **Deployment Configuration**
   - Select ⚫ **"Add a new forest"**
   - **Root domain name:** `studentID.MST300.com` (replace studentID with yours, e.g., `asabra1.MST300.com`)
   - Click **"Next"**

3. **Domain Controller Options**
   - **Forest functional level:** Windows Server 2016
   - **Domain functional level:** Windows Server 2016
   - Check ☑️ **DNS server**
   - Check ☑️ **Global Catalog (GC)**
   - **DSRM Password:** [Create and save a password]
   - Click **"Next"**

4. **DNS Options**
   - Ignore the warning about DNS delegation
   - Click **"Next"**

5. **Additional Options**
   - **NetBIOS domain name:** Should auto-populate (e.g., ASABRA1)
   - Click **"Next"**

6. **Paths**
   - Keep default paths
   - Click **"Next"**

7. **Review Options**
   - Review your settings
   - Click **"Next"**

8. **Prerequisites Check**
   - Ignore warnings (yellow triangles are okay)
   - Errors (red X) must be resolved
   - Click **"Install"**
   - ⏱️ **Wait 10-15 minutes**
   - **VM will restart automatically**

### Part 5.3: Create Domain Users

1. **Reconnect to dc-vm**
   - Wait 2-3 minutes after restart
   - Connect via Bastion
   - **Username:** `DOMAINNAME\azureuser` (e.g., `ASABRA1\azureuser`)
   - **Password:** [your password]

2. **Open Active Directory Users and Computers**
   - Server Manager → Tools → Active Directory Users and Computers

3. **Create Domain Admin User**
   - Expand your domain (e.g., asabra1.MST300.com)
   - Right-click **"Users"** → New → User
   - **First name:** studentID
   - **Last name:** admin
   - **User logon name:** `studentID.admin` (e.g., `asabra1.admin`)
   - Click **"Next"**
   - **Password:** [Create a strong password - save it!]
   - Uncheck ☐ "User must change password at next logon"
   - Check ☑️ "Password never expires"
   - Click **"Next"** → **"Finish"**

4. **Add to Domain Admins Group**
   - Find the user **studentID.admin**
   - Right-click → **Properties**
   - Go to **"Member Of"** tab
   - Click **"Add"**
   - Type: `Domain Admins`
   - Click **"Check Names"** → **"OK"** → **"OK"**

5. **Create Domain User**
   - Right-click **"Users"** → New → User
   - **First name:** studentID (e.g., asabra1)
   - **User logon name:** `studentID` (e.g., `asabra1`)
   - Click **"Next"**
   - **Password:** [Create a password - save it!]
   - Uncheck ☐ "User must change password at next logon"
   - Check ☑️ "Password never expires"
   - Click **"Next"** → **"Finish"**

### Verification:
✅ Domain controller is running  
✅ DNS role installed  
✅ Two users created: studentID.admin and studentID  
✅ studentID.admin is in Domain Admins group  
✅ studentID is in Domain Users group

---

## Part 6: Webserver VM

Create a Windows Server VM that will host IIS web server.

### Steps:

1. **Create Virtual Machine**
   - Go to Virtual machines → **"+ Create"**

2. **Basics Tab**
   - **Resource group:** MST300-project1-rg
   - **Virtual machine name:** `webserver-vm`
   - **Region:** (US) East US
   - **Image:** Windows Server 2019 Datacenter - Gen2
   - **Size:** Standard_B1s (or larger if needed)
   - **Username:** `azureuser`
   - **Password:** [same password as dc-vm for simplicity]
   - **Public inbound ports:** None

3. **Disks Tab**
   - **OS disk type:** Standard HDD

4. **Networking Tab**
   - **Virtual network:** MST300-vnet2
   - **Subnet:** vnet2-subnet1
   - **Public IP:** None

5. **Review + Create**
   - Click **"Review + create"**
   - Click **"Create"**

### Verification:
✅ webserver-vm is running  
✅ Connected to MST300-vnet2  
✅ No public IP

---

## Part 7: Client VM

Create a Windows 10 client workstation.

### Steps:

1. **Create Virtual Machine**
   - Go to Virtual machines → **"+ Create"**

2. **Basics Tab**
   - **Resource group:** MST300-project1-rg
   - **Virtual machine name:** `client-vm`
   - **Region:** (US) East US
   - **Image:** Windows 10 Pro, Version 22H2 - Gen2 (or latest available)
   - **Size:** Standard_B1s (or larger if needed)
   - **Username:** `azureuser`
   - **Password:** [same password]
   - **Licensing:** Check the Windows Client license box
   - **Public inbound ports:** None

3. **Disks Tab**
   - **OS disk type:** Standard HDD

4. **Networking Tab**
   - **Virtual network:** MST300-vnet3
   - **Subnet:** vnet3-subnet1
   - **Public IP:** None

5. **Review + Create**
   - Click **"Review + create"**
   - Click **"Create"**

### Verification:
✅ client-vm is running  
✅ Connected to MST300-vnet3  
✅ No public IP

---

## Part 8: Virtual Network Peering

Connect the three isolated networks so VMs can communicate.

### Peering 1: vnet1 ↔ vnet2

1. **Navigate to MST300-vnet1**
   - Go to Virtual Networks → MST300-vnet1
   - Click **"Peerings"** in left menu
   - Click **"+ Add"**

2. **Configure Peering**
   - **This virtual network - Peering link name:** `vnet1-to-vnet2`
   - **Remote virtual network - Peering link name:** `vnet2-to-vnet1`
   - **Virtual network:** MST300-vnet2
   - Leave all other settings as default
   - Click **"Add"**

### Peering 2: vnet1 ↔ vnet3

1. **Add Another Peering**
   - Still in MST300-vnet1 → Peerings
   - Click **"+ Add"**

2. **Configure Peering**
   - **This virtual network - Peering link name:** `vnet1-to-vnet3`
   - **Remote virtual network - Peering link name:** `vnet3-to-vnet1`
   - **Virtual network:** MST300-vnet3
   - Click **"Add"**

### Peering 3: vnet2 ↔ vnet3

1. **Navigate to MST300-vnet2**
   - Go to Virtual Networks → MST300-vnet2
   - Click **"Peerings"**
   - Click **"+ Add"**

2. **Configure Peering**
   - **This virtual network - Peering link name:** `vnet2-to-vnet3`
   - **Remote virtual network - Peering link name:** `vnet3-to-vnet2`
   - **Virtual network:** MST300-vnet3
   - Click **"Add"**

### Verification:
✅ All three VNets show peering status as "Connected"  
✅ Six peering connections total (bidirectional)

---

## Part 9: Joining VMs to Domain

### Part 9.1: Configure DNS Settings

Before joining VMs to the domain, they must use the Domain Controller as their DNS server.

#### Get Domain Controller IP Address

1. Go to Virtual Machines → **dc-vm**
2. Click **"Networking"** in left menu
3. Note the **Private IP address** (e.g., 131.131.131.4)

#### Configure DNS for vnet2 (Webserver)

1. **Navigate to MST300-vnet2**
   - Go to Virtual Networks → MST300-vnet2
   - Click **"DNS servers"** in left menu

2. **Set Custom DNS**
   - Select ⚫ **"Custom"**
   - **DNS server:** [Your DC IP, e.g., 131.131.131.4]
   - Click **"Save"**

3. **Restart webserver-vm**
   - Go to webserver-vm
   - Click **"Restart"**
   - Wait for restart to complete

#### Configure DNS for vnet3 (Client)

1. **Navigate to MST300-vnet3**
   - Go to Virtual Networks → MST300-vnet3
   - Click **"DNS servers"**

2. **Set Custom DNS**
   - Select ⚫ **"Custom"**
   - **DNS server:** [Your DC IP, e.g., 131.131.131.4]
   - Click **"Save"**

3. **Restart client-vm**
   - Go to client-vm
   - Click **"Restart"**

### Part 9.2: Join webserver-vm to Domain

1. **Connect to webserver-vm**
   - Use Bastion to connect
   - **Username:** `azureuser`
   - **Password:** [your password]

2. **Verify DNS**
   - Open PowerShell as Administrator
   - Run: `ipconfig /all`
   - Verify DNS server shows DC IP (e.g., 131.131.131.4)
   - Run: `nslookup studentID.MST300.com` (e.g., `nslookup asabra1.MST300.com`)
   - Should resolve to DC IP

3. **Join Domain**
   - Right-click **"This PC"** → **Properties**
   - Click **"Advanced system settings"**
   - Go to **"Computer Name"** tab
   - Click **"Change"**
   - Select ⚫ **"Domain"**
   - Type: `studentID.MST300.com` (e.g., `asabra1.MST300.com`)
   - Click **"OK"**

4. **Enter Domain Credentials**
   - **Username:** `studentID.admin` (e.g., `asabra1.admin`)
   - **Password:** [your domain admin password]
   - Click **"OK"**

5. **Welcome Message**
   - You should see "Welcome to the [domain] domain"
   - Click **"OK"**
   - Click **"OK"** to restart
   - Click **"Restart Now"**

### Part 9.3: Join client-vm to Domain

Repeat the exact same process for client-vm:
1. Connect via Bastion
2. Verify DNS
3. Join to domain using domain admin credentials
4. Restart

### Verification:
✅ Both VMs joined to domain successfully  
✅ Both VMs restarted  
✅ DNS resolving correctly on both VMs

---

## Part 10: IIS Configuration

Install and configure Internet Information Services on the webserver.

### Part 10.1: Install IIS Role

1. **Connect to webserver-vm**
   - Use Bastion
   - **Username:** `studentID.admin@studentID.MST300.com` (e.g., `asabra1.admin@asabra1.MST300.com`)
   - **Password:** [your domain admin password]

2. **Open Server Manager**
   - Should open automatically

3. **Add Roles and Features**
   - Click **"Add roles and features"**
   - Click **"Next"** (Before you begin)
   - **Installation Type:** Role-based
   - Click **"Next"**
   - Select **webserver-vm**
   - Click **"Next"**

4. **Select Server Role**
   - Check ☑️ **"Web Server (IIS)"**
   - Click **"Add Features"**
   - Click **"Next"**

5. **Features**
   - Keep defaults
   - Click **"Next"**

6. **Web Server Role (IIS)**
   - Click **"Next"**

7. **Role Services**
   - Keep defaults (includes Management Tools)
   - Click **"Next"**

8. **Confirmation**
   - Click **"Install"**
   - ⏱️ Wait 3-5 minutes
   - Click **"Close"**

### Part 10.2: Create DNS Record for Webserver

1. **Connect to dc-vm**
   - Use Bastion with domain admin credentials

2. **Open DNS Manager**
   - Server Manager → Tools → DNS

3. **Create A Record**
   - Expand your server name
   - Expand **"Forward Lookup Zones"**
   - Click on your domain (e.g., asabra1.MST300.com)
   - Right-click → **New Host (A or AAAA)...**
   - **Name:** `webserver-vm`
   - **IP address:** [webserver-vm private IP - get from Azure Portal]
   - Check ☑️ "Create associated pointer (PTR) record" (optional)
   - Click **"Add Host"**
   - Click **"OK"**
   - Click **"Done"**

### Part 10.3: Modify IIS Default Page

1. **On webserver-vm, open File Explorer**
   - Navigate to: `C:\inetpub\wwwroot`

2. **Edit iisstart.htm**
   - Right-click `iisstart.htm` → **Open with** → **Notepad**

3. **Find and Replace Title**
   - Press `Ctrl + F` to find
   - Search for: `<title>IIS Windows Server</title>`
   - Replace with: `<title>studentID's Windows Server</title>` (e.g., `<title>asabra1's Windows Server</title>`)

4. **Save File**
   - File → Save
   - Close Notepad

### Verification:
✅ IIS is installed and running  
✅ DNS record created for webserver-vm  
✅ Default page title modified

---

## Part 11: DNS Configuration

Ensure client-vm can resolve the webserver FQDN.

### Part 11.1: Test DNS Resolution

1. **Connect to client-vm**
   - Use Bastion
   - **Username:** `studentID.admin@studentID.MST300.com`
   - **Password:** [domain admin password]

2. **Open PowerShell**
   - Press `Win + X` → Windows PowerShell

3. **Test DNS**
   ```powershell
   nslookup webserver-vm.studentID.MST300.com
   ```
   - Should return the webserver's IP address
   - If it fails, flush DNS:
   ```powershell
   ipconfig /flushdns
   ipconfig /registerdns
   ```

### Verification:
✅ DNS resolves webserver FQDN to correct IP  
✅ Client can ping webserver by name

---

## Part 12: User Permissions

Configure user permissions so domain users can log in via Bastion.

### Part 12.1: Configure Group Policy

1. **Connect to dc-vm**
   - Use Bastion with domain admin account

2. **Open Group Policy Management**
   - Server Manager → Tools → Group Policy Management

3. **Edit Default Domain Policy**
   - Expand Forest → Domains → [your domain]
   - Right-click **"Default Domain Policy"** → **Edit**

4. **Navigate to User Rights Assignment**
   - Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → User Rights Assignment

5. **Configure "Allow log on locally"**
   - Double-click **"Allow log on locally"**
   - Check ☑️ "Define these policy settings"
   - Click **"Add User or Group"**
   - Type: `Domain Users`
   - Click **"Check Names"** → **"OK"**
   - Click **"OK"**

6. **Configure "Allow log on through Remote Desktop Services"**
   - Double-click **"Allow log on through Remote Desktop Services"**
   - Check ☑️ "Define these policy settings"
   - Click **"Add User or Group"**
   - Add: `Domain Users` and `Remote Desktop Users`
   - Click **"OK"**

7. **Close Group Policy Editor**

### Part 12.2: Add Users to Remote Desktop Users Group

1. **Open Active Directory Users and Computers**
   - Server Manager → Tools → Active Directory Users and Computers

2. **Navigate to Builtin Container**
   - Expand your domain
   - Click **"Builtin"**

3. **Edit Remote Desktop Users Group**
   - Double-click **"Remote Desktop Users"**
   - Go to **"Members"** tab
   - Click **"Add"**
   - Type: `Domain Users`
   - Click **"Check Names"** → **"OK"** → **"OK"**

### Part 12.3: Apply Group Policy

1. **On dc-vm, open PowerShell**
   ```powershell
   gpupdate /force
   ```

2. **On webserver-vm, open PowerShell**
   ```powershell
   gpupdate /force
   ```

3. **On client-vm, open PowerShell**
   ```powershell
   gpupdate /force
   ```

4. **Restart all VMs**
   - Restart dc-vm, webserver-vm, and client-vm

### Verification:
✅ Domain Users can log on locally  
✅ Domain Users can log on via Remote Desktop  
✅ Group policy applied successfully

---

## Part 13: Testing and Verification

Verify all components are working correctly.

### Test 1: Domain User Login

1. **Connect to client-vm via Bastion**
   - **Username:** `studentID@studentID.MST300.com` (e.g., `asabra1@asabra1.mst300.com`)
   - **Password:** [domain user password]
   - Should log in successfully

### Test 2: Website Access via FQDN

1. **On client-vm, open browser**
   - Open Microsoft Edge or Internet Explorer

2. **Navigate to webserver**
   - Type: `http://webserver-vm.studentID.MST300.com`
   - Press Enter

3. **Verify**
   - Page should load showing IIS welcome page
   - Browser tab should show your custom title: "studentID's Windows Server"

### Test 3: Network Connectivity

1. **On client-vm, open PowerShell**
   ```powershell
   # Test DNS
   nslookup webserver-vm.asabra1.MST300.com
   
   # Test connectivity
   ping webserver-vm.asabra1.MST300.com
   
   # Test HTTP
   Test-NetConnection -ComputerName webserver-vm.asabra1.MST300.com -Port 80
   ```

### Test 4: Domain Membership

1. **On dc-vm, verify computers in domain**
   - Active Directory Users and Computers → Computers
   - Should see: CLIENT-VM and WEBSERVER-VM

2. **On dc-vm, verify users**
   - Active Directory Users and Computers → Users
   - Should see: studentID and studentID.admin

### Verification Checklist:
✅ Can log into client-vm with domain user (not admin)  
✅ Can access website using FQDN  
✅ Website shows custom title  
✅ DNS resolution works  
✅ Both VMs appear in AD Computers  
✅ Both users appear in AD Users  
✅ Virtual network peering is active  
✅ No public IPs on VMs (only Bastion)

---

## Troubleshooting Guide

### Issue 1: Cannot Connect via Bastion

**Symptoms:**
- "Connection Error" when trying to connect
- "Target machine is unreachable"

**Solutions:**
1. Verify VM is running (not stopped or deallocated)
2. Check Bastion is deployed and running
3. Verify VM is in MST300-vnet1 (or peered network)
4. Wait 2-3 minutes after VM restart
5. Try different browser or incognito mode

---

### Issue 2: Cannot Join Domain

**Symptoms:**
- "Domain not found" error
- "No logon servers available"

**Solutions:**
1. **Verify DNS Settings:**
   ```powershell
   ipconfig /all
   # DNS Server should show DC IP
   ```

2. **Test DNS Resolution:**
   ```powershell
   nslookup studentID.MST300.com
   # Should return DC IP
   ```

3. **Verify Network Peering:**
   - Check peering status is "Connected"
   - Verify peering exists between networks

4. **Restart VM after DNS change:**
   - Always restart VM after changing VNet DNS settings

5. **Check DC is running:**
   - Verify dc-vm is running
   - Verify AD DS service is running on DC

---

### Issue 3: Cannot Login with Domain User

**Symptoms:**
- "Username or password is incorrect"
- Works with admin but not regular user

**Solutions:**
1. **Check Group Membership:**
   - Verify user is in "Domain Users" group
   - Verify user is in "Remote Desktop Users" group (or add Domain Users to this group)

2. **Check Group Policy:**
   - Verify "Allow log on locally" includes Domain Users
   - Verify "Allow log on through Remote Desktop Services" includes Domain Users

3. **Force Group Policy Update:**
   ```powershell
   gpupdate /force
   ```

4. **Reset User Password:**
   - On DC, reset user password in AD Users and Computers
   - Uncheck "User must change password at next logon"
   - Check "Password never expires"

5. **Check Account Status:**
   - Verify account is not disabled
   - Verify account is not locked
   - Check "Log On To" allows all computers

---

### Issue 4: Website Not Accessible

**Symptoms:**
- Cannot access `http://webserver-vm.studentID.MST300.com`
- DNS resolution fails
- Connection timeout

**Solutions:**
1. **Test DNS Resolution:**
   ```powershell
   nslookup webserver-vm.studentID.MST300.com
   # Should return webserver IP
   ```

2. **Flush DNS Cache:**
   ```powershell
   ipconfig /flushdns
   ipconfig /registerdns
   ```

3. **Verify IIS is Running:**
   - On webserver-vm, open Server Manager
   - Check IIS role is installed
   - Open browser locally and test: `http://localhost`

4. **Check Firewall:**
   - On webserver-vm:
   ```powershell
   # Check if HTTP is allowed
   Get-NetFirewallRule -DisplayName "*World Wide Web*"
   ```

5. **Verify DNS Record:**
   - On dc-vm, open DNS Manager
   - Verify A record exists for webserver-vm
   - Verify IP address is correct

6. **Test Connectivity:**
   ```powershell
   # From client-vm
   Test-NetConnection -ComputerName webserver-vm.studentID.MST300.com -Port 80
   ```

---

### Issue 5: VM Crashes or Restarts

**Symptoms:**
- VM becomes unresponsive
- DC crashes during promotion
- Out of memory errors

**Solutions:**
1. **Increase VM Size:**
   - Stop the VM
   - Go to Size in Azure Portal
   - Select larger size (e.g., Standard_B2s instead of B1s)
   - Start VM

2. **For Domain Controller:**
   - Use at least Standard_B2s (2 vCPU, 4 GB RAM)
   - B1s may not have enough resources for AD DS

---

### Issue 6: Peering Not Working

**Symptoms:**
- Cannot ping between networks
- VMs cannot communicate
- DNS resolution fails between networks

**Solutions:**
1. **Verify Peering Status:**
   - All peerings should show "Connected"
   - Check both directions of each peering

2. **Check Address Space:**
   - Ensure no overlapping IP ranges between VNets

3. **Recreate Peering:**
   - Delete existing peering
   - Wait 2 minutes
   - Create new peering

4. **Verify Network Security Groups:**
   - Check NSG rules aren't blocking traffic
   - Default rules should allow VNet-to-VNet traffic

---

### Issue 7: Azure Bastion Username Format

**Symptoms:**
- "Username in domain\username format is not supported"
- Cannot use DOMAIN\username format

**Solutions:**
Azure Bastion requires specific username formats:

✅ **Correct Formats:**
- `username@domain.com` (UPN format) - **RECOMMENDED**
- `username` (if computer is properly domain-joined)
- `.\username` (for local accounts)

❌ **Incorrect Formats:**
- `DOMAIN\username` (backslash format - NOT supported)
- `domain.com\username`

**Examples:**
- Domain admin: `asabra1.admin@asabra1.MST300.com`
- Domain user: `asabra1@asabra1.MST300.com`
- Local admin: `.\azureuser` or just `azureuser`

---

### Issue 8: Password Issues

**Symptoms:**
- Forgot password
- Account locked
- Password expired

**Solutions:**
1. **Reset Local Admin Password:**
   - In Azure Portal, go to VM
   - Click "Reset password"
   - Enter new password
   - Click "Update"

2. **Reset Domain User Password:**
   - Connect to DC with domain admin
   - Active Directory Users and Computers
   - Find user → Reset Password
   - Uncheck "User must change password"
   - Check "Password never expires"

3. **Unlock Account:**
   - In AD Users and Computers
   - User Properties → Account tab
   - Check "Unlock account"

---

## Project Submission

### Video Recording Requirements

Create a video demonstration (under 7 minutes) showing:

#### 1. Introduction (30 seconds)
- Show your face on camera
- State your name
- Show CloudLab account
- Show date and time
- Confirm it's your own work

#### 2. Azure Bastion Login (1 minute)
- Connect to client-vm using domain user (NOT admin)
- Show username: `studentID@studentID.MST300.com`
- Successfully log in

#### 3. Website Access (1.5 minutes)
- From client-vm, open browser
- Navigate to: `http://webserver-vm.studentID.MST300.com`
- Show custom page title in browser tab

#### 4. Virtual Networks (2 minutes)
- In Azure Portal, show three VNets
- Show subnets in each VNet
- Show peering configuration
- Demonstrate all peerings are "Connected"

#### 5. Domain Controller (2 minutes)
- Connect to dc-vm with admin account
- Open Active Directory Users and Computers
- Show domain name
- Show two users: studentID and studentID.admin
- Show computers: CLIENT-VM and WEBSERVER-VM

#### 6. Conclusion (30 seconds)
- Summarize what was demonstrated
- Thank the viewer

### Simple Video Script

**Refer to the main guide for the detailed step-by-step script.**

### Recording Software Options

**Windows Game Bar (Built-in):**
```
Press Win + G
Click record button
Easy and quick
```

**OBS Studio (Free, Professional):**
```
Download from obsproject.com
More features
Better quality
```

**Microsoft Stream:**
```
Record directly on stream.microsoft.com
No separate upload needed
```

### Submission Steps

1. **Record video**
   - Follow script above
   - Keep under 7 minutes
   - Ensure camera and mic are working

2. **Review video**
   - Watch entire video
   - Verify audio is clear
   - Verify screen is visible

3. **Upload to Microsoft Stream**
   - Go to Microsoft Stream
   - Click "Upload"
   - Select your video file
   - Title: "MST300 Project 1 - [Your Name]"
   - Set appropriate permissions for instructor

4. **Submit to Blackboard**
   - Copy video link from Stream
   - Submit link in Blackboard assignment

---

## Grading Rubric

### Component Breakdown:

| Component | Points | Requirements |
|-----------|--------|--------------|
| Azure Bastion | 25% | Login with domain user (not admin) |
| Web Server | 25% | Access via FQDN, custom title shown |
| Virtual Network Peering | 25% | Three VNets, proper peering, demonstrate configuration |
| Domain Controller | 25% | Show domain, users, domain-joined computers |

### Penalties:

| Issue | Penalty |
|-------|---------|
| Not using CloudLab account | Project not graded |
| Not using assigned IP address | -20% |
| Incorrect resource naming | -20% |
| No audio/video in presentation | Arrange with instructor or not graded |
| Late submission (per day, max 3 days) | -10% |
| After 3 days | Grade of 0 |

---

## Final Checklist

Before recording your video, verify:

### Azure Resources:
- [ ] Resource group: MST300-project1-rg created
- [ ] Three VNets created with correct names
- [ ] VNet1 has 2 subnets (including AzureBastionSubnet)
- [ ] VNet2 and VNet3 each have 1 subnet
- [ ] All subnets properly calculated from your /24 assignment
- [ ] Azure Bastion deployed and running
- [ ] All three VMs created and running
- [ ] No public IPs on VMs (only Bastion has public IP)

### Domain Configuration:
- [ ] Domain name: studentID.MST300.com
- [ ] Two users created: studentID and studentID.admin
- [ ] studentID.admin in Domain Admins group
- [ ] studentID in Domain Users group
- [ ] Both webserver-vm and client-vm joined to domain
- [ ] Both VMs appear in AD Computers container

### Network Configuration:
- [ ] All three VNets peered (6 peering connections total)
- [ ] All peering status shows "Connected"
- [ ] DNS configured on vnet2 and vnet3 pointing to DC
- [ ] DNS resolution working from client to webserver

### IIS Configuration:
- [ ] IIS installed on webserver-vm
- [ ] DNS A record created for webserver-vm
- [ ] Default page title modified to "studentID's Windows Server"
- [ ] Website accessible via FQDN from client-vm

### Permissions:
- [ ] Domain Users added to "Allow log on locally" policy
- [ ] Domain Users added to "Allow log on through Remote Desktop Services"
- [ ] Domain Users added to Remote Desktop Users group
- [ ] Can log in to all VMs with domain user (not just admin)

### Testing:
- [ ] Can log into client-vm with studentID@studentID.MST300.com
- [ ] Can access http://webserver-vm.studentID.MST300.com from client
- [ ] Website shows custom title
- [ ] All VMs communicating through peered networks

---

## Appendix A: Subnetting Quick Reference

### Calculator Method

If you need help calculating subnets, use an online subnet calculator:
- https://www.subnet-calculator.com/
- https://www.calculator.net/ip-subnet-calculator.html

**How to use:**
1. Enter your assigned IP (e.g., 131.131.131.0)
2. Enter subnet mask: /24
3. Enter number of subnets needed: 4
4. Calculator will show you the four /26 subnets

### Binary Explanation (Optional - For Understanding)

**Why does /26 give us 64 addresses?**

```
/24 = 11111111.11111111.11111111.00000000
                                 ^^^^^^^^
                                 8 bits = 2^8 = 256 addresses

/26 = 11111111.11111111.11111111.11000000
                                 ^^
                                 2 bits borrowed for subnets
                                   ^^^^^^
                                   6 bits for hosts = 2^6 = 64 addresses
```

**The four /26 subnets in binary:**

```
Subnet 1: xxx.xxx.xxx.00000000 = xxx.xxx.xxx.0   (/26)
Subnet 2: xxx.xxx.xxx.01000000 = xxx.xxx.xxx.64  (/26)
Subnet 3: xxx.xxx.xxx.10000000 = xxx.xxx.xxx.128 (/26)
Subnet 4: xxx.xxx.xxx.11000000 = xxx.xxx.xxx.192 (/26)
```

### Subnetting Formula

```
Number of subnets = 2^n (where n = borrowed bits)
Number of hosts per subnet = 2^h - 2 (where h = host bits)

For /26:
- Network bits: 26
- Host bits: 32 - 26 = 6
- Borrowed bits (from /24): 26 - 24 = 2
- Number of subnets: 2^2 = 4 subnets
- Hosts per subnet: 2^6 - 2 = 64 - 2 = 62 usable hosts
```

### Subnet Mask Conversion Table

| CIDR | Decimal Mask | Wildcard Mask | # Addresses | Usable Hosts |
|------|--------------|---------------|-------------|--------------|
| /24 | 255.255.255.0 | 0.0.0.255 | 256 | 254 |
| /25 | 255.255.255.128 | 0.0.0.127 | 128 | 126 |
| /26 | 255.255.255.192 | 0.0.0.63 | 64 | 62 |
| /27 | 255.255.255.224 | 0.0.0.31 | 32 | 30 |
| /28 | 255.255.255.240 | 0.0.0.15 | 16 | 14 |

### Practice Exercise

**Given network: 192.168.10.0/24**

Calculate four /26 subnets:

**Answer:**
```
Subnet 1: 192.168.10.0/26   (192.168.10.0 - 192.168.10.63)
Subnet 2: 192.168.10.64/26  (192.168.10.64 - 192.168.10.127)
Subnet 3: 192.168.10.128/26 (192.168.10.128 - 192.168.10.191)
Subnet 4: 192.168.10.192/26 (192.168.10.192 - 192.168.10.255)
```

---

## Additional Resources

### Microsoft Documentation:
- [Azure Virtual Networks](https://docs.microsoft.com/azure/virtual-network/)
- [Azure Bastion](https://docs.microsoft.com/azure/bastion/)
- [Active Directory Domain Services](https://docs.microsoft.com/windows-server/identity/ad-ds/)
- [IIS Configuration](https://docs.microsoft.com/iis/)

### Useful PowerShell Commands:

```powershell
# Check DNS settings
ipconfig /all

# Test DNS resolution
nslookup domain.com

# Flush DNS cache
ipconfig /flushdns
ipconfig /registerdns

# Force Group Policy update
gpupdate /force

# Test network connectivity
Test-NetConnection -ComputerName hostname -Port 80
ping hostname

# Check domain membership
(Get-WmiObject Win32_ComputerSystem).Domain

# Restart network adapter
Restart-NetAdapter -Name "Ethernet"

# Get IP configuration
Get-NetIPConfiguration

# Check firewall rules
Get-NetFirewallRule | Where-Object {$_.Enabled -eq "True"}
```

---

## Tips for Success

### Planning:
1. Read the entire guide before starting
2. Have your assigned IP address ready
3. Save all passwords in a secure location
4. Budget 2-3 hours for completion
5. Complete subnetting calculations BEFORE starting

### During Setup:
1. Complete one section at a time
2. Verify each component before moving on
3. Don't skip verification steps
4. Take screenshots of important configurations

### Troubleshooting:
1. Read error messages carefully
2. Check Azure Portal for resource status
3. Use PowerShell commands to test connectivity
4. Restart VMs after configuration changes
5. Wait 2-3 minutes after restarts before testing

### Video Recording:
1. Practice once before final recording
2. Speak clearly and at moderate pace
3. Show your face at the beginning
4. Keep under 7 minutes
5. Follow the script provided

---

## Project Timeline

**Recommended schedule for 2-3 hour completion:**

| Time | Task |
|------|------|
| 0:00-0:10 | Subnetting calculation and planning |
| 0:10-0:25 | Resource Group and VNets |
| 0:25-0:40 | Azure Bastion (deployment takes 10-15 min) |
| 0:40-0:55 | Create all three VMs |
| 0:55-1:25 | Configure Domain Controller and AD DS |
| 1:25-1:40 | Virtual Network Peering |
| 1:40-1:55 | Join VMs to Domain |
| 1:55-2:10 | Install and Configure IIS |
| 2:10-2:25 | DNS Configuration and Testing |
| 2:25-2:40 | User Permissions and Group Policy |
| 2:40-2:55 | Testing and Verification |
| 2:55-3:10 | Record video demonstration |
