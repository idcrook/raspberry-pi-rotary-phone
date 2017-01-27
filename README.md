# raspberry-pi-rotary-phone #

Ansible role(s) for Raspberry Pi deployment, configuration, and maintenance

**STATUS**: Very much a Work-In-Progress! Not yet ready for using.

# Getting started #

So you've flashed a raspbian-compatible image to an SD card and installed in your Raspberry Pi(s). Power it up with Ethernet cable attached. Your ansible host (where you _run_ ansible) needs to be able to access the network the Pi is on in order to SSH to it.

## Customize inventory ##

Copy template file to your own that will be editted.

```
cp inventory/inventory.cfg.example inventory/inventory.cfg
```

In the inventory file, the `ansible_host` IP must be SSH-able from the computer you are running ansible on. The inventory name (`rpi2` in example below) will be assigned on the Pi as its hostname when you run the playbook.

```
rpi2	ansible_host=10.0.1.52
```

You will need the IP address that get assigned to your Pi when it is booted up. Typically this is assigned by DHCP server on your network. You Pi _may_ also be addressable at a Zeroconf-assigned address like `raspberrypi.local`, since this is how a fresh Raspbian boots.

RECOMMENDED: Use a router feature, sometimes called DHCP reservations or similar, to assign a DHCP IP "statically" to your Pi(s), especially if you have more than one Pi. Ansible playbook will work with as many Pi's as you have available.


## SSH keys for password-less  ###

Update config in `bootstrap-playbook.yaml` if necessary, after ensuring as SSH identity is available and/or copying identity to `keys/`. This is used to configure password-less SSH access.


## Customize other config variables  ###

Many ansible-related control variables can be found and editted in `inventory/group_vars/all.yml`


## Deploy ##

So you have your inventory up-to-date and have pointed to SSH indentity and customized and local installation specific variables? Run the playbook!

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

If you'd like, you can set variables on the command line too:

```
ansible-playbook -i inventory/inventory.cfg --ask-pass \
  -e "enable_apt_cacher_ng_proxy=true"  \
  bootstrap-playbook.yaml
```

## Maintenance: TBD

Additional config and maintenance tasks

```
ansible-playbook -i inventory/inventory.cfg \
  -e "enable_apt_cacher_ng_proxy=true"  \
  maintenance-playbook.yaml
```

```
ansible-playbook \
  -e "enable_apt_cacher_ng_proxy=true"  \
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
   - [ ] maintenance (apt update cache + upgrade + reboot, e.g.)
   - [ ] k8s, docker
   - [ ] python/pip related
   - [ ] nodejs/nvm related
   - [ ] IoT related (GPIO, node pkgs, python pkgs, IoT/cloud frameworks)
- `raspi-config`
  - [ ] `--expand-rootfs` and reboot
  - [ ] `nonint do_hostname`
  - [ ] `nonint do_boot_behavior {B1|B2|B3|B4}`
  - [ ] Others
	- [ ] overscan
	- [ ] memory split
	- [ ] vnc
	- [ ] wifi_country
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
- Servers
  - [x] Install NTP
- Package management
   - [x] custom package lists (dselect, screen, etc.)
   - [x] maintenance (apt update cache + upgrade + reboot, e.g.)
   - [x] Custom apt repositories config
