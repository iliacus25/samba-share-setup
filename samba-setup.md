# Samba setup technical walkthrough

## Setup Process

1. Updated package lists and system packages:

```bash
sudo apt update && sudo apt upgrade
```

2. Install Samba server and client tools:

```bash
sudo apt install samba
```

3. Ensure Samba service is running:

```bash
sudo systemctl status smbd
```
Should show that service is active and enabled. This is to ensure that the service will start automatically when laptop is turned on. 

Disabled the service whilst editing the config file:

```bash
# Stop service:
systemctl stop smbd

# Check status again:
sudo systemctl status smbd
```

 4. Locate config file:

 ```bash
 # Change directory
 cd /etc/samba

 # List contents
 ls
```
Located file `smb.conf` and renamed to `smb.conf.bak`then created new file:

```bash
sudo nano smb.conf
```

This was in order to preserve the original as a backup whilst creating new config file from scratch.

---

## Creating config file

```ini
[Global]
server string = <MySamba>           ; Description of the Samba server to identify it on the network
workgroup = <WORKGROUP>             ; Name set to whatever you want
security = user                     ; Suitable setting for small home network
map to guest = never                ; Set to never for security and privacy
name resolve order = bcast host     ; Check for hostnames via network broadcast before lookup in local /etc/hosts file
include = /etc/samba/shares.conf    ; Split shares into a second config file for clarity

; Security settings
hosts allow = <lan_subnet>; Alllowed internal network range
hosts deny = <all_others>; Block external hosts

; Enforce SMB2 as minimum protocol standard:
server min protocol = SMB2
client min protocol = SMB2
```

### Separate shares file 

This can be included in the Global settings file.

```ini
[<Directory_name>]              ; Create desired directory name 
path = /share/<directory_name>  ; Create path to shared directory
browseable = yes                ; Makkes share visible to the network

; Allow write permissions to the share
read only = no
writable = yes

; 
valid users = <username>        ; Restricts access to this Samba user
force user = <username>         ; Files created as this user (prevents permission headaches)
force group = <groupname>       ; Files created as this group

; Set permissions for shared directory
create mask = 0664              ; Permissions for new files
directory mask = 0775           ; Permissions for new directories
```

---

## Create share directory(s), users, and groups

Create directory with desired path to match the config file:

```bash
sudo mkdir -p /share/<directory_name>
```

### User management

Some tutorials recommend creating a dedicated Samba user and group to isolate file access.
Create group to match group named in the config file:

```bash
sudo groupadd --system <groupname>

# Confirm with 
cat /etc/group
```

Create user:

```bash
sudo useradd --system --no-create-home --group <groupname> -s /bin/false <username>

# No home directory needed for this user, logins as this user should be prevented.
```
Set permissions:
```bash
sudo chown -R <username>:<groupname> /share

# Give group write permission:
sudo chmod -R g+w /share

# Check permissions:
ls -l /share
```


In this setup, the system’s primary user account was reused for Samba access.
This was chosen because:

- The environment is single-user

- All devices are personally owned

- No external access is enabled

- Simplicity and maintainability were prioritised

---

## Restart Samba and check configuration

```bash
sudo systemctl start smbd

# Check status:
sudo systemctl status smbd

# Check Samba config for errors:
testparm

# Set Samba password
sudo smbpasswd -a <username>

# List the SMB shares offered by a server:
smbclient -L <host> -U <username>
```
This last command should return a list of available shares.  These are the folders that Windows will be available to mount. 
Final entry should be `IPC$`, or Inter-Process Communication share, built in control channel. 

---

## UFW (Uncomplicated Firewall) setup and troubleshooting.

Ran into difficulties getting Windows to access the share which turned out to be an error in the config file- had created the config file as if to create a dedicated Samba user, and the decided to have it set as my own username, but failed to go back and update the config with the change.

As part of troubleshooting, tested to see if UFW was causing the problem with:

```bash 
sudo ufw status
```

UFW turned out to be inactive, so set it up for improved security:

```bash
# Set the firewall default policy to block all incoming network traffic unless explicitly allowed:
sudo ufw default deny incoming

# Set firewall to allow all outgoing network traffic unless explicitly blocked:
sudo ufw default allow outgoing

# Set firewall to explicitly allow Samba:
sudo ufw allow samba

# Check settings:
sudo ufw status verbose
```
---

## Final steps

### Mapped the Samba share as a drive on Windows

On Windows:

1. Open File Explorer

2. Right-click This PC

3. Click Map network drive

4. Assign drive letter and locate folder, the click finish.

### Set up DHCP reservation for Linux laptop on router.

Navigated to router default IP (often 192.168.1.1), logged in, and added the Linux laptop to the DHCP reservation list to prevent IP rotation on lease expiration from breaking the Samba connection.