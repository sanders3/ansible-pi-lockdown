# Ansible Playbooks for Initial Raspberry Pi Lockdown

Simple Ansible playbooks, roles and tasks to lock down and perform initial setup for a new Raspberry Pi.

## Assumptions and Dependencies

These playbooks assume a freshly minted Raspberry Pi running the current version of either Raspbian or Raspbian Lite. Other Raspberry Pi distros exist and [YMMV](https://www.urbandictionary.com/define.php?term=ymmv).

These playbooks also assume that you have [Ansible installed](https://docs.ansible.com/ansible/latest/intro_installation.html) and ready on your control machine.

You also need to have `sshpass` installed on host machine
```bash
$ sudo apt-get install sshpass
```

## Inventory

When a Pi first boots it (usually) receives a DHCP assigned IP address, which the Lockdown playbook changes to a static IP.

To save having to create an inventory file and then immediately update it, these playbooks use a _feature_ of the `--inventory` command line argument for `ansible-playbook` where you can supply an IP address followed _**immediately**_ by a comma so that Ansible knows the inventory is a list of hosts (even though there's a single host being targeted).

Like this ... `--inventory 192.168.10.20,`

## Password Playbook

Changes the password for the default `pi` account.

Why the separate playbook? As this playbook changes the password that Ansible is using to authenticate, Ansible will have reload its inventory and host variables, which will fail as the password provided at the start of the playbook is no longer valid.

See [this discussion](https://github.com/ansible/ansible/issues/15227) for more background.

### Usage

```bash
$ ansible-playbook --user pi --ask-pass --inventory 'IP-ADDRESS,' password.yml
```

Running this playbook on a Raspberry Pi with an initial DHCP assigned IP address of `192.168.1.237` will look something like this.

```bash
$ cd plays
$ ansible-playbook --user pi --ask-pass --inventory '192.168.1.237,' password.yml
SSH password:
New pi account password:
confirm New pi account password:

PLAY [Default "pi" account password reset playbook] ****************************

TASK [Gathering Facts] *********************************************************
ok: [192.168.1.237]

TASK [pi-password : Set a new password for the default "pi" account] ***********
changed: [192.168.1.237]

PLAY RECAP *********************************************************************
192.168.1.237              : ok=2    changed=1    unreachable=0    failed=0   
```


## Lockdown Playbook

Performs some initial setup and lockdown on your new Pi.

* Sets the hostname for the Pi
* Creates a new user and deploys an SSH public key for the user
* Disables password authentication and enforces SSH key authentication
* Doesn't set a static IP address, because my hub will do that
* Expands the root filesystem to fill any remaining space on the Pi's SD card

### Usage

```bash
$ cd plays
$ ansible-playbook --user pi --ask-pass --inventory 'IP-ADDRESS,' lockdown.yml
```

Running this playbook on the same Raspberry Pi described above, with a static IP of `192.168.1.2` looks something like this (remember to use the new password for the `pi` account!)

```bash
$ ansible-playbook --user pi --ask-pass --inventory '192.168.1.237,' lockdown.yml
SSH password:
Hostname: dns.vicchi.local
User name: guest
Password:
confirm Password:
Username description: Guest Account
Path to public SSH key: /tmp/id_rsa.pub

PLAY [Application server specific playbook] ************************************

TASK [Gathering Facts] *********************************************************
ok: [192.168.1.237]

TASK [set-hostname : Set the hostname] *****************************************
changed: [192.168.1.237]

TASK [set-hostname : Update /etc/hosts with new hostname] **********************
changed: [192.168.1.237]

TASK [create-user : Create a (non default) user account] ***********************
changed: [192.168.1.237]

TASK [create-user : Deploy user's SSH key] *************************************
changed: [192.168.1.237]

TASK [disable-passwords : Disable SSH password authentication] *****************
changed: [192.168.1.237]

RUNNING HANDLER [disable-passwords : reboot] *******************************************
changed: [192.168.1.237]

PLAY RECAP *********************************************************************
192.168.1.237              : ok=7    changed=6    unreachable=1    failed=0  
```
