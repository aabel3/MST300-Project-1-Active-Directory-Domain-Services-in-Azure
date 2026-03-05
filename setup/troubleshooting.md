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


