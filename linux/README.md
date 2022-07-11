# Things to do after installing a debian linux server

In this page, we will go over things to install, configure the debian server to harden the security and get ready for general use. Assuming all the commands are run from the user root, else add the `sudo` keyword as prefix to all commands below.

## _Update to latest patches_
```sh
apt update
apt dist-upgrade -y
apt clean
apt autoremove
reboot
```

## _SSH - Reconfigure ssh server_
```sh
cd /etc/ssh/
rm ssh_host_*
dpkg-reconfigure openssh-server
```

## _SSH Public Key Pair_
```sh
ssh-keygen -b 4096
```

Copy the public key to remote server
```sh
ssh-copy-id user@ip-address
```
