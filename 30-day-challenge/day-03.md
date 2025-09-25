# Day 3 - Log collection and artifacts using Sysmon and 

📅 2025-09-21

---
## 🎯 Goal
Day 3 after getting our lab set up, let start collec  ting some sample log. we'll break down the goal in to :
1. Collect real linux auth logs
2. Export windows event or sysmon log

---

## 🎬 Step Taken


### Collect real linux auth logs
After spinning up the Proxmox server and running Ubuntu, we’ll collect real logs from the Ubuntu server and save them as artifacts. For easier access, I’m using SSH via Windows PowerShell instead of directly accessing the server via TTY. Use the following command to log in:
```bash
ssh <USERNAME>@<UBUNTUSERVERADDRESS>
```
![login](assets/day-3/terminal%202025-09-21%20035155.png)
Now that we’ve entered the system, we can start collecting logs. But first, we need to create a directory:
```bash
sudo mkdir -p artifacts
sudo chown $(whoami) /artifacts
```

Next, we’ll grab the last 50 authentication log entries, which will later be split into files:
```bash
sudo tail -n 50 /var/log/auth.log > /tmp/auth_tail.log
```
You can view the logs by typing:
```bash
cat /tmp/auth_tail.log
```
![taillog](assets/day-3/tail%20log%202025-09-2%20141724.png)

Finally, we’ll extract lines containing "sudo", "failed password", "accepted password", or "session opened". Create a script file to split the logs:
```bash
sudo nano split_auth.sh
```
Add the following script:
```bash
#!/bin/bash
mkdir -p /artifacts/linux
INPUT=/tmp/auth_tail.log
OUTDIR=/artifacts/linux
grep -Ei "sudo:|Failed password|Accepted password|session opened" $INPUT | nl -ba | while read -r num rest; do
  idx=$(printf "%03d" "$num")
  echo "$rest" > "$OUTDIR/$(date -u +"%Y-%m-%d_%H-%M-%S")_linux_auth_${idx}.log"
done
```

Give the script execution permissions:
```bash
sudo chmod +x split_auth.sh
```
and run
```bash
./split_auth.sh
```
you can find the artifacts on `/artifacts/` root directory

### Collect log from Sysmon 
After collecting logs from Linux, we now need to gather logs from a Windows server. Start the Windows server on Proxmox. It’s similar to Windows 10—just type `eventvwr.msc` after `windows + r` and you can find a lot of logs.
![winlog](assets/day-3/winlog%202025-09-26%20011017.png)
However, since we already have a Windows Server running on Proxmox, we’ll install Sysmon on it. First, `powershell` as `admin` :
```bash
New-Item -Path C:\artifacts -ItemType Directory -Force
```
create a temp folder
```bash 
$td = "$env:TEMP\sysmon_install"
New-Item -Path $td -ItemType Directory -Force
```
download Sysmon 
```bash
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "$td\Sysmon.zip"
Expand-Archive "$td\Sysmon.zip" -DestinationPath $td
```
pull a vetted config 
```bash 
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "$td\sysmonconfig-export.xml"
```
Install Sysmon with the config
```bash
cd $td
.\Sysmon64.exe -accepteula -i .\sysmonconfig-export.xml
```
Sysmon will now start capturing activity. You can log in to Active Directory or use Remote Desktop to generate logs. To export the logs, use the following command:

```bash
wevtutil epl "Microsoft-Windows-Sysmon/Operational" C:\artifacts\Sysmon_$(Get-Date -Format "yyyy-MM-dd_HH-mm-ss").evtx
```
and just by double click you can see the logs
![sysmonlog](assets/day-3/Sysmonlog%202025-09-26%20014544.png)

## 📝 Notes
The simulation is progressing smoothly so far, and we’ve successfully collected logs from both Linux and Windows systems.