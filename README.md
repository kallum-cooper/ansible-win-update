# Automated Windows Updates with Ansible + WinRM

This repository contains an Ansible configuration for automating **Windows Server updates** across a domain-joined environment using **WinRM (CredSSP)**.  

It sets up scheduled patching and logging to keep systems updated with minimal manual effort.

---

##  Features
- Automated Windows Updates with Ansible
- WinRM configured for remote management (CredSSP support)
- Monthly scheduled run via `cron` (2nd of every month at 02:00)

---

##  Prerequisites
- **Ansible control node** (Linux server with Ansible installed)
- **Windows hosts** joined to your domain
- Network access to port `5985/TCP` from Ansible to target hosts
- Local or Group Policy allowing WinRM auto-configuration

---

##  Setup

### 1. Configure Windows Hosts
A startup script (via GPO or manual run) enables and configures WinRM:

```powershell
Set-ExecutionPolicy RemoteSigned -Force

# Enable and configure WinRM
winrm quickconfig -force
Enable-PSRemoting -Force

# Allow basic authentication and unencrypted traffic
Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true
Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value $true

# Enable CredSSP support
Enable-WSManCredSSP -Role Server -Force

# Open firewall port 5985
New-NetFirewallRule -Name "Allow WinRM" `
    -DisplayName "Allow WinRM" `
    -Protocol TCP `
    -LocalPort 5985 `
    -Direction Inbound `
    -Action Allow `
    -Profile Any `
    -Enabled True
```
Clone This Repository
```
git clone https://github.com/kallum-cooper/ansible-win-update.git
cd ansible-win-update
```

Update your inventory with target hosts and then run:
```
ansible-playbook playbooks/windows_update.yml
```

Add a Cron job:
```
0 2 2 * * cd /home/ansible/ansible && ansible-playbook playbooks/windows_update.yml
```