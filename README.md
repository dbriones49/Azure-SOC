# Building a SOC + Honeynet in Azure (Live Traffic)

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/c278c18d-e192-4f6e-9ede-654f03a81479)


# Azure Instances
![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/35fe6cdb-c3b7-4602-aa17-c68c8d91ac28)


# Configured windows registry
Once SQL server was installed, I configured windows registry to provide full permissions to the server. Configurations were made in Windows Registry. Once completed, audit access was configured using auditpol.

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/7326f527-5d12-45c8-93ca-a166269e6556)


# Configured SQL Server Management 
Login auditing was then configured to allow both failed and succesful logins.

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/b243579a-2267-42e4-82dc-14c86d731473)


# Provisioned an attack VM 
An "attack" VM was created to attempt to access the windows VM, and the Linux SQL Server. I attempted three failed logins on each VM, from the "attack" VM.


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/2e40d81c-17a6-4dd9-b1dc-ef8a4ac636a5)


# Reviewed event logs from the Windows VM
I left the windows and Linux VMs on for several hours. From basic filters, I was able to see numerous attempts to access the VMs


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/58a5bc8b-5df1-4411-ad39-fffedf3b8299)

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/68d312e8-b49f-4273-bccd-a4ece55b0ca1)


# Reviewed logs from the Linux VM
From running a few filters, we are also able to see the numerous attempts to access the Linux server

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/ba6cc010-f3d4-4aba-96b7-3828eaf7c4a1)

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/1ace935e-1e26-4eed-92ee-ed8c7245c4bb)

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/f393f876-8bc6-4801-a8f4-51d697aff330)


# Configured Active Directory
Azure Active Directory logs, subscription level logs, and the resource level logs were created in Azure Active Directory to capture log activity.
These will then be used to forward to the centralized log analytics workspace.

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/6b999ec3-ed2a-45e6-a858-46451bbf8388)

# Configured Log Analytics Workshpace and Sentinel
A Log analytics workspace was created. Azure Sentiniel(SIEM) was also deployed. Once Sentinel was deployed a Watchlist was created (list of network blocks) 
and used to derive geolocation from IP addresses from attackers. These will plot attack spots on our SIEM map.


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/eee8d8ab-7860-44d0-bdde-89aa19815bbd)


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/6b59d71a-cafe-4078-9e9f-42e1bcab4079)


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/a55fabff-f3a0-452d-a126-15e39a153237)


# Configured data collection rules. 
Windows Defender for Cloud auto installs an agent on VMs that will allows us to forward logs into the log analytics work space. Once this was enabled, NSG flow logs were created for each security group and flow logs were enabled. 
This will enable Windows Defender for Cloud to analyze traffic and will determine which traffic is malicious and/or benign. 

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/6fba919b-3ab3-408a-8c91-5051f48b4547)








## Introduction

In this project, I built a mini honeynet in Azure and ingested log sources from various resources into a Log Analytics workspace. I then used Microsoft Sentinel to build attack maps, trigger alerts, and create incidents. I measured some security metrics in the insecure environment for 24 hours, applied some security controls to harden the environment, measured metrics for another 24 hours, and then obtained the results below. The metrics we will show are:

- SecurityEvent (Windows Event Logs)
- Syslog (Linux Event Logs)
- SecurityAlert (Log Analytics Alerts Triggered)
- SecurityIncident (Incidents created by Sentinel)
- AzureNetworkAnalytics_CL (Malicious Flows allowed into our honeynet)

## Architecture Before Hardening / Security Controls
![Architecture Diagram](https://i.imgur.com/aBDwnKb.jpg)


## Architecture After Hardening / Security Controls
![Architecture Diagram](https://i.imgur.com/YQNa9Pp.jpg)


## Configured data collection rules.
Configuring the data collection rules helps Windows Defender for Cloud specify which logs from the VMs are forwarded to the log analytics workspace.


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/88bd60d8-b870-4b29-b0ff-26647d5b4cf7)




The architecture of the mini honeynet in Azure consists of the following components:

- Virtual Network (VNet)
- Network Security Group (NSG)
- Virtual Machines (2 windows, 1 linux)
- Log Analytics Workspace
- Azure Key Vault
- Azure Storage Account
- Microsoft Sentinel

For the "BEFORE" metrics, all resources that were originally deployed were xposed to the internet. The Virtual Machines had both their Network Security Groups and built-in firewalls configured to be wide open, and all other resources are deployed with public endpoints visible to the Internet; aka, no use for Private Endpoints.

For the "AFTER" metrics, Network Security Groups were hardened by blocking ALL traffic with the exception of my admin workstation, and all other resources were protected by their built-in firewalls as well as Private Endpoint

## Attack Maps Before Hardening / Security Controls
![NSG Allowed Inbound Malicious Flows](https://i.imgur.com/1qvswSX.png)<br>
![Linux Syslog Auth Failures](https://i.imgur.com/G1YgZt6.png)<br>
![Windows RDP/SMB Auth Failures](https://i.imgur.com/ESr9Dlv.png)<br>

## Metrics Before Hardening / Security Controls

The following table shows the metrics we measured in our insecure environment for 24 hours:
Start Time 2023-03-15 17:04:29
Stop Time 2023-03-16 17:04:29

| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 19470
| Syslog                   | 3028
| SecurityAlert            | 10
| SecurityIncident         | 348
| AzureNetworkAnalytics_CL | 843

## Attack Maps Before Hardening / Security Controls

```All map queries actually returned no results due to no instances of malicious activity for the 24 hour period after hardening.```

## Metrics After Hardening / Security Controls

The following table shows the metrics we measured in our environment for another 24 hours, but after we have applied security controls:
Start Time 2023-03-18 15:37
Stop Time	2023-03-19 15:37

| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 8778
| Syslog                   | 25
| SecurityAlert            | 0
| SecurityIncident         | 0
| AzureNetworkAnalytics_CL | 0

## Conclusion

In this project, a mini honeynet was constructed in Microsoft Azure and log sources were integrated into a Log Analytics workspace. Microsoft Sentinel was employed to trigger alerts and create incidents based on the ingested logs. Additionally, metrics were measured in the insecure environment before security controls were applied, and then again after implementing security measures. It is noteworthy that the number of security events and incidents were drastically reduced after the security controls were applied, demonstrating their effectiveness.
