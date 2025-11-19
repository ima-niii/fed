#NFS_and_Samba_Configuration.md
Network File Sharing Setup on Fedora using Samba and NFS

This document explains how to configure Samba and NFS on a Fedora Server for file sharing within a local network.
Both methods allow clients to access shared directories, but they work differently. Samba is commonly used with Windows systems, while NFS is often used with Linux systems.

------------------------------------------
1. SAMBA CONFIGURATION
------------------------------------------
1.1 Install Samba Packages

Ensure Samba is installed on the Fedora server:

sudo dnf install samba samba-common samba-client

1.2 Create a Samba User

Samba requires authentication. Create a Samba user that corresponds to an existing Linux user:

sudo smbpasswd -a your_username


You will be prompted to create a password for the Samba user.

1.3 Create a Directory to Share
sudo mkdir -p /srv/samba/shared
sudo chmod -R 755 /srv/samba/shared

1.4 Backup Samba Configuration File
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak

1.5 Edit Samba Configuration File

Open the file:

sudo nano /etc/samba/smb.conf

Add the following at the bottom:
[shared]
    comment = Samba Shared Folder
    path = /srv/samba/shared
    browseable = yes
    read only = no
    valid users = your_username


path → directory you created

valid users → Samba user allowed to access it

Save and exit.

1.6 Restart Samba Services
sudo systemctl restart smb
sudo systemctl enable smb

1.7 Test Samba Configuration
testparm


If no errors are shown, Samba is configured correctly.

1.8 Access Samba Share from Client
Linux client
smbclient //server_ip/shared -U your_username

Windows client

Open File Explorer and enter:

\\server_ip\shared

------------------------------------------
2. NFS CONFIGURATION
------------------------------------------
2.1 Install NFS Utilities
sudo dnf install nfs-utils

2.2 Create NFS Shared Directory
sudo mkdir -p /srv/nfs/shared
sudo chown -R nobody:nobody /srv/nfs/shared
sudo chmod -R 777 /srv/nfs/shared   # temporary full access

2.3 Configure NFS Exports

Open the exports file:

sudo nano /etc/exports


Add:

/srv/nfs/shared 192.168.193.0/24(rw,sync,no_root_squash)


Replace the subnet with your network: 192.168.193.0/24

Save and exit.

2.4 Apply Export Rules
sudo exportfs -r


Check:

sudo exportfs -v

2.5 Start and Enable NFS Server
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
sudo systemctl status nfs-server

2.6 Allow NFS Through Firewall
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --reload

2.7 Accessing the NFS Share from a Client
On the CLIENT machine:
Create mount point:
sudo mkdir -p /mnt/nfs_shared

Mount the NFS share:
sudo mount -t nfs 192.168.193.128:/srv/nfs/shared /mnt/nfs_shared


Note: Replace 192.168.193.128 with your Fedora server’s IP address.

If access is denied, the issue is likely permissions, SELinux, or UID/GID mismatch.

2.8 (Optional) Auto-mount at Boot

Add to the client’s /etc/fstab:

192.168.193.128:/srv/nfs/shared  /mnt/nfs_shared  nfs  defaults  0 0

------------------------------
