# Week 2 Day 5

## Proxmox

### Creating Container templates

* Delete ssh host keys
* apt update and apt upgrade
* Clean apt cache
* Delete machine id with truncate.
* Ensure the symbolic link to machine id from /var/lib/dbus/machine-id is present.
* Then convert the container to a template.
* After converting, log in into the container and run the following.x`
* `sudo dpkg-reconfigure openssh-server` - regen the ssh keys on freshly installed clone containers.

### User management on Proxmox

Click Datacenter > Users

#### Two realms for users

* pam - linux auth system
* pve - proxmox user - proxmox VE auth system

The different is where the user is created and where the info on user is stored. **only PAM users can ssh into the servers and manage them**.
Even tho you created a user with PAM, when you check the user in proxmox shell with `cat /etc/passwd` you won't see the user. You need to add the user via command line `adduser mkk`. only then you can see the user. If you create a user with PVE, you can't see the user in shell either.

If you log out and log back in with any user, you can't see anything except the node pve1.
We need to change that.

To change that,

1. Create a new group
2. Then click Permissions and add group permission with necessary permissions.
3. Add user to the group.

### Backupsups and Snapshots

Backup is the clone of actual disk. You can move the backup to other media.
Snapshot is the part of the VM itself. You can't move or copy to other media.

When you are testing a software, you don't know how the software will behave or you are not sure if you like it or not. You can take a snapshot of the VM before installing it and revert back to that, if you don't like the software. You can always uninstall the software of course, but some software might leave cache file and jump files and they might remain.

#### Snapshots

Install, apache2 in VM then

* 102 (webserver)> Snapshots> Take snapshot

Uninstall apache2, install nginx, then

* 102 (webserver)> Snapshots> Rollback to snapshot_name

If you take the snapshot with Include RAM option, it will save the RAM as well, maintining the state of the VM.

#### Backups

* 102 (webserver)> Backup> Backup Now

There is 3 mode, **Snapshot, Suspend, Stop**, when to choose which one depends on the situation, how much downtime you can afford.

* **Stop** - Proxmox will orderly shutdown the VM and take a backup, **having the most consistent backup**. After that Proxmox will restart the VM if it was running before the backup
* **Suspend** - Proxmox will put the VM on suspend like on laptop or desktop then take a backup. In suspend, you won't be able to access the VM. It will be like the VM is off. This mode take the most amount of downtime, but it increase the consistency of the backup.
* **Snapshot** - Proxmox will take a snapshot of the VM and take a backup. This mode take the least amount of downtime, but it decrease the consistency of the backup. Installing qemu-guest-agent on VM is a must(Help with increasing consistency).

## Read More

* [End point security](https://www.fortinet.com/resources/cyberglossary/what-is-endpoint-security)
* [Why does DNS use TCP Port 53 and UDP Port 53](https://www.techtarget.com/searchnetworking/tip/Why-does-DNS-use-TCP-Port-53-and-UDP-Port-53)
* [Virtual Patching](https://www.indusface.com/learning/what-is-virtual-patching/)

## MISC

* `sudo dpkg-reconfigure openssh-server` - re-run the config scripts and to regenrate ssh host keys if the are missing.
* Use case and example always think. why.