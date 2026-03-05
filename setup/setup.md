# MST300 Project 1: Active Directory Domain Services in Azure

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

# Part 2: Virtual Networks Configuration

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

# Part 3: Azure Bastion Setup
Azure Bastion provides secure RDP/SSH access without exposing VMs to the public internet.

# Steps:

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

# Part 4: Domain Controller VM
The Domain Controller manages authentication, user accounts, and DNS.

# Steps:

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

# Part 5: Active Directory Domain Services

Configure the Domain Controller to provide Active Directory services.

# Part 5.1: Install AD DS Role

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

# Part 6: Webserver VM
Create a Windows Server VM that will host IIS web server.

# Steps:
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

# Part 7: Client VM
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

# Part 8: Virtual Network Peering
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

# Part 9: Joining VMs to Domain

# Part 9.1: Configure DNS Settings

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

# Part 10: IIS Configuration
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

# Part 11: DNS Configuration
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

# Part 12: User Permissions
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

# Part 13: Testing and Verification
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
