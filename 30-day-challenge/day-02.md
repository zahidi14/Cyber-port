#  Day 2 - Setting up a Lab for logs

🗓️ 2025-09-20

---

## 🎯 Goal 
so todays goal is just setting for lab no need to overcomplicate it. im going to show homelab setup to generate and analyze logs.

---

## 🎬 Steps Taken
1. Created a new VM in Proxmox:
   - OS: Ubuntu Server 22.04
   - RAM: 1.5GB
   - Disk: 20GB
   - Network: NAT

    ![create VM](assets/day-2/create%20VM.png)

2. Installed basic tools:
   ```bash
   sudo apt update && sudo apt install rsyslog
   sudo systemctl status rsyslog

3. Generated some logs:
    from another device terminal just trigger the logs by typing:
    ```bash
    ssh test@localhost
    (entered wrong password 3 times)
    ```
    
    ![terminal](assets/day-2/password%20trigger%202025-09-20.png)

4. Checked logs with:
    ```bash
    sudo tail -n 20 /var/log/auth.log
    ```

    ![logs](assets/day-2/faillogs%202025-09-20.png)

---

## ✅ Findings
- Invalid login attemps were recorded in `/var/log/auth.log` .
- I can filter logs for SSH events with:

    ```bash 
    grep "Failed password" /var/log/auth.log"

---

## 📋 Notes
The simulation is simple just a proxmox or a vm with 1.5GB resource is enough to generate logs. a lighweight linux just works.
---
