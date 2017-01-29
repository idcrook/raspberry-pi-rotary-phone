# raspberry-pi-rotary-phone #

Ansible roles for Raspberry Pi deployment, configuration, and maintenance


## STATUS

Works amazingly well! In use for normalizing Raspbian-flavored Pis on Home LAN.

Developer configuration role (currently found in the _maintenance_ playbook/role) is very useful for configuring many Pi-s the same-- for example, if you need many Pi-s configured identically with software, git clones, etc. for a demo.

For further status, see TODO / DONE sections below for features desired, planned or completed.


# Getting started #

## Assumptions

 - ansible installed, control machine platform: _macOS_
   - may work on Linux, etc. but haven't tested
 - Raspbian SD card image flashed onto an SD card
   - SSH enabled?


## Enable SSH on image

So you've flashed a Raspbian-compatible image to an SD card?

In November 2016, there was a change made to defaults so that SSH server is not enabled by default. This will cause problems with "headless" installs on first-boot.

 https://www.raspberrypi.org/blog/a-security-update-for-raspbian-pixel/

> The boot partition on a Pi should be accessible from any machine with an SD
> card reader, on Windows, Mac, or Linux. If you want to enable SSH, all you
> need to do is to put a file called ssh in the /boot/ directory. The contents
> of the file don’t matter: it can contain any text you like, or even nothing
> at all.

OK. On macOS, the `/boot` partition will usually get mounted at `/Volumes/boot`. Touch will create an empty file if none exist at that location.

``` bash
sudo touch /Volumes/boot/ssh
```

Tell macOS to safely Eject. One way to do this is right-click on the "boot" volume in Finder/Desktop and use the `[Eject]` item.

## Connect to LAN and power up

Now you've updated the uSD card image to enable `ssh` and inserted into your Raspberry Pi(s)?

Power up Pi with Ethernet cable attached and connected to your LAN. Your ansible host (where you _run_ ansible) needs to be able to access the network the Pi is on in order to SSH to it.

The first boot in `jessie` Raspbian on recent images automatically performs the expand-fs step and reboots, so there is usually no need to explicitly do it any more. Your Pi will use DHCP to get an IP address, and should appear on the network with a Zeroconf address like `raspberrypi.local`

If you need troubleshoot anything, you should be able to connect a USB mouse+keyboard and HDMI monitor cable to access the console.

## Customize inventory ##

Copy example inventory file to your own (that will be edited).

```
cp inventory/inventory.cfg.example inventory/inventory.cfg
```

In the inventory file, the `ansible_host` IP must be SSH-able from the computer you are running ansible on. The inventory name (`rpi2` in example below) will be assigned on the Pi as its hostname when you run the playbook.

```
rpi2	ansible_host=10.0.1.52
```

You will need the IP address that get assigned to your Pi when it is booted up. Typically this is assigned by DHCP server on your network. Your Raspberry Pi _may also_ be addressable at a Zeroconf-assigned address like `raspberrypi.local`, since this is how a fresh Raspbian boots these days. However, a non-changing IP address for your Pi-s is a basic requirement for least trouble.

**RECOMMENDED**: On your network, use a router feature, sometimes called DHCP reservations or similar, to assign a DHCP IP "statically" to your Pi(s), especially if you have more than one Pi. These work by associating an Ethernet MAC address with an IP assignment. The ansible playbooks will work with as many Pi's as you have available, but if the IPs assigned are constantly changing, then there are going to be problems.

Something else to consider: set up **`~/.ssh/config`** to have the same matching hostnames and IP address as you used in your inventory file. Here's a matching example from my `~/.ssh/config` file:

``` ini
Host rpi2
  HostName 10.0.1.52
  User pi
```

## SSH keys for password-less  ###

Update SSH key config in `bootstrap-playbook.yaml` if necessary, after ensuring as SSH identity is available and/or copying identity to **`keys/`**. The SSH keys are used to configure password-less SSH access.  Search google for how to make an ssh identity using `ssh-keygen` if you do not already have one.

**FILES**

- `roles/raspbian_bootstrap/vars/main.yml`
``` ini
#ansible_ssh_private_key_file: "keys/id_rsa"
#dbs_ssh_pubkey: "{{ lookup('file', 'keys/id_rsa.pub') }}"
dbs_ssh_pubkey: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```


## Customize other config variables  ###

Many ansible control variables for the **`raspbian_bootstrap`** role should be edited to reflect your environment. The ansible files for these variable can be found at:

**FILES**

- `inventory/group_vars/all.yml`
- `roles/raspbian_bootstrap/defaults/main.yml`
- `roles/raspbian_bootstrap/vars/main.yml`

## Deploy ##

So you have your inventory up-to-date and have pointed to an SSH identity and customized and localized all the config variables? Run the playbook!

```
ansible-playbook -i inventory/inventory.cfg --ask-pass \
  bootstrap-playbook.yaml
```

You'll be greeted with two password prompts:

``` shell
SSH password:
Enter new password for user pi:
```

 1. The first is the SSH password for the `pi` user. **Default**: `raspberry`
 1. The second is for assigning a _new_ login password for `pi` user.

If you'd like, you can explicitly set ansible variables on the command line too, for example:

```
ansible-playbook -i inventory/inventory.cfg --ask-pass \
  -e "dbs_expand_filesystem=true" \
  -e "enable_apt_cacher_ng_proxy=true"  \
  bootstrap-playbook.yaml

```

Once you have successfully transferred your public SSH key, you should not need the `--ask-pass` option.

```
ansible-playbook -i inventory/inventory.cfg \
  -e "enable_apt_cacher_ng_proxy=true"  \
  bootstrap-playbook.yaml
```

## Maintenance

Once the bootstrap playbook is run successfully for a target host, you shouldn't need to run it again in the future. (unless you edit the ansible files, of course!)

There is another playbook for additional raspberry pi configuration. It is designated to perform a maintenance role, but also includes developer setup. It can even install Git repositories and NPM modules.

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

And deluxe IO enabling:

```
ansible-playbook \
  -e "enable_apt_cacher_ng_proxy=true"  \
  -e "rpicfg_disable_i2c=0"  \
  -e "rpicfg_disable_spi=0"  \
  -e "install_developer_tools=true"  \
  -e "install_git_repositories=true"  \
  maintenance-playbook.yaml
```




It has been tested successfully to run multiple times. So, if you need to upgrade to latest versions of packages, etc. it is there to help.

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
- [ ] wi-fi config
- [ ] GUI mode support vs. "headless"
- Servers
  - [ ] shairport-sync
- Package management
   - [ ] k8s, docker
   - [ ] IoT related (GPIO, node pkgs, python pkgs, IoT/cloud frameworks)
- `raspi-config`
  - Others
  - [ ] overscan
  - Dangerous:
  - [ ] overclock
  - [ ] serial

## DONE ##

- System config
- [x] Assert systemd (systemctl, etc. assume Raspbian jessie)
  - Update user login information
  - [x] Change password from default
  - [x] groups (serial, e.g.)
  - [x] copy an ssh key
- [x] Timezone, locale
- [x] bashrc, emacs, vimrc, etc.
- `raspi-config`
  - [x] `--expand-rootfs` and reboot
  - [x] `nonint do_hostname`
  - [x] `nonint do_boot_behavior {B1|B2|B3|B4}`
  - Dangerous:
  - [x] ssh
  - [x] Others
	- [x] memory split
	- [x] vnc
	- [x] wifi_country
	- [x] spi, i2c
- Servers
  - [x] Install NTP
- Package management
   - [x] custom package lists (dselect, screen, etc.)
   - [x] maintenance (apt update cache + upgrade + reboot, e.g.)
   - [x] Custom apt repositories config
   - [x] nodejs/nvm related
   - [x] python/pip related
