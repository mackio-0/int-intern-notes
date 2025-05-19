# Week 2 Day 3

## Proxmox

[Learn Linux Channel Proxmox Playlist](https://www.youtube.com/playlist?list=PLT98CRl2KxKHnlbYhtABg6cF50bYa8Ulo)

**NOTE** - If you don't enable Virtualize intel..... option in cpu tab of VMware while installing Proxmox, VM will not be able to boot.

### Containers vs VMs on Proxmox

* Containers go down when migrating
* VMs do not go down when migrating
* Containers are faster and lighter than VMs
* VMs are more resource intensive than containers.

### Lunching VMs on Proxmox

It's a good practice to have management network and VM network separated.

**ERROR** : `Unable to get an ip address from dhcp server on eth0(or)ens18` - run `ip link set eth0(or)ens18 up`

`Discard` in Create Virtual Machine -> Hard Disk is the option to trim unused space in the disk, **ONLY TURN ON WHEN SERVER HAS SSD.**

#### Setting up static IP address for VM

* [Server World Reference](https://www.server-world.info/en/note?os=Ubuntu_24.04&p=initial_conf&f=3)

* Go to `/etc/netplan/50-cloud-init.yaml` and add the following:

    ```yaml
    network:
    ethernets:
      ens18:
        dhcp4: false
        addresses: [192.168.2.200/24]
        routes:
          - to: default
            via:  192.168.2.2 # gateway, if you don't know the gateway check with `ip route` in the host machine(Proxmox).
            metric: 100
        nameservers:
          addresses: [8.8.8.8]
    version: 2
    ```

* **gateway**, if you don't know the gateway, check with `ip route` in the host machine(Proxmox).
* `sudo netplan try` for testing the config file and `sudo netplan apply` for applying the config file.

#### The problems

* Even tho we now have an static ip address for the VM, but we can't ssh into VM.
* VM also can't access network outside too, `ping 8.8.8.8` doesn't work.
* But it can ping the host machine(Proxmox) and vice versa.

#### The Solutions

* **While running proxmox on VMware, VMware need to be running with sudo privileges, run `sudo vmware` and select the proxmox VMX file.**

#### Install qemu-guest-agent on VM

* `sudo apt install qemu-guest-agent`
* qemu-guest-agent is a helper daemon that provides information about the guest OS to the host OS.
* `sudo systemctl enable qemu-guest-agent` - there will be error starting up the service we just installed. It's ok. We have to start qemu-guest-agent in the gui offerred by Proxmox.
* `VM> Options> QEMU Guest Agent> Enable`

## MISC

* `cat /proc/cpuinfo` - see details about the cpu
* `hostnamectl` - see details about the system
* `sudo timedatectl set-timezone <timezone_name>` - to change timezone
* `sudo ssh-keygen -A` - add ssh host keys to the system
