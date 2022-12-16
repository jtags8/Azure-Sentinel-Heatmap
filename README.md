# Azure-Sentinel-Heatmap

<h2>Description</h2>
A basic project where I used Azure to create a honeypot VM, created a custom log and SIEM to gather data, and plot the data on a heatmap in Azure Sentinel.
<br />


<h2>Languages and Utilities Used</h2>

- <b>Terminal</b> 
- <b>ping</b> 
- <b>ipgeolocation[.]io</b> 

<h2>Environments Used </h2>

- <b>Windows 11</b>
- <b>Microsoft Azure</b>

<h2>Process</h2> 

<b>1. Create a VM in Azure</b>
- Purpose: to setup a VM that will act as a honeypot
- First, go to Create Virtual Machine. Create new resource group, which I named "honeypotlab". Name the VM, which I named "honeypot-vm". Choose a size, but usually the default one is sufficient. Create username and password that is easy to remember to log in later. Click next, then next again to go to Networking. In networking, under NIC network security group, check off Advanced, then Create New. There you should delete default rule that is already there. Create new rule with * destination port, action allow, priority 100. Name the rule anything, but I named it “DANGER_ANYTHING_IN". This allows all traffic into VM. Once finished, click Review + Create, and then Create.


<b>2. Log Analytics Workspace</b>
- Purpose: to ingest logs from VM
- Create new. Resource group, select “honeypotlab”. Name the Log Analytics Workspace, which I named "law-honeypot1". Keep region default US East 2.


3.	Security Center (Microsoft Defender for Cloud) – enable ability to gather logs from VM to LAW


4.	Setup Sentinel
- <b>Search Microsoft Sentinel --> Click LAW where all logs are --> Add</b>


<b>5.	Log into the VM</b>
- Note Public IP in Azure VMs. Mine was "20.119.183.185".
- Log into the VM with remote desktop connection on host
- Use different account, then accept certificate warning
- Click no for sharing --> 

6. Event Viewer
- Windows Logs, then click Security
- Look for 4625 Audit Failure 

![Screenshot 2022-12-15 150542](https://user-images.githubusercontent.com/114617963/207998455-8c8c6f08-8cba-4c0c-b3f6-872a8393d82e.png)
 
- Take note of IP address. Use ipgeolocation.io and look up the IP address. Use this info to create custom log and send to the LAW.
i.	"ip": 
ii.	"country_name": 
iii.	"state_prov":  
iv.	"city": 
v.	"latitude": 
vi.	"longitude": 
vii.	"time_zone": 
viii.	"isp": 
ix.	"currency": 
x.	"country_flag": USA</b>

7.	Ping test and disable Windows Firewall on VM
- Test ping in command line the public IP (ping 20.119.183.185). Should have 4 packets lost. Go back to VM, search wf.msc. Firewall Properties, then disable firewall on domain, private, public.
- Test ping again on host machine – ping ip, should work

8.	Download .ps1 file
- <b>GitHub - joshmadakor1/Sentinel-Lab --> Go to the Custom Security Log Exporter .ps1 file --> Copy whole code --> Go to Windows Powershell ISE  New  Paste code  Save to Desktop --> Need to get own API key from ipgeolocation.io -0130bdb421fd474d8cb3d085d9edb619 --> Press Play button to run script --> Creates failed logon data in C:/ProgramData/failed_rdp</b>


9.	Back to Azure
- <b>LAW  Click law-honeypot1  Custom Logs  Add custom log --> Need to copy and paste log data from VM to host computer, and make new file --> C:\ProgramData\failed_rdp.log --> Then name and create log --> In Logs, can enter FAILED_RDP_WITH_GEO_CL in first line, then click Run --> Bunch of logs show up (during this process, an attacker from Mexico City was actually brute forcing with username “administrator” IP 189.254.74.74)</b>

![Screenshot 2022-12-15 184800 BRUTE FORCE](https://user-images.githubusercontent.com/114617963/207998335-3b68b5f5-ade6-4823-8484-66d2819bc267.png)

![Screenshot 2022-12-15 184844 BRUTE FORCE in LAW](https://user-images.githubusercontent.com/114617963/207998326-7c80b34c-654f-442b-91f1-17637e4f6550.png)


10.	Extract RawData to make fields
- <b>Right click one of the logs, and click Extract --> Highlight numbers next to latitude, enter name “latitude”, then drop down menu select Numeric --> Repeat for all other data labels (numeric for latitude longitude, time/date for the timestamp, text for all others)</b>


11.	Make map in Sentinel
- Search Sentinel --> Click Workbook  Add New  Edit  remove the two default widgets --> Add  Add query  copy and paste this query
i.	FAILED_RDP_WITH_GEO_CL | summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
ii.	| where destinationhost_CF != "samplehost"
iii.	| where sourcehost_CF != ""

![Screenshot 2022-12-15 191850](https://user-images.githubusercontent.com/114617963/207998220-41556d19-5745-46cb-a322-0718db013567.png)

- Visualization  Map --> Size  Large or Full --> Fix map settings (latitude, longitude, metric label by label_CF, metric value change to event_count, size by event_count) --> Save workbook --> Can set to auto-refresh --> 


![Screenshot 2022-12-15 191900](https://user-images.githubusercontent.com/114617963/207998156-c00be791-304e-4164-87bb-7661f77dcf15.png)

![Screenshot 2022-12-15 194809](https://user-images.githubusercontent.com/114617963/207998110-c2f909d5-403d-4cdd-a831-8ebf5ae94642.png)

12. <b>Wait for attacks to come!</b>


<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
