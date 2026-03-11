## Hardware

This project utilizes the following hardware components:

### 1. Raspberry Pi
A small, affordable single-board computer that is highly versatile. It serves as the main controller for the project, running the necessary software and handling communication between connected devices.

### 2. SD Card
A microSD card is used as the primary storage for the Raspberry Pi. It stores the operating system, project files, and application data required for smooth operation.

### 3. Shelly Power Plugs
Smart Wi-Fi-enabled plugs that allow remote control and automation of connected appliances. They can be integrated with the Raspberry Pi for monitoring and controlling power usage.

## Installation

### Prepare SD Card Image of Devuan Pi Operating System

- Download the Daedalus arm64 image zipfile for the Raspberry Pi 5 [nightly builds](https://arm-files.devuan.org/RaspberryPi%20Latest%20Builds/) on the Devuan ARM files site (the zipfile begins with `rpi-5-devuan-daedalus-` and ends with `.zip`).

- Transfer the image to the SD card using your prefered disk tool.

- Insert the SD card into the Devuan RPi5.

- Power up the Devuan RPi5, log in (devuan/devuan), and ensure the Raspberry Pi is connected to your network (either through the ethernet interface or you must configure the WiFi details for your local network with the command `menu-config`).

- Note down the IP address allocated to the Devuan RPi5.

- On the ansible provisioning host, ssh into the RPi5 via username and password (devuan/devuan) via ssh `devuan@IP_address_noted`, and issue `sudo su -` followed by `ssh-keygen`. Press return twice. Log out (control-D).

- On the ansible provisioning host, generate an SSH key pair (`ssh-keygen`) and add the public key to the RPi5 root SSH authorized_keys configuration file `/root/.ssh/authorized_keys`.

### Clone the Repository on the Ansible Provisioning Host

```sh
git clone https://github.com/dyne/lauds-iot-backend.git
cd lauds-iot-backend/ansible
```

### Provisioning via ansible

#### Remote ansible preparation

- On the ansible provisioning host a hostname and local IP address must be uniquely defined for the Devuan RPi5 in the `inventory.yml` file (eg, a host configuration example for `flirc-rpi5` is given as an example)

The unique hostname should be related to the LAUDS factory sitename or consortium member name and the IP address must correspond to the address noted in the previous step.

- Individual host VPN IP address configuration is configured in `ansible/host_vars/` with the configuration file matching the hostname (eg, the host configuration example file in the repo is `ansible/host_vars/flirc-pi5`):

```sh
client_ip_addr: 192.168.10.[address]
```
where [address] is a unique host address that will be allocated to your install.


- On the ansible provisioning host, create or add to the file `../.env` with the following content:
```sh
# LAUDS GATEWAY SERVER DETAILS
#
# export GATEWAY_SERVER_HOSTNAME="flirc-pi5"
#
## WIREGUARD VPN DETAILS (used in ansible)
#
export VPN_SERVER_IPV4="138.201.89.108"
export VPN_SERVER_IPV6="2a01:4f8:c17:3dd5::1"
export VPN_SERVER_HOSTNAME="zorro.free2air.net"
export VPN_SERVER_PORT="51194"
# TODO: Publish public key & lookup in DNS when BIND9.18.33 SIG0 update bugfix is published
export VPN_SERVER_PUBLIC_KEY="IstwnIfVuvgfb7LzaE3YLb24FAT2oUEhVcsZILDhHXk="
export VPN_CLIENT_IPV4_NETMASK="/24"
```

#### Remote ansible provisioning

On the ansible provisioning host:

```sh
source ../.env && ansible-playbook -i inventory.yml playbook.yml
```

### Configure local credentials

Create file `.env` to set default credentials

```sh
# NodeRed admin user password
# export NR_ADMIN_PW="<your NR admin user login password>"
export NR_ADMIN_PW="laudsgateway"

# Jupyter Notebook token
# export JUPYTER_TOKEN="<your Jupyter Notebook login token>"
export JUPYTER_TOKEN="laudsgateway"
```


