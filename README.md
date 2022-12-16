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
- <b}Microsoft Azure</b>


<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>

- <b>Process</b> 
1. Create a VM in Azure
- Purpose: setup a VM that will act as a honeypot
- Create Virtual Machine --> Create new resource group – "honeypotlab" --> Name VM – "honeypot-vm" --> Fixed size --> Create username and password that is easy to remember to log in later --> Click next, next again to go to Networking --> NIC network security group  Advanced  Create New --> Delete default rule --> Create new rule with * destination port, action allow, priority 100, name anything “DANGER_ANYTHING_IN" (This allows all traffic into VM) --> Review + Create  Create
2. Log Analytics Workspace
– Purpose: to ingest logs from VM
- Create new --> Resource group “honeypotlab” --> Name it – "law-honeypot1" --> Region – Default US East 2
3.	Security Center (Microsoft Defender for Cloud) – enable ability to gather logs from VM to LAW
4.	Setup Sentinel
- Search Microsoft Sentinel --> Click LAW where all logs are --> Add
5.	Go back to VMs
- Note Public IP - 20.119.183.185; log into it with a remote desktop
6.	Go to host computer and use Remote Desktop Connection
- Connect to public IP --> Use different account --> Accept certificate warning
7.	On VM
- Set up edge, click no for sharing --> Event Viewer  Windows Logs  Security --> Look for 4625 Audit Failure --> Take note of IP address --> Use ipgeolocation.io
i.	"ip": 
ii.	"country_name": 
iii.	"state_prov":  
iv.	"city": 
v.	"latitude": 
vi.	"longitude": 
vii.	"time_zone": 
viii.	"isp": 
ix.	"currency": 
x.	"country_flag": USA
- Use this info to create custom log and send to the LAW
- Use Sentinel to plot attackers on map
8.	Back on host
- Test ping in command line the public IP (ping 20.119.183.185) --> Should have 4 packets lost --> Go back to VM, search wf.msc  Firewall Properties  disable firewall on Domain, private, public --> Host machine – ping ip, should work
9.	Download .ps1 file
- GitHub - joshmadakor1/Sentinel-Lab --> Go to the Custom Security Log Exporter .ps1 file --> Copy whole code --> Go to Windows Powershell ISE  New  Paste code  Save to Desktop --> Need to get own API key from ipgeolocation.io -0130bdb421fd474d8cb3d085d9edb619 --> Press Play button to run script --> Creates failed logon data in C:/ProgramData/failed_rdp
10.	Back to Azure
- LAW  Click law-honeypot1  Custom Logs  Add custom log --> Need to copy and paste log data from VM to host computer, and make new file --> C:\ProgramData\failed_rdp.log --> Then name and create log --> In Logs, can enter FAILED_RDP_WITH_GEO_CL in first line, then click Run --> Bunch of logs show up (during this process, an attacker from Mexico City was actually brute forcing with username “administrator” IP 189.254.74.74)
11.	Extract RawData to make fields
- Right click one of the logs, and click Extract --> Highlight numbers next to latitude, enter name “latitude”, then drop down menu select Numeric --> Repeat for all other data labels (numeric for latitude longitude, time/date for the timestamp, text for all others)
12.	Make map in Sentinel
- Search Sentinel --> Click Workbook  Add New  Edit  remove the two default widgets --> Add  Add query  copy and paste this query 
i.	FAILED_RDP_WITH_GEO_CL | summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
ii.	| where destinationhost_CF != "samplehost"
iii.	| where sourcehost_CF != ""
- Visualization  Map --> Size  Large or Full --> Fix map settings (latitude, longitude, metric label by label_CF, metric value change to event_count, size by event_count) --> Save workbook --> Can set to auto-refresh --> Wait for attacks to come!
