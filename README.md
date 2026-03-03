# MST300 Project 1 – Active Directory Domain Services in Azure

Welcome to MST300 Project 1!

In this project, you will design and implement a complete Azure cloud infrastructure using Virtual Networks, Azure Bastion, Windows Server, and Active Directory Domain Services. The focus is on cloud architecture, secure connectivity, domain configuration, and network segmentation, preparing you for real-world enterprise cloud deployments.

📚 Project Objectives:
+ Design and deploy a multi-VNet Azure architecture
+ Configure Azure Bastion for secure VM access without public IPs
+ Deploy and configure a Windows Server Domain Controller (ADDS)
+ Create and manage domain users and organizational structure
+ Join multiple VMs to an Active Directory domain
+ Configure Virtual Network Peering for cross-network communication
+ Deploy and configure IIS on a web server
+ Access the web server using a fully qualified domain name (FQDN)Document all
+ Configuration steps with screenshots and commands

🏗 Project Components:

You will build the following Azure resources:
+ 1 Resource Group
+ 3 Virtual Networks
+ 4 Subnets (including AzureBastionSubnet)
+ 1 Azure Bastion Host
+ 1 Domain Controller VM (Windows Server 2019)
+ 1 Web Server VM (IIS)
+ 1 Client VM (Windows 10)
+ Virtual Network Peering between all VNets

🧩 Tips & Best Practices:
+ Ensure all VNets are properly peered before attempting domain join.
+ Verify DNS settings point to the Domain Controller’s private IP.
+ Always test connectivity before installing roles or joining domains.
+ Use domain credentials when logging in via Azure Bastion (not local admin).
+ Take screenshots during every major configuration step for grading proof.
+ Delete unused resources after testing to avoid unnecessary Azure charges.

🧪 What This Project Demonstrates:
+ Cloud infrastructure deployment in Microsoft Azure
+ Enterprise-level Active Directory setup
+ Secure remote access architecture
+ Network segmentation and peering
+ Domain-based authentication
+ FQDN-based service access

📖 References:
+ Azure Bastion Documentation (https://learn.microsoft.com/en-us/azure/bastion/)
+ Azure Virtual Network Documentation (https://learn.microsoft.com/en-us/azure/virtual-network/)
+ Azure Virtual Network Peering Documentation (https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)
+ Windows Server ADDS Documentation (https://learn.microsoft.com/en-us/windows-server/)
+ Microsoft Learn – Azure Virtual Machines (https://learn.microsoft.com/en-us/azure/virtual-machines/)
