# A4 Encrypted Filesystems

## 1. Preparation

install necessary tools

```shell
    sudo apt update
    sudo apt -y install cryptsetup
    sudo apt -y install gnupg
    sudo apt -y install haveged
    sudo apt install -y gocryptfs
```

Enable dm_crypt and aes, disable cryptloop.

```shell
sudo modprobe dm_crypt
sudo modprobe aes
sudo modprobe -r cryptoloop
```

check if mods are correctly set up

```shell
lsmod | grep [mod name]
```

## 2. Encrypting a single file using GnuPG

> Let's start with a simple encryption using GPG. Begin by creating a GPG keypair on both lab1 and lab2 using the RSA algorithm and 2048 bit keys. Exchange (and verify) the public keys between lab1 and lab2.
>
> Create a plaintext file with some text in it on lab1. Encrypt the file using lab2's public key, and send the encrypted file to lab2. Now decrypt the file.
> 
> Finally, sign a plaintext file on lab2, send the file with its signature to lab1. Verify on lab1 that it really was the lab2 user that signed the message.

Full process of encrypt a file:

- lab1 uses lab2's public key to encrypt a file and send the encrypted file to lab2.
- lab2, receiving lab1's encrypted file, uses its own private key to decrypt the file

Generate keys for both lab1 and lab2, and user interactive mode to configure the key (name, email, comment)

```shell
gpg --full-generate-key
```

output public keyfile, `--armor` will output ascii-armored content.

```shell
gpg --armor --output lab1key.gpg --export vargrant@lab2
```

Then a `lab1key.gpg` and a `lab2key.gpg` will appear on lab1 and lab2 respectfully, copy the files to the other machine.

Sign the keys received from the other machine

```shell
gpg --edit-key vagrant@lab1
```

After signing the key, use the public key of the recipient to encrypt a file, output into test.gpg

```shell
gpg --output test.gpg --encrypt --recipient vagrant@lab2 test.txt
```

Upon receiving the file, use private key to decrypt the file.

```shell
gpg --output result.txt --decrypt test.gpg
```

Full process of sign a file

- On lab2, use a secret key to sign a file (signed.txt). A signed file will be generated (signed.txt.gpg).
- One can use verify the key using `--verify` option to output the signature info from the signed file

sign a file, use `--default-key` to set the default key to use. a `signed.txt.gpg` will be generated.

```shell
gpg --sign --default-key vagrant@lab2 signed.txt
```

Verify a file

```shell
gpg --verify signed.txt.gpg
```

Verification output

```shell
gpg: Signature made Mon Mar 11 12:23:53 2024 UTC
gpg:                using RSA key A20A8FA5028DD139E73F243919BD9526874AC8DC
gpg:                issuer "vagrant@lab2"
gpg: Good signature from "lab2key <vagrant@lab2>" [ultimate]
```

Decryption

```shell
gpg --output outsigned.txt --decrypt signed.txt.gpg
```

## 3. Crypto filesystem with loopback and device mapper

> Create a loopback device for the file using losetup(8). Then using cryptsetup(8), format the loopback device and map it to a pseudo-device. Please use LUKS with aes-cbc-essiv:sha256 cipher (should be default).
>
> Create an ext2 filesystem on the pseudo-device, created in the previous step. The filesystem can be created with mkfs.ext2(8).
>
> After this, you have successfully created an encrypted filesystem into a file. The filesystem is ready, and requires a passphrase to be entered when you luksOpen it.
>
> Now mount your filesystem. Create some files and directories on the encrypted filesystem. Check also what happens if you try to mount the system with a wrong key.


Create a file with random data

```shell
dd if=/dev/urandom of=loop.img bs=1k count=32k
```

Find the first unused loopback device

```shell
losetup -f
```

Upon find the unused loopback device (/dev/loop5) in this case, associate loop.img with the loopback device.

```shell
sudo losetup /dev/loop5 loop.img
```

Format the loopback device with LUKS

```shell
sudo cryptsetup luksFormat /dev/loop5
```

Open the encrypted device with `cryptsetup` to map it to a pseudo-device. The device will be available as `/dev/mapper/sercrets`

```shell
sudo cryptsetup luksOpen /dev/loop5 secrets
```

Ceate an ext2 filesystem on the pseudo-device using `mkfs.ext2`.

```shell
mkfs.ext2 /dev/mapper/secrets
```

Mount the newly created ext2 filesystem to a mount point. Then the file system will be available at the mount point.

```shell
sudo mount /dev/mapper/secrets ~/mp
```

Unmount the filesystem and close the device

```shell
sudo umount ~/mp
cryptsetup close secrets
losetup -d /dev/loop5
```

Bad passphrase:

![alt text](image.png)


## 4. Gocryptfs

> Using gocryptfs, mount an encrypted filesystem on a directory of your choice. This gives you the encryption layer. After this, create a few directories, and some files in them. Unmount gocryptfs using Fuse's fusermount.
>
> Check what was written on the file system.

Create a directory and init a gocryptfs filesystem

```shell
gocryptfs -init ~/goencrypt
```

Create a directory to mount the encrypted directory

```shell
gocryptfs ~/goencrypt ~/mp/gocryptfs
```

Unmount the directory

```shell
fusermount -u ~/mp/gocryptfs
```

## 5. TrueCrypt and alternatives

> On this course we used to have a TrueCrypt assignment where students were required to create a hidden volume inside another volume. However, since 2014 there has been a lot of discussion about the security of TrueCrypt. Read arguments against and for TrueCrypt and based on your knowledge of the subject make a choice to use either TrueCrypt or one of the alternative forks that can create hidden volumes. Using the software of your choice create a hidden volume within an encrypted volume.
>
> If you decide to use veracrypt, the command line syntax for veracrypt is
>
> ```shell
> veracrypt [OPTIONS] VOLUME_PATH [MOUNT_DIRECTORY]
> ```
>
> and the options can be found by running
>
> ```shell
> veracrypt -h
> ```

