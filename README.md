markdown

## Hybrid Authentication System — FreeIPA + NFS + Ubuntu Client

In modern IT environments,maintaning a unified and secured authentication system across multiple platforms is essential for efficiency and centralized management. This guide demonstrates the set up of Hybrid Authentication System using Freeipa on Rocky Linux as the primary LDPA server,integrated with Ubuntu 24.40 as LDPA client through SSSD(System Security Services Daemon).The configuration ensures seamless user authentication across platforms,centralized identity management,and consistent access control.Additionally,it implements cross-platform user home directories using NFS(Network File System),maintaning consistent UID/GID mappings,and enforces sudo policies.

markdown

## Hybrid Authentication System — FreeIPA + NFS + Ubuntu Client


---


## 1. Initial Setup and Firewall (Server - Rocky Linux)
```bash
# Update the system
$ sudo dnf update -y
```


---


## Set the hostname (use your actual FQDN)
```bash
$ sudo hostnamectl set-hostname ipa-server.lab.local
```


---


## Enable and start the firewall service
```bash
$ sudo systemctl enable --now firewalld
$ sudo systemctl start firewalld
```


---


## Open required FreeIPA ports
```bash
$ sudo firewall-cmd --add-service=freeipa-server --permanent
$ sudo firewall-cmd --reload
```
2. Install and Configure FreeIPA


---
## Install the FreeIPA server and DNS components
Run the Installation Wizard
```bash
$ sudo dnf install freeipa-server freeipa-server-dns -y
```


---


## Start the interactive setup with integrated DNS
```bash
$ sudo ipa-server-install --setup-dns
```


---


During setup, be sure to:

Set your Domain name (e.g., lab.local)

Set your Realm name (e.g., LAB.LOCAL)

Configure DNS forwarders (e.g., 8.8.8.8,1.1.1.1)

Set strong passwords for the Directory Manager and IPA admin user

3. Verification
Once installation completes, authenticate and perform a basic check.


---


## Get a Kerberos ticket for the 'admin' user
```bash
$ kinit admin
```


---

## Verify a user can be found
```bash
$ ipa user-find
```


---


Configure NFS for Home Directories (Server)
Setting up NFS allows user home directories to be stored centrally on the server and accessed by any domain client.

1. Install and Start NFS



----


## Install NFS utilities
```bash
$ sudo dnf install nfs-utils -y
```


---



## Enable and start NFS server service
```bash
$ sudo systemctl enable --now nfs-server
$ sudo systemctl start nfs-server
```


---
2. Create and Export Directory


---



## Create parent directory for IPA users’ home directories
```bash
$ sudo mkdir -p /home/ipausers
```

---

## Change ownership to prevent initial permission issues
```bash
$ sudo chown -R nobody:nogroup /home/ipausers
```


---



## Add export definition to /etc/exports
```bash
$ echo "/home/ipausers *(rw,sync,no_root_squash)" | sudo tee -a /etc/exports
```


---



## Export the directories immediately
```bash
$ sudo exportfs -r
```


---


3. Configure Firewall for NFS


---



## Allow NFS services through the firewall
```bash
$ sudo firewall-cmd --add-service=nfs --add-service=rpc-bind --add-service=mountd --permanent
$ sudo firewall-cmd --reload
```


---


Client Setup — Ubuntu (LDAP Client using SSSD)
1. Initial Setup and Packages


---


## Update the system
```bash
$ sudo apt update -y
```


---
## Set the hostname (use your actual FQDN)
```bash
$ sudo hostnamectl set-hostname ubuntu-client.lab.local
```


---


## Install required client packages
```bash
$ sudo apt install freeipa-client sssd-ldap realmd oddjob oddjob-mkhomedir adcli -y
```


2. Joining Ubuntu to the FreeIPA Domain



---


## Join the domain and automatically configure home directory creation
```bash
$ sudo ipa-client-install --mkhomedir
```


---


During the installation, you will be prompted for:

Domain name - ipa-server.lab.local # as an example

Admin username and password - ADMIN@LAB.LOCAL 

3. Mount NFS Home Directories (Client)
The local mount point must match the home directory path used on the server (/home/ipausers).


---



## Install NFS utilities
```bash
$ sudo apt install nfs-common -y
```


---



## Create local mount point
```bash
$ sudo mkdir -p /home/ipausers
```


---



## Mount the NFS share temporarily (replace with your server FQDN)
```bash
$ sudo mount ipa-server.lab.local:/home/ipausers /home/ipausers
```


---


## Make the mount permanent in /etc/fstab
```bash
$ echo "ipa-server.lab.local:/home/ipausers /home/ipausers nfs defaults 0 0" | sudo tee -a /etc/fstab
```

4. Test Authentication
You should now be able to look up and log in as a FreeIPA user.


---


## Look up the admin user information
```bash
$ id admin
$ getent passwd admin
```


---


## Log in as the admin user
```bash
$ su - admin
```

---


The home directory should automatically be created on the NFS share.

5. UID/GID Consistency Check
Verify that the server and client agree on the user and group IDs — this is essential for NFS consistency.


---


## On Rocky Linux (Server)
```bash
$ ipa user-show admin | grep UID
$ ipa group-show admins | grep GID
```

---


On Ubuntu (Client):
```bash
$ getent passwd admin  # Check the UID 
$ getent group admins  # Check the GID
```


---


The UID and GID values must be identical on both systems.

You’ve now configured:

A FreeIPA server on Rocky Linux for centralized authentication

Ubuntu client joined to the IPA domain using SSSD

NFS shared home directories for domain users

Consistent UID/GID mapping between systems

Author: Harison Kimutai Chirchir


Email: harisonchirchir2@gmail.com


