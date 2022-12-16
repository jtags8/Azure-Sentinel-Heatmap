# Azure-Sentinel-Heatmap

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
a.	Note Public IP - 20.119.183.185; log into it with a remote desktop
6.	Go to host computer and use Remote Desktop Connection
a.	Connect to public IP
b.	Use different account
c.	Accept certificate warning
7.	On VM
a.	Set up edge, click no for sharing
b.	Event Viewer  Windows Logs  Security
c.	Look for 4625 Audit Failure
d.	Take note of IP address – JEFF-PC is 73.195.193.100
e.	Use ipgeolocation.io
i.	"ip": 73.195.193.100
ii.	"country_name": United States
iii.	"state_prov": New Jersey
iv.	"city": Southampton Township
v.	"latitude": 39.92802
vi.	"longitude": -74.77175
vii.	"time_zone": America/New_York
viii.	"isp": Comcast
ix.	"currency": US Dollar
x.	"country_flag": USA
f.	Use this info to create custom log and send to the LAW
g.	Use Sentinel to plot attackers on map
8.	Back on host
a.	Test ping in command line the public IP (ping 20.119.183.185)
b.	Should have 4 packets lost
c.	Go back to VM, search wf.msc  Firewall Properties  disable firewall on Domain, private, public
d.	Host machine – ping ip, should work
9.	Download .ps1 file
a.	GitHub - joshmadakor1/Sentinel-Lab
b.	Go to the Custom Security Log Exporter .ps1 file
c.	Copy whole code
d.	Go to Windows Powershell ISE  New  Paste code  Save to Desktop
e.	Need to get own API key from ipgeolocation.io -0130bdb421fd474d8cb3d085d9edb619
f.	Press Play button to run script
g.	Creates failed logon data in C:/ProgramData/failed_rdp
10.	Back to Azure
a.	LAW  Click law-honeypot1  Custom Logs  Add custom log
b.	Need to copy and paste log data from VM to host computer, and make new file
c.	C:\ProgramData\failed_rdp.log
d.	Then name and create log
e.	In Logs, can enter FAILED_RDP_WITH_GEO_CL in first line, then click Run
f.	Bunch of logs show up (during this process, an attacker from Mexico City was actually brute forcing with username “administrator” IP 189.254.74.74)
11.	Extract RawData to make fields
a.	Right click one of the logs, and click Extract
b.	Highlight numbers next to latitude, enter name “latitude”, then drop down menu select Numeric
c.	Repeat for all other data labels (numeric for latitude longitude, time/date for the timestamp, text for all others)
12.	Make map in Sentinel
a.	Search Sentinel
b.	Click Workbook  Add New  Edit  remove the two default widgets
c.	Add  Add query  copy and paste this query 
i.	FAILED_RDP_WITH_GEO_CL | summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
ii.	| where destinationhost_CF != "samplehost"
iii.	| where sourcehost_CF != ""
d.	Visualization  Map
e.	Size  Large or Full
f.	Fix map settings (latitude, longitude, metric label by label_CF, metric value change to event_count, size by event_count)
g.	Save workbook
h.	Can set to auto-refresh
i.	Wait for attacks to come!
