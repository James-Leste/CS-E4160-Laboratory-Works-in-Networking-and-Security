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

### 2.1 Demostrate a working configuration

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

### 2.2 Is it possible to encrypt all NFS traffic? How?

- On the server we can decide that we don't want to trust any requests made as root on the client. We can do that by using the root_squash option in /etc/exports:

```shell
/home slave1(rw,root_squash)
```

- By configuring `/etc/hosts.deny` to deny all by default, it reduces the attack surface of the server. This means fewer entry points are available for potential attackers, making your system more secure.

    Editing `/etc/hosts.allow` to specify which machines can access the portmapper and related services (like `nfsd`, `mountd`, `ypbind/ypserv`, etc.) allows for precise control over who can communicate with these services. This limits access to trusted machines only, reducing the risk of unauthorized access.
    Do not put anything but IP NUMBERS in the portmap lines of these files. Host name lookups can indirectly cause portmap activity which will trigger host name lookups which can indirectly cause portmap activity which will trigger...

- SUID (Set owner User ID up on execution) is a special type of file permissions given to a file. Normally in Linux/Unix when a program runs, it inherits access permissions from the logged in user. SUID is defined as giving temporary permissions to a user to run a program/file with the permissions of the file owner rather that the user who runs it. In simple words users will get file owner's permissions as well as owner UID and GID when executing a file/program/command.
  On the client we can decide that we don't want to trust the server too much a couple of ways with options to mount. For example we can forbid suid programs to work off the NFS file system with the nosuid option.
  Using the nosuid option is a good idea and you should consider using this with all NFS mounted disks. It means that the server's root user cannot make a suid-root program on the file system, log in to the client as a normal user and then use the suid-root program to become root on the client too.

## 3. Configuring and testing samba

> Samba is unix/linux implementation for normal Windows network shares(netbios and CIFS (common internet filesystem)). You can configure samba via /etc/samba/smb.conf. You can access files shared with samba with command smbclient or by mounting the filesystem via mount, like with NFS. Mounting will require cifs-utils to be installed on lab2.
>
> Start by unmounting with umount(8) the NFS directory in lab2 from the previous assignment. If unmounting complains "resource busy", you have a shell with your current directory in the /mnt directory and you need to switch to another directory.
>
> Make sure you have samba installed on lab1. Share /home with read and write permissions (/home shares are already at smb.conf but it needs a little bit of tweaking) and reload samba. Run smbpasswd on lab1 and add testuser1 as a user. Try to mount //lab1/home/testuser1 to lab2:/mnt with username testuser1 and testuser1's password. If it doesn’t work, check that necessary services and ports are open.

### 3.1 Demonstrate a working configuration

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

### Only root can use mount. What problem does this pose for users trying to access their remote home directories? Is there a workaround for the problem?

Problems:

This limitation means that regular users without root privileges are unable to mount their home directories stored on a remote NFS server, which can be inconvenient in environments like universities, research institutions, or any scenario where users need to access their files from various workstations.

Work around:

- Automounting: The automounter (autofs) is a service that automatically mounts filesystems when they are accessed and unmounts them after a period of inactivity. This service can be configured by the system administrator to mount user home directories on demand, without requiring user intervention or root privileges. The configuration involves setting up autofs on client machines to mount home directories from the NFS server automatically when the directories are accessed.

- Sudoers Configuration: System administrators can configure the /etc/sudoers file to allow specific users or groups to execute the mount and umount commands without needing to enter a password. Care must be taken to restrict these privileges to specific mounts to avoid security risks.

- User-space Filesystem Tools: Tools like sshfs allow mounting filesystems in user space, which can be used as an alternative to traditional NFS mounts. While sshfs is based on SSH and not NFS, it can be used in similar scenarios to provide users access to their remote files without needing root privileges.

## 4. Configuring and testing sshfs

> sshfs is filesystem for FUSE (filesystem in userspace).
>
> Start by unmounting the samba share on lab2.
>
> Next mount lab1:/home/testuser1 to lab2:/home/testuser1/mnt using sshfs. Demonstrate this to the assistant.

### 4.1 Provide the commands that you used

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

### 4.2 When is sshfs a good solution?

**User-space operation:** SSHFS is really simple to build, install, and use. Because it’s in user space, any user can mount a remote filesystem using only SSH (port 22). Therefore you only need to have one open port on the client, port 22. Moreover, because users can take advantage of it whenever needed, it does not involve admin intervention. It doesn't require complex setup or root privileges, making it easy for users to set up their own remote mounts as needed.

**Increased CPU burden:** This imposes a CPU load on the client and server. A quick comparison of CPU load on the server and client for all three workloads for both NFS and SSHFS revealed that the load on a single core was much greater for SSHFS than for NFS. The increased workload on the client during sequential read tests was a little higher than I expected.

**Cross platform and compatiability:** SSH is a protocol and not dependent on the OS. SSHFS is available on various UNIX-like operating systems, including Linux and macOS, and there are solutions available for Windows as well. This makes it a flexible choice in mixed-OS environments.

### 4.3 What are the advantages of FUSE(file system in user space)?

- Crashes in user space programs do not crash the system like kernel filesystem would
- User space programs are easier to debug (debuggers, syscall tracers, etc)
- User space code can be written in any language, is easy to debug, and you can use a rich set of libraries to make your job easier. So handling strings, network connections, encryption, compression, dedupe etc will use less lines of code with FUSE than a kernel driver.
- Writing kernel code is much more complicated and mistakes can cause reboots.
FUSE is portable, kernel drivers are not portable between different operating systems.

### 4.4 Why doesn't everyone use encrypted channels for all network filesystems?

- Encryption and decryption processes add computational overhead.
- Many existing network filesystems and infrastructures were designed and implemented before the widespread adoption of encryption. Updating these systems to support encrypted channels can be challenging
- Some organizations might choose to implement encryption at different layers of their infrastructure.
- Within controlled and secure environments, such as internal networks that are behind firewalls and other security measures, the risk of data interception might be deemed low enough that the added burden of encryption is not justified.

## 5. Configuring and testing WebDAV

> WebDAV (Web-based Distributed Authoring and Versioning) is a set of extensions to the HTTP protocol which allows users to collaboratively edit and manage files on remote web servers.
>
> In this exercise we'll use the built-in WebDAV module of Apache2 server platform. Check that apache2 is installed and enable the dav_fs module. Restart apache2 every time after enabling a module.
>
> Create a directory /var/www/WebDAV for storing WebDAV related files and add subdirectory files to be shared using WebDAV. Change the owner of the directories created to www-data (Apache's user ID) and the group to your user ID. Change the permissions if needed.
>
> Create an alias to the virtual host file (/etc/apache2/sites-available/000-default.conf) so that /var/www/WebDAV/files can be reached through <http://localhost/webdav>. Enable the virtual host by creating a symbolic link between /etc/apache2/sites-available/000-default.conf and /etc/apache2/sites-enabled/.
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
> Restart Apache2 and test the server from another machine using cadaver(1). You should reach the server <http://lab1/webdav>.

### 5.1 Demonstrate a working setup. (View for example a web page on one machine and edit it from another using cadaver)

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

### 5.2 Demonstrate mounting a WebDAV resource into the local filesystem

mount

```shell
sudo mount -t davfs http://lab1/webdav /mnt/webdav
```

## 6. Raid 5

> In this task, you are going to establish a Network Attached Storage (NAS) system with lab1 as a server. The server should use Raid for data integrity. Set up Raid 5 on the NAT server and create EXT4 filesystem on the array.
>
> You need at least three partitions to do this, you can either partition current storage or add more virtual storage to your virtual machine. Then use mdadm tool to create the raid 5. Share the NAS device you setup with NFS.

### 6.1 What is raid?  What is parity? Explain raid5?

RAID (Redundant Array of Independent Disks) is a data storage virtualization technology that combines multiple physical disk drives into one or more logical units to improve performance, increase storage capacity, and provide redundancy in case of disk failure. RAID levels describe the balance and trade-off between performance, redundancy, and capacity.

Parity is a method used in RAID configurations to provide fault tolerance. It involves storing extra data (parity information) calculated from the actual data across the array's disks. If one disk fails, this parity information, along with the remaining data, can be used to reconstruct the lost data. Parity allows a RAID array to continue operating in a degraded mode after a single disk failure and fully recover the lost data once the failed disk is replaced.

RAID 5 is a popular RAID level that offers a good balance between storage efficiency, fault tolerance, and performance. It requires at least three disks and uses block-level striping with parity data distributed across all disks in the array.

### 6.2 Show that your raid5 solution is working

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

### 6.3 Access the NAS device from lab2 over NFS

On lab2, mount //lab1/mnt/md0 to a directory

```shell
sudo mount -t nfs lab1:/mnt/md0 mnt
```
