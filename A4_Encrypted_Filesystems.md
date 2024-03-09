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




