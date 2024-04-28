# Building a SOC + Honeynet in Azure (Live Traffic)


## Introduction

In this project, I built a mini honeynet in Azure and ingested log sources from various resources into a Log Analytics workspace. I then used Microsoft Sentinel to build attack maps, trigger alerts, and create incidents. I measured some security metrics in the insecure environment for 24 hours, applied some security controls to harden the environment, measured metrics for another 24 hours, and then obtained the results below. The metrics we will show are:

- SecurityEvent (Windows Event Logs)
- Syslog (Linux Event Logs)
- SecurityAlert (Log Analytics Alerts Triggered)
- SecurityIncident (Incidents created by Sentinel)
- AzureNetworkAnalytics_CL (Malicious Flows allowed into our honeynet)



![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/c278c18d-e192-4f6e-9ede-654f03a81479)


# Azure Instances
I started with deploying both Linux and a Windows VM, and installed MySQL servers on each. No data was entered in the server.

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/35fe6cdb-c3b7-4602-aa17-c68c8d91ac28)


# Configured windows registry
Once SQL server was installed, I configured windows registry to provide full permissions to the server. Configurations were made in Windows Registry. Once completed, audit access was configured using auditpol.

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/7326f527-5d12-45c8-93ca-a166269e6556)


# Configured SQL Server Management 
Login auditing was then configured to allow both failed and succesful logins.

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/b243579a-2267-42e4-82dc-14c86d731473)


# Provisioned an attack VM 
An "attack" VM was created to attempt to access the windows VM, and the Linux SQL Server. 


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
These were used to forward to the centralized log analytics workspace.

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/6b999ec3-ed2a-45e6-a858-46451bbf8388)

# Configured Log Analytics Workshpace and Sentinel
A Log analytics workspace was created. Azure Sentiniel(SIEM) was also deployed. Once Sentinel was deployed a watchlist was created (list of network blocks) 
and used to derive geolocation from IP addresses from attackers. These will plot attack spots on our SIEM map.


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/eee8d8ab-7860-44d0-bdde-89aa19815bbd)


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/6b59d71a-cafe-4078-9e9f-42e1bcab4079)


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/a55fabff-f3a0-452d-a126-15e39a153237)


## Configured data collection rules.
Configuring the data collection rules helps Windows Defender for Cloud specify which logs from the VMs are forwarded to the log analytics workspace.


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/88bd60d8-b870-4b29-b0ff-26647d5b4cf7)


# Created Data Flow Logs. 
Windows Defender for Cloud auto installs an agent on VMs that will allows us to forward logs into the log analytics work space. Once this was enabled, NSG flow logs were created for each security group and flow logs were enabled. 
This will enable Windows Defender for Cloud to analyze traffic and will determine which traffic is malicious and/or benign. 

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/6fba919b-3ab3-408a-8c91-5051f48b4547)




# Confirmed both VMs are reflected in Log Analytics Workspace. 



![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/68a02ab5-61a6-4e8e-a7e4-e64a1bd9d140)




![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/bd98bafa-4a78-4ed0-ab97-252356dbc41d)


# Confirmed Syslog ingestion.
At this point, I pulled upt the Log Analytics Workshpace and confirmed the logs were imported. It was interesting to see the amount of infiltration efforts attempted by running a KQL brute force qeuery.

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/eb7f2383-7af3-459b-9f9f-394b6f2ecb37)


# Created a test user account to test new log attempts.
I created a test "dummy" account and attempted to login using false credentials, to ensure the new logs were flowing through.


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/29acfd41-fa47-4a47-ae9c-069ed2da63c5)


# Created a back up user account.
I next created a "break glass" account with global administrative access, incase something would have happened to my account.


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/cf30b261-07f5-4ee3-b032-648236b4d52f)


# Exported Activity Logs
Next, I Exported Azure activity logs to the Log Analytics Workspace, and new resource group were created to generate testing logs. 


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/c8d8da22-6c1d-4536-8d33-d6aec6434abf)


# Enabled flow logs from the storage(blob) account.
The Storage logs and Key Vault logs still need to be enabled at this point. Here I enbabled the storage logs to flow to the Log Analytics Workspace.

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/32b14f8a-5505-431d-937d-84c23e5042f8)


# Generated key vault secret
 Once I created a key vault instance, I then generated secret to the vault. Copying the secret to the clipboard will generate a log for testing purposes. Next Ienabled the diagnostic settings to send the audit logs into the Log Anayltics workspace.
 
 ![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/fe4d2ca2-73ba-49a1-aa9d-e56e5f76bef0)


 ![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/8bb69c8a-86f4-44c2-8b53-bffc8e4a8d0f)

# Blog log test
Finally, I conducted a query test for the blog logs.

 ![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/fb8c54ab-dd0c-4721-a300-3a3fda5a1804)


 # Word Map Construction
 
I used pre-built JSON Maps to reduce the number of errors.
These maps will display the locations of the threat actors from a global standpoint. The maps were created to track the following:

-Map 1 will track failed authentications for the windows VM(remote desktop authentications)
-Map 3 will track SSH auth for linux VM
-Map 3 will track attacks on the Microsoft MySQL server
-Map 4 will track alicious inbound flows for the network security group,

To construct the maps, JSON code was entered a Sentinel workbook.


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/9a82161c-af1f-4978-841e-0c60208a7db7)


From here(maps are below), we can extract the map queries in Sentinel and insert them into Log Analytics Workspace, and perform deep dives on specific security event using the geodata.



![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/925c7b76-28ad-4246-b0a9-b683a424cbe4)





# New Test Rules Created
Next, I manually created analytics rules to create alerts, which will generate incidents within the environment. 


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/67fe78b4-850c-4329-8029-a424f8248a0c)



![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/880fc1bc-4f93-42a8-af13-57ecfa4d2715)



# Tested New Rules
I completed 10 brute force attempts from the windows VM to confirm if this would trigger our new rules. Sure enough, it did.


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/32226bcc-6d2a-4fb5-b09e-4a9bab82f5bc)



# Final Rules
Now that I had a successful test run with flagging incidents in Sentinel, I deleted the test rules and results, and enterd in the final query rules for Brute Force Attempts, Malware Detected, Privilege Escalation, and Lateral Movement.


![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/f3364771-7dc3-4984-a745-79f622797e41)







## Architecture Before Hardening / Security Controls
![Architecture Diagram](https://i.imgur.com/aBDwnKb.jpg)


## Architecture After Hardening / Security Controls
![Architecture Diagram](https://i.imgur.com/YQNa9Pp.jpg)





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

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/84d0a0b2-f1f3-4a00-b3eb-7cca032af758)








## Final testing of rules
Finally, I manually triggered and tested some of the custom analytic rules inside of Azure Active Directory. These rules address Brute Force attempts in both MSSQL and Azure Active Directory, Malware, Possible Privilege Escalation, Windows Host Firewall Tampering, and Excessive Password Resets. To test them, I copied the queries from the analytics screen within Sentinel and ran them in Log Analytics. I also ensured they were visible in the "incidents" screen within Sentinel. For the malware test, I used a malware script and ran it within the windows VM. To trigger the Windows Host Firewalls, I enbaled the firewalls in Windows VM defender, then unabled them again. 

![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/39e3bb96-daee-4bf8-acf9-90793eff26cf)



![image](https://github.com/dbriones49/Azure-SOC/assets/143753667/512aeb76-462a-43e3-956c-04fc37e36f99)






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
