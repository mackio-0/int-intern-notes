# Week 2 Day 4

## Proxmox

### Creating Virtual Machine Template

A couple of things before converting the VM to a template:

* The VM itself must be **properly configured** with `cloud-init` and `qemu-guest-agent` installed. So the clone VMs running from that template will also have them. `cloud-init` allow you to automate a series of tasks needed to perform everytime you set up a linux server. (like resetting ssh host keys, etc.)

* The VM must be **stopped** before converting it to a template.

* We need to change some things that will remain the same, but which we don't want it to be the same.

* We need to clean up the VM's ssh host keys before converting it to a template, if not the template will have that shh host keys and the clones spun up from that template. If so, when we ssh into the clone VMs the ssh agent will be very confused and will not be able to ssh into the VMs. 

    ```bash
    cd /etc/ssh
    sudo rm -r ssh_host_*
    ```

    `cloud-init` will re-generate the ssh host keys when the VM is started.

* We need to delete the `machine-id` in ubuntu and some other OS that has it. If not, every VM will have the same machine-id and it will be confusing But we can't just delete the file, it won't work. We need to empty the file completely.

    ```bash
    sudo truncate -s 0 /etc/machine-id
    ```

    There is a symbolic link to the `machine-id` file in `/var/lib/dbus`. We need to make sure it is symbolic link.

    ```bash
    ls -l /var/lib/dbus/machine-id # check if it is a symbolic link if not make it one
    sudo ln -s /etc/machine-id /var/lib/dbus/machine-id
    ```

* Clean the apt cache.

    ```bash
    sudo apt clean
    sudo apt autoremove
    sudo apt purge
    ```

* Clean the cloud-init cache.

    ```bash
    sudo cloud-init clean
    ```

Finally, we can convert the VM to a template.

After we get the template, we need to set up a few things in the template.

* Remove the CD drive that ubuntu iso is mounted on.
* Add the `cloud-init drive` to the template. template> Hardware> Add> CloudInit Drive
* Configure the cloud-init drive in the template with username, password, ssh keys, etc. We need to change IP Config(net0) to DHCP to automatically get the ip addresses for the new VMs. Then click regenerate image.

### Creating Containers

* [Gemini Chat Link](https://gemini.google.com/app/82797783752b41ef)

Proxmox use Linux Containers or `LXC`(pronounced `lexi`).

Normally containers don't maintain state, you have to attach some kind of storage to them.

But LXC containers maintain state, so they act like mini VMs. Unlike VMs that virtualizes entire OS kernel and hardware, lxc containers share the host's kernel to create multiple isolated user-space environments.

You need CT templates to create containers and
You can download the templates from `your_node(pve1)> Local> CT Templates> Templates> search`

Try to ssh into the container, you will notice that **you can't ssh into the container showing permission denied**. That is because root ssh into container is not allowed.

To solve that we need to **create a user** and add it to `sudo` group.

```bash
adduser mkk
usermod -aG sudo mkk #-aG add Group groupname username
```

## Read More

* [Learn Linux clout-init](https://www.youtube.com/watch?v=exeuvgPxd-E)

## MISC

* `compgen -u` - list all users recongnized by the shell
* `compgen -g` - list all groups recongnized by the shell
* `adduser username` - create a user
* `usermod -aG groupname username` - add a user to a group
* `sudo ssh-keygen -A` - add ssh host keys to the system
* `sudo timedatectl set-timezone <timezone_name>` - to change timezone
