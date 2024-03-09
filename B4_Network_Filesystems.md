# B4 Network Filesystems

## 1. Preparation

> You'll need two virtual machines for this exercise. Add aliases (lab1 and lab2) for the addresses to /etc/hosts.
>
> Create two new users (e.g. "testuser1" and "testuser2") with adduser to both the computers. Ensure that users have the same UID on both computers (eg. testuser1 UID is 1001 on lab1 and lab2, testuser2 is 1002). The easiest way is to create both users in the same order onboth computers. Make sure you have a home directory for testuser1 on lab1.

Create User

```shell
sudo useradd -m -u 10001 testuser1
sudo useradd -m -u 10002 testuser2
```

`-m`: create a home directory
`-u [uid]`: set a UID

## 2. Configuring and testing NFS

> NFS is an acronym for "network filesystem". NFS is implemented for nearly all unix variations and even for windows.
>
> Make sure you have `nfs-kernel-server` installed on lab1. Export `/home` directory via `/etc/exports`. Restart the NFS server daemon. Mount `lab1:/home` to `lab2:/mnt`. You can change user with su, e.g. `su testuser1`. Test that NFS works by writing a file in `lab1:/home/testuser1/test.txt` and open the same file at `lab2:/mnt/testuser1/test.txt`.

Install NFS Server on lab1

```shell
sudo apt install nfs-kernel-server -y
```

Open and edit `etc/exports`

```shell
/home   lab2(rw,sync,no_subtree_check)
```

Install NFS client on lab2

```shell
sudo apt install nfs-common
sudo mount lab1:/home /mnt
```

Check mount status

```shell
mount -l | grep nfs
```

Create file on lab1 and check on lab2

lab1(switch to testuser1's home directory, /home/testuser1)

```shell
echo "this is a new line" > test.txt
```

lab2 (switch to testuser2's mnt directory, /mnt/testuser1)

```shell
cat test.txt
```

To see if the test.txt has the exact same content as lab1

## 3. Configuring and testing samba

On lab1 install `samba`

```shell
sudo apt install samba
```

Configure /etc/samba/smb.conf, add following configuration at the bottom.

```text
[home]
    path=/home
    browseable = yes
    read only = no
    writable = yes
    valid users = testuser1
```

Save the file, restart the service and turn off firewall

```shell
sudo service smbd restart
sudo service smbd status
sudo ufw allow samba
```

add a user to samba and set a password

```shell
sudo smbpasswd -a testuser1
```

On lab2 install smbclient

```shell
sudo apt install samba-client
```

check if you can access to the lab1 shared folder using samba client

```shell
smbclient //lab1/home -U testuser1
```

If the test is successful, mount the target folder, and check mnt folder.

```shell
sudo mount -t cifs -o username=testuser1 //lab1/home/testuser1 /mnt
```

Check mounting status on lab2

```shell
mount -l | grep cifs
```

## 4. Configuring and testing sshfs

> sshfs is filesystem for FUSE (filesystem in userspace).
>
> Start by unmounting the samba share on lab2.
>
> Next mount lab1:/home/testuser1 to lab2:/home/testuser1/mnt using sshfs. Demonstrate this to the assistant.

Unmount the mounted folder

```shell
sudo umount /mnt
```

Install sshfs on lab2

```shell
sudo apt install sshfs
```

generate rsa keys on lab2, and add the public key to [`testuser1@lab1:/home/testuser1/.ssh/authorized_keys`]. Test if from testuser1@lab2 can ssh to testuser1@lab1

If not, open `/etc/ssh/sshd_config` to uncomment `PubkeyAuthentication yes`

If yes, continue to mount `testuser1@lab1:/home/testuser1` to `testuser1@lab2:/home/testuser1/mnt`. From `testuser1@lab2:/home/testuser1`:

```shell
sshfs testuser1@lab1:/home/testuser1 mnt
```

Check mount status

```shell
mount -l | grep sshfs
```

I get

```text
$ mount -l |grep sshfs
testuser1@lab1:/home/testuser1 on /home/testuser1/mnt type fuse.sshfs (rw,nosuid,nodev,relatime,user_id=10001,group_id=10001)
```

## 5. Configuring and testing WebDAV

> WebDAV (Web-based Distributed Authoring and Versioning) is a set of extensions to the HTTP protocol which allows users to collaboratively edit and manage files on remote web servers.
>
> In this exercise we'll use the built-in WebDAV module of Apache2 server platform. Check that apache2 is installed and enable the dav_fs module. Restart apache2 every time after enabling a module.
>
> Create a directory /var/www/WebDAV for storing WebDAV related files and add subdirectory files to be shared using WebDAV. Change the owner of the directories created to www-data (Apache's user ID) and the group to your user ID. Change the permissions if needed.
>
> Create an alias to the virtual host file (/etc/apache2/sites-available/000-default.conf) so that /var/www/WebDAV/files can be reached through http://localhost/webdav. Enable the virtual host by creating a symbolic link between /etc/apache2/sites-available/000-default.conf and /etc/apache2/sites-enabled/.
>
> Restart apache2 and check that you can reach the server with, for example, elinks(1).
>
> Set up Authorization
>
> Enable the auth_digest module for apache. Create a password file for a testuser with htdigest(1) to /var/www/WebDAV. Edit permissions of the file so that only www-data and root can access it. Use the following template to add the location to the virtual host file:
>
> ```text
> <Location /webdav>
>  DAV On
>  AuthType Digest
>  AuthName "your_auth_name"
>  AuthUserFile path_to_your_password_file
>  Require valid-user
> </Location>
> ```
>
> Restart Apache2 and test the server from another machine using cadaver(1). You should reach the server http://lab1/webdav.


Install Apache2 on lab1

```shell
sudo apt install apache2
```

Enable all the mods that will be used and restart the service

```shell
sudo a2enmod dav
sudo a2enmod dav_fs
sudo a2enmod auth_digest
sudo systemctl restart apache2
sudo systemctl status apache2
```

create a folder `/var/www/WebDAV` and add alias to `/etc/apache2/sites-available/000-default.conf`

```text
// Outside <VirtualHost></VirtualHost>

Alias /webdav /var/www/WebDAV
<Location /webdav>
  DAV On
  AuthType Digest
  AuthName "webdav"
  AuthUserFile /etc/apache2/webdav.password
  Require valid-user
</Location>
```

genereate password

```shell
sudo htdigest -c /var/www/WebDAV/webdav.password "webdav" testuser
```

User cadaver on lab2 to test the webdav setup

```shell
cadaver http://lab1/webdav
```

## 6. Raid 5

Check available disks using `lsblk`, result

```text
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0  40.4M  1 loop /snap/snapd/20671
loop1    7:1    0    87M  1 loop /snap/lxd/27428
loop2    7:2    0  63.9M  1 loop /snap/core20/2182
loop3    7:3    0 111.9M  1 loop /snap/lxd/24322
loop4    7:4    0  63.9M  1 loop /snap/core20/2105
sda      8:0    0    40G  0 disk
└─sda1   8:1    0    40G  0 part /
sdb      8:16   0    10M  0 disk
sdc      8:32   0     1G  0 disk
sdd      8:48   0     1G  0 disk
sde      8:64   0     1G  0 disk
```

Add raid 5 array

```shell
sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdc /dev/sdd /dev/sde
```

Format the /dev/md0 with EXT4 filesystem

```shell
sudo mkfs.ext4 /dev/md0
```

mount the filesystem to a directory

```shell
sudo mount /dev/md0 /mnt/md0
```

verify the mounting

```shell
df -h
```

add to `/etc/exports` to export the dir and `restart nfs-kernel-server`

```text
/mnt/md0 *(rw,sync,no_subtree_check)
```

On lab2, mount //lab1/mnt/md0 to a directory

```shell
sudo mount -t nfs lab1:/mnt/md0 mnt
```
