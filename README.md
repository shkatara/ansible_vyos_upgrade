Getting Started With VyOS / Cisco IOS Upgrade
===================================

I will not be explaining Ansible. There are plenty of good examples for it over the internet. This is just something I did when I was bored and someone asked for it.

## Some links for more in depth learning
* [Learn Ansible](https://www.ansible.com/resources/videos/quick-start-video) Official Ansible Documentation.
* [Ansible Module Index](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html) Official Ansible documentation for Ansible modules.


### Pre-requisites
This is going to have some good pre-requisites for you to work on getting your VyOS devices upgraded.

You need basic understanding of ansible with network devices, be comfortable with network_cli connection plugin and to have working knowledge of VyOS / CISCO IOS. 

You also need a working VyOS virtual machine.

### Things you'll need and setting up
--------------
This is done on with following setup.

1. A VyOS device. I am going with rolling version of vyos-1.3-rolling-202005280117-amd64.iso. You can download it from https://downloads.vyos.io/rolling/current/amd64/vyos-1.3-rolling-202006080117-amd64.iso

2. Ansible, obviously. I am using Ansible 2.8.1 from EPEL repo. You can use any ansible version above 2.7. You can get it from https://fedoraproject.org/wiki/EPEL

### VyOS Installation and configuration.
--------------------
1. Once you have created a vm for VyOS, ( it can be a standard 2GB Memory, 2 cpu and 5 GB machine ), you need to install the os to the disk because it first boots live. 

2. For installation steps, you can go to : https://wiki.vyos.net/wiki/Installation

### VyOS Configuration for Ansible
--------------------

1. There is not much we need to do, but we need to give a reachable IP address to the VM and tell it to allow ssh. This can be done using https://wiki.vyos.net/wiki/Network_address_setup

2. You also need to allow ssh for ansible to connect to. Have a look here, https://www.networkinghowtos.com/howto/enable-ssh-on-vyatta-vrouter-vyos/

Once done, you will have a workable ansible and a workable VyOS device which you can use to test your upgrade.

IMPORTANT : DO NOT CHANGE THE PASSWORD FOR VYOS USER. LET IT BE VYOS. WE WILL USE IT LATER.

3. Also, very important to make sure you have the outgoing connection from your VyOS device to the internet so it can download the new image for you. 

### Breaking the inventory 
------------------------
Do not change the name of the inventory file. Keep it 'inventory'

The inventory file has only one entry.

It is for the VyOS machine. Make sure to change the IP ADDRESS ONLY for the vm. 

1. ansible_password variable defines the password as 'vyos'
2. ansible_connection variable defines the connection type to 'network_cli'
3. ansible_network_os variable defines what type of Network OS we are using with the value as 'vyos'

ALL THESE 3 VARIABLES ARE MANDATORY.

Note: I am not using ansible-vault to save the password securely. I am just using for the ease. Feel free to make changes if you feel like.

### Testing the connectivity.
---------------------
Once making sure you have changed the IP address for your VM, we can now go on and check the connection to vm.

shub ocp4.2 ~ % ansible all -m vyos_facts

If all is good, variables and files in their place, it will be giving you the facts, making sure you can connect to the vm. 

### Upgrading the device. 
-------------------------
Now, for the big elephant in the room, the upgrade. Let us break it down to the steps.

1. We are first checking if the system is vyos, if not, we stop.
2. We show the old system information to verify it is not having any new images.
3. We download the iso on the filesystem. We could also have added it from the url directly, but then it fails the gpgcheck. Adding it from the filesystem only checks the MD5 sum.
4. We install the iso from the filesystem. This is where the fun begins. We are using the cli_command module which can handle multiple prompts because upgrading the VyOS vm requires a couple of prompts. 
5. Once installed, the latest installed image automatically becomes the default boot image. We now print the new image version. 

### Going into action
---------------------

The playbook that upgrades the devices does not automatically reboot the vm. You will need to do it manually. The system, once rebooted, will automatically boot into the new installed image.

I however, am keeping the same image I am using right now as default. You can change that from the playbook if you want to.

shub ocp4.2 ~/ansi % ansible-playbook fixed_play_wget.yml 

This will go through all the steps and will make sure your VyOS device is upgraded to the latest version.

