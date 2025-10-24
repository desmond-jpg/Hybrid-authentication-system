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

<img width="903" height="397" alt="Screenshot from 2025-10-23 16-18-28" src="https://github.com/user-attachments/assets/552583a7-2fb8-4a69-9239-4f203b3f2c16" />

---

<img width="913" height="206" alt="Screenshot from 2025-10-23 16-19-27" src="https://github.com/user-attachments/assets/effe942e-1d17-4f9e-928d-1dcd698c29b5" />


---


## Set the hostname (use your actual FQDN)
```bash
$ sudo hostnamectl set-hostname ipa-server.lab.local
```

---

<img width="926" height="28" alt="Screenshot from 2025-10-23 16-21-50" src="https://github.com/user-attachments/assets/8eb93fe1-47fd-453f-9aa5-2ba62b451417" />

---


## Enable and start the firewall service
```bash
$ sudo systemctl enable --now firewalld
$ sudo systemctl start firewalld
```

---
<img width="872" height="531" alt="Screenshot from 2025-10-23 16-55-07" src="https://github.com/user-attachments/assets/f79e7152-092e-4564-bdbc-5d4a1431f1aa" />

---

Sometimes you might encounter an error after running those command due to firewall services not recognised like the one above,
This is something fixable trough nano or vi  the preistall editor
1. Nano you just run this command and enter the xml file below
```bash
$ sudo nano /etc/firewalld/services/freeipa-server.xml
```
After that press CRTL + O, followed by ENTER then exit using CRTL + X

2 VI editor command 
```bash
$ sudo vi /etc/firewalld/services/freeipa-server.xml
```
This is the xml file that should be enter in either of the editors i mention above
sudo tee /etc/firewalld/services/freeipa-server.xml > /dev/null <<EOF
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>FreeIPA Server</short>
  <description>FreeIPA Identity Management Server</description>
  <port protocol="tcp" port="80"/>
  <port protocol="tcp" port="443"/>
  <port protocol="tcp" port="389"/>
  <port protocol="tcp" port="636"/>
  <port protocol="udp" port="88"/>
  <port protocol="tcp" port="88"/>
  <port protocol="udp" port="464"/>
  <port protocol="tcp" port="464"/>
  <port protocol="tcp" port="53"/>
  <port protocol="udp" port="53"/>
</service>
EOF

How to save on vi editor
- Press i "you'll will see --INSERT MODE"
- Press esc to exit INSERT MODE
- Type :wq to save
- Then finally press enter
  
---

## Open required FreeIPA ports
```bash
$ sudo firewall-cmd --add-service=freeipa-server --permanent
$ sudo firewall-cmd --reload
```

---
<img width="897" height="49" alt="Screenshot from 2025-10-23 16-57-40" src="https://github.com/user-attachments/assets/fc8555c5-5f6a-4935-87b3-0de1d33da992" />

---
<img width="897" height="49" alt="Screenshot from 2025-10-23 16-57-10" src="https://github.com/user-attachments/assets/1c81a231-21c2-4951-a8eb-a38c8de15f20" />

---

2. Install and Configure FreeIPA

---
## Install the FreeIPA server and DNS components
Run the Installation Wizard
```bash
$ sudo dnf install freeipa-server freeipa-server-dns -y
```

---
<img width="931" height="641" alt="Screenshot from 2025-10-23 17-03-54" src="https://github.com/user-attachments/assets/ab09867f-6daf-40ca-b595-48b8e655cc82" />

---
<img width="950" height="105" alt="Screenshot from 2025-10-23 17-04-28" src="https://github.com/user-attachments/assets/c2ff72e3-90ef-46bd-aa7b-9636fe9b8923" />

---


## Start the interactive setup with integrated DNS
```bash
$ sudo ipa-server-install --setup-dns
```

---
<img width="844" height="625" alt="Screenshot from 2025-10-23 17-56-52" src="https://github.com/user-attachments/assets/4b906151-89be-4190-94e0-f79457d56142" />

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
<img width="882" height="62" alt="Screenshot from 2025-10-23 18-07-12" src="https://github.com/user-attachments/assets/e4ed044b-e3a9-406a-a348-498a2acd015f" />


---

## Verify a user can be found
```bash
$ ipa user-find
```

---
<img width="905" height="348" alt="Screenshot from 2025-10-23 18-14-41" src="https://github.com/user-attachments/assets/65da3596-1375-4325-93bd-8fafc597420f" />


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

<img width="800" height="156" alt="Screenshot from 2025-10-23 20-15-35" src="https://github.com/user-attachments/assets/5b7b3294-e705-409f-aa13-0eb48e096017" />

---



## Enable and start NFS server service
```bash
$ sudo systemctl enable --now nfs-server
$ sudo systemctl start nfs-server
```

---
<img width="975" height="66" alt="Screenshot from 2025-10-23 20-16-57" src="https://github.com/user-attachments/assets/fa84c0f3-2c53-4d81-9ca7-f4249211ba12" />

---
<img width="966" height="116" alt="Screenshot from 2025-10-23 20-18-27" src="https://github.com/user-attachments/assets/a46fe4b5-a754-4062-b038-99a972ec3637" />

---

2. Create and Export Directory

---



## Create parent directory for IPA users’ home directories
```bash
$ sudo mkdir -p /home/ipausers
```

---
<img width="811" height="25" alt="Screenshot from 2025-10-23 21-45-43" src="https://github.com/user-attachments/assets/3c35c7e3-67fb-43c1-8527-df3ae1d108f6" />


---

## Change ownership to prevent initial permission issues
```bash
$ sudo chown -R nobody:nobody /home/ipausers
```

---
<img width="938" height="26" alt="Screenshot from 2025-10-23 20-28-53" src="https://github.com/user-attachments/assets/8a970d1e-9f9b-4ef2-8339-ac8ec9496479" />

---

## Add export definition to /etc/exports
If you encounter error when running this command, IP address can be used to replace the asteric part
```bash
$ echo "/home/ipausers *(rw,sync,no_root_squash)" | sudo tee -a /etc/exports
```

---
<img width="979" height="54" alt="Screenshot from 2025-10-23 20-34-32" src="https://github.com/user-attachments/assets/84deae19-193f-4145-92e9-da67d788cd9c" />

---



## Export the directories immediately
```bash
$ sudo exportfs -r
```

---
<img width="946" height="23" alt="Screenshot from 2025-10-23 20-35-45" src="https://github.com/user-attachments/assets/35e433e5-bb37-425f-9d12-8ec97ca82187" />

---

3. Configure Firewall for NFS

---

## Allow NFS services through the firewall
```bash
$ sudo firewall-cmd --add-service=nfs --add-service=rpc-bind --add-service=mountd --permanent
$ sudo firewall-cmd --reload
```

---
<img width="1055" height="71" alt="Screenshot from 2025-10-23 20-37-24" src="https://github.com/user-attachments/assets/284746dc-3143-44bb-9cf0-6d731f208e2f" />

---
<img width="1054" height="53" alt="Screenshot from 2025-10-23 20-38-02" src="https://github.com/user-attachments/assets/a8e5c75c-75e7-4a51-b635-b472cd404375" />

---



Client Setup — Ubuntu (LDAP Client using SSSD)
1. Initial Setup and Packages

---

## Update the system
```bash
$ sudo apt update -y
```

---
<img width="1012" height="376" alt="Screenshot from 2025-10-23 21-05-05" src="https://github.com/user-attachments/assets/e2ed8878-1f9c-4284-9e40-9f83cb1fe0fc" />

---

## Set the hostname (use your actual FQDN)
```bash
$ sudo hostnamectl set-hostname ubuntu-client.lab.local
```

---
<img width="830" height="28" alt="Screenshot from 2025-10-23 21-06-54" src="https://github.com/user-attachments/assets/f4aa13c7-6113-48bf-8f76-ec584df0aa8d" />

---
<img width="1018" height="441" alt="Screenshot from 2025-10-23 21-10-21" src="https://github.com/user-attachments/assets/e0566bae-2408-464a-9a98-9984f5c94a76" />

---

## Install required client packages
```bash
$ sudo apt install freeipa-client sssd-ldap realmd oddjob oddjob-mkhomedir adcli -y
```

---
<img width="1019" height="165" alt="Screenshot from 2025-10-23 21-13-31" src="https://github.com/user-attachments/assets/9442498b-b95c-409d-8227-0d2b31aa129d" />



2. Joining Ubuntu to the FreeIPA Domain



---


## Join the domain and automatically configure home directory creation
```bash
$ sudo ipa-client-install --mkhomedir
```

---
<img width="1002" height="148" alt="Screenshot from 2025-10-23 21-42-33" src="https://github.com/user-attachments/assets/d9485f69-c203-40c4-a090-fd6c5117042f" />

---
<img width="976" height="138" alt="Screenshot from 2025-10-23 21-43-00" src="https://github.com/user-attachments/assets/6df74a38-490b-4d48-98e3-4e3cd27cb3b5" />

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
<img width="825" height="234" alt="Screenshot from 2025-10-23 21-45-09" src="https://github.com/user-attachments/assets/6dcaf399-8a9a-40bf-a19e-89b691b91f3e" />

---

---

## Create local mount point
```bash
$ sudo mkdir -p /home/ipausers
```

---
<img width="811" height="25" alt="Screenshot from 2025-10-23 21-45-43" src="https://github.com/user-attachments/assets/a0e2af31-30a5-4320-a965-24b548ef9f6e" />

---

## Mount the NFS share temporarily (replace with your server FQDN)
```bash
$ sudo mount ipa-server.lab.local:/home/ipausers /home/ipausers
```

---
<img width="894" height="31" alt="Screenshot from 2025-10-23 21-46-21" src="https://github.com/user-attachments/assets/608d5248-87f7-489b-80a3-3da681eed633" />

---

## Make the mount permanent in /etc/fstab
```bash
$ echo "ipa-server.lab.local:/home/ipausers /home/ipausers nfs defaults 0 0" | sudo tee -a /etc/fstab
```

---
<img width="1307" height="49" alt="Screenshot from 2025-10-23 21-47-26" src="https://github.com/user-attachments/assets/406d1172-71a4-471b-9c1e-80ed662d745f" />

---

4. Test Authentication
You should now be able to look up and log in as a FreeIPA user.


---


## Look up the admin user information
```bash
$ id admin
$ getent passwd admin
```

---
<img width="744" height="51" alt="Screenshot from 2025-10-23 22-44-06" src="https://github.com/user-attachments/assets/1b3ac22a-9389-44e7-9ca9-c0de061b4deb" />

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
<img width="744" height="67" alt="Screenshot from 2025-10-23 22-47-45" src="https://github.com/user-attachments/assets/dcd6f16c-09d5-4ddb-a991-91747bfd40da" />

---
<img width="744" height="50" alt="Screenshot from 2025-10-23 22-48-44" src="https://github.com/user-attachments/assets/d025127f-1667-42ce-85f6-43cf064c91fb" />

---

On Ubuntu (Client):
```bash
$ getent passwd admin  # Check the UID 
$ getent group admins  # Check the GID
```

---
<img width="790" height="49" alt="Screenshot from 2025-10-23 21-48-30" src="https://github.com/user-attachments/assets/8649decd-7089-4276-a424-6d90351b6fbf" />

---
<img width="746" height="52" alt="Screenshot from 2025-10-23 22-50-09" src="https://github.com/user-attachments/assets/a18a1dc1-d4c0-4356-acbb-060b0a4b00fa" />

---

The UID and GID values must be identical on both systems.

You’ve now configured:

A FreeIPA server on Rocky Linux for centralized authentication

Ubuntu client joined to the IPA domain using SSSD

NFS shared home directories for domain users

Consistent UID/GID mapping between systems

Author: Harison Kimutai Chirchir


Email: harisonchirchir2@gmail.com


