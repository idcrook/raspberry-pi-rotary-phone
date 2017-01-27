# raspberry-pi-rotary-phone #

Ansible roles for Raspberry Pi deployment, configuration, and maintenance


## STATUS

A Work-In-Progress

Have successfully normalized a Pi and did  developer configuration

See TODO / DONE section at end for tasks in the roles and features desired, planned or completed.


# Getting started #

**Assumptions**

 - ansible installed, control machine platform: _macOS_
   - may work on Linux, etc. but haven't tested

So you've flashed a raspbian-compatible image to an SD card and installed in your Raspberry Pi(s)?

Power it up with Ethernet cable attached. Your ansible host (where you _run_ ansible) needs to be able to access the network the Pi is on in order to SSH to it.

## Customize inventory ##

Copy template file to your own that will be editted.

```
cp inventory/inventory.cfg.example inventory/inventory.cfg
```

In the inventory file, the `ansible_host` IP must be SSH-able from the computer you are running ansible on. The inventory name (`rpi2` in example below) will be assigned on the Pi as its hostname when you run the playbook.

```
rpi2	ansible_host=10.0.1.52
```

You will need the IP address that get assigned to your Pi when it is booted up. Typically this is assigned by DHCP server on your network. Your Raspberry Pi _may_ also be addressable at a Zeroconf-assigned address like `raspberrypi.local`, since this is how a fresh Raspbian boots these days.

RECOMMENDED: On your network, use a router feature, sometimes called DHCP reservations or similar, to assign a DHCP IP "statically" to your Pi(s), especially if you have more than one Pi. The ansible playbooks will work with as many Pi's as you have available, but if the IPs assigned are constantly changing, then there are goings to be problems.



## SSH keys for password-less  ###

Update config in `bootstrap-playbook.yaml` if necessary, after ensuring as SSH identity is available and/or copying identity to `keys/`. This is used to configure password-less SSH access.

**FILES**

- `roles/raspbian_bootstrap/vars/main.yml`
``` ini
#ansible_ssh_private_key_file: "keys/id_rsa"
#dbs_ssh_pubkey: "{{ lookup('file', 'keys/id_rsa.pub') }}"
dbs_ssh_pubkey: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```


## Customize other config variables  ###

Many ansible control variables should be edited to reflect your envrironment and can be found in:

**FILES**

- `inventory/group_vars/all.yml`
- `roles/raspbian_bootstrap/vars/main.yml`

## Deploy ##

So you have your inventory up-to-date and have pointed to an SSH identity and customized and localized control variables? Run the playbook!

```
ansible-playbook -i inventory/inventory.cfg --ask-pass \
  bootstrap-playbook.yaml
```

You'll be greeted with two password prompts:

``` shell
SSH password:
Enter new password for user pi:
```

The first is the SSH password for the `pi` user. **Default**: `raspberry`
The second is for assigning a _new_ login password for `pi` user.

If you'd like, you can explicitly set ansible variables on the command line too, for example:

```
ansible-playbook -i inventory/inventory.cfg --ask-pass \
  -e "enable_apt_cacher_ng_proxy=true"  \
  bootstrap-playbook.yaml
```

## Maintenance

Once the bootstrap playbook is run successfully for a target host, you shouldn't need to run it again in the future. (unless you edit the ansible files, of course!)

There is another playbook for additional raspberry pi configuration and developer setup. It is designated to run perform the maintenance role.

**FILES**

- `inventory/group_vars/all.yml`
- `roles/raspbian-maintenance/vars/main.yml`

```
ansible-playbook -i inventory/inventory.cfg \
  maintenance-playbook.yaml
```

Here is how I have run it

```
ansible-playbook \
  -e "enable_apt_cacher_ng_proxy=true"  \
  -e "install_developer_tools=true"  \
  -e "install_git_repositories=true"  \
  maintenance-playbook.yaml
```


# Credits #


Borrowed parts from

- https://galaxy.ansible.com/geerlingguy/raspberry-pi/ - Configures a Raspberry Pi.
  - https://github.com/geerlingguy/ansible-role-raspberry-pi (License: BSD, MIT)

- https://github.com/rhietala/raspberry-ansible - Example Ansible scripts for setting up Raspberry Pi with Raspbian OS
  - http://www.hietala.org/automating-raspberry-pi-setup-with-ansible.html
  - `roles/raspbian_bootstrap` and its subdirs under GPLv2 LICENSE

# TODO #

Add ansible tags such as `maintenance`.

- System config
- [ ] Assert systemd (systemctl, etc. assume Raspbian jessie)
- [ ] wi-fi config
- [ ] GUI vs. headless
- Servers
  - [ ] shairport-sync
- Package management
   - [ ] k8s, docker
   - [ ] IoT related (GPIO, node pkgs, python pkgs, IoT/cloud frameworks)
- `raspi-config`
  - [ ] `--expand-rootfs` and reboot
  - [ ] Others
	- [ ] overscan
	- [ ] overclock
	- [ ] spi, i2c,
  - [ ] Dangerous: ssh, serial

## DONE ##

- System config
  - Update user login information
  - [x] Change password from default
  - [x] groups (serial, e.g.)
  - [x] copy an ssh key
- [x] Timezone, locale
- [x] bashrc, emacs, vimrc, etc.
- `raspi-config`
  - [x] `nonint do_hostname`
  - [x] `nonint do_boot_behavior {B1|B2|B3|B4}`
  - [x] Others
	- [x] memory split
	- [x] vnc
	- [x] wifi_country
- Servers
  - [x] Install NTP
- Package management
   - [x] custom package lists (dselect, screen, etc.)
   - [x] maintenance (apt update cache + upgrade + reboot, e.g.)
   - [x] Custom apt repositories config
   - [x] nodejs/nvm related
   - [x] python/pip related
