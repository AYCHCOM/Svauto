
# SVAuto - The Save Automation

SVAuto is an Open Source Automation Tool, it glues together a different set of tools for building immutable servers images (QCoWs, OVAs, VHDs) and for Infrastructure Automation (Servers or Desktops).

With SVAuto, you can create QCoWs, VMDKs, OVAs, and etc, with Packer and Ansible! It uses the official ISO Images as a base.

There is also a preliminary support for Vagrant with Ansible, that uses regular Ubuntu / CentOS boxes hosted on Atlassian.

Looking forward to add support for Linux Containers (systemd-nspawn, LXD and Docker).

SVAuto was designed for Ubuntu Bionic 18.04 (latest LTS), Server or Desktop. But CentOS is also supported.

It uses the following Open Source projects:

* Ubuntu Bionic 18.04
* LXD 3.0
* QEmu 2.11
* systemd 237
* Ansible 2.5
* Packer 1.0.4
* VirtualBox 5.2

On SVAuto's radar:

* Vagrant 2.0.2
* Docker 18.06
* Amazon EC2 AMI & API Tools

It contains Ansible Playbooks for Automated deployments of:

* Ubuntu
* CentOS
* OpenStack on Ubuntu LTS

# Quick Procedure

## 1- Install Ubuntu 18.04 (Server or Desktop), let's say with:

- Hostname: "myserver-1"
- User: "ubuntu"
- Password: "whatever"
    
## 2- Hostname and Hosts Basic Configuration

One line in `/etc/hostname` with your host's name (lol):

    myserver-1
    
First two lines of `/etc/hosts` (leave the rest of your file untouched):

    127.0.0.1 localhost.localdomain localhost
    127.0.1.1 myserver-1.yourdomain.com myserver-1 myserver
    
*NOTE: If you have fixed IP (v4 or v6), you can use it here (recommended), instead of 127.0.1.1. But the default works very well, especially if you're connected via DHCP / WIFI.*

Make sure it is working:

    hostname        # Must returns ONLY your Hostname, nothing more.
    hostname -d     # Must returns ONLY your Domain.
    hostname -f     # Must returns your FQDN.
    hostname -i     # Must returns 127.0.1.1 (or your local IP, if you configured it previsouly).
    hostname -a     # Must returns your aliases.

## 3- Install Git & Python Minimal on Ubuntu 18.04

    sudo apt install -y git python-minimal

## 4- Downloading SVAuto

To download SVAuto into your home directory, clone it with git, as follows:

    cd ~
    git clone https://github.com/tmartinx/svauto

## 5- Bootstrapping local systems with the "Preset Scripts"

Bootstrap Ubuntu 18.04 Desktop, it upgrades and installs many useful applications, like Google Chrome and etc...

    cd ~/svauto
    ./scripts/preset-ubuntu-desktop.sh

Bootstrap Ubuntu 18.04 Server, it upgrades and install a few applications for Servers.

    cd ~/svauto
    ./scripts/preset-ubuntu-server.sh

*NOTE: You can edit the Preset Scripts above and add `--dry-run` to the line that starts with `./svauto.sh ...`, this way, it doesn't automatically runs Ansible, it only outputs Ansible's Inventory and Playbook files. Then, you can run Ansible manually, like: `pushd ~/svauto/ansible ; ansible-playbook -i ansible-hosts-XXXX ansible-playbook-XXXX.yml` later, if you want.*

## 6- Creating O.S. Images: QCoW, OVAs, VHD, etc 

### 6.1- Bootstrapping your Ubuntu (Desktop or Server) for SVAuto

To build QCoWs/OVA images with Packer and Ansible, you need to "Bootstrap Ubuntu For SVAuto", so you can take advantage of all SVAuto's features to build O.S. images.

NOTE: The following procedure download a few ISO images from the Internet, and store it under Libvirt's directory.

Ubuntu Desktop

    cd ~/svauto
    ./scripts/preset-svauto-desktop.sh

Ubuntu Server

    cd ~/svauto
    ./scripts/preset-svauto-server.sh

After this, you'll be able to use Packer to build O.S. Images with Ansible!

### 6.2- Packer, Baby Steps

To make sure that your Packer installation is good and that you can actually run it and have a RAW Image in the end of the process, lets go baby steps first.

SVAuto comes with bare-minimum Packer Templates, also very minimal Kickstart and Preseed files.

*NOTE: Packer requires root because it runs the KVM binary directly (i.e., not via Libvirt), so, even if you're member of "libvirt" group, you still need to run it as root.*

Building an Ubuntu 18.04 or CentOS 7 RAW Images with just Packer:

    cd ~/svauto
    sudo packer build packer/ubuntu18.yaml

or:

    sudo packer build packer/centos7.yaml

Enable KVM SDL Screen to see what Packer is doing!

Allow local root user to use X Window System:

    xhost local:root

    sudo packer build packer/ubuntu18-gui.yaml

or:

    sudo packer build packer/centos7-gui.yaml

NOTE: The resulting images are created under `packer/ubuntu18-tmpl`, or `packer/centos7-tmpl`, or ...

### 6.3- Packer and Ansible

Those small Packer Templates tested on previous baby steps, are the base for the rest of "SVAuto Image Factory".

For example: `packer/ubuntu18.yaml` is the base for `packer/ubuntu18-template.yaml`, where the `*-template.yaml` is the one actually being used by `svauto.sh`.

SVAuto basically glues together Packer and Ansible, under temporaries subdirectories (`packer/build-something`), it then goes there and runs `packer build` for you.

Building an Ubuntu 18.04 QCoW (compressed) with Packer and Ansible:

    cd ~/svauto
    ./packer-build/ubuntu18.sh

Building a CentOS 7 QCoW (compressed) with Packer and Ansible:

    cd ~/svauto
    ./packer-build/centos7.sh

*NOTES:*

*Those O.S. Images have Cloud Init support and its file system grows automatically in a Cloud Environment!*

## 7- SVAuto Local Config File

The local config file is `~/.svauto.conf`, it overrides part of `~/svauto/svauto.conf` variables.

If might have the following contents (example):

    SVAUTO_DIR=/home/ubuntu/svauto
    DOCUMENT_ROOT="public_dir"
    DNS_DOMAIN="yourdomain.com"
    SVAUTO_YUM_HOST="ftp.$DNS_DOMAIN"

## 8- Cleaning it up

To remove the temporary files, run:

    cd ~/svauto
    ./svauto.sh --clean-all

## 9- Deploy OpenStack using SVAuto

SVAuto supports deploying OpenStack in a very simple way, instructions at the following link:

[OpenStack SVAuto](README.OpenStack.md)
