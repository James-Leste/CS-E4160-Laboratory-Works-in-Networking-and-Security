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


