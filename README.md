# raspberry-pi-rotary-phone #

Ansible roles for Raspberry Pi deployment, configuration, and maintenance


<!-- toc -->



<!-- tocstop -->


# STATUS

Works amazingly well! In use for normalizing Raspbian-flavored Pis on Home LAN.

Developer configuration role (currently found in the _maintenance_ playbook/role) is very useful for configuring many Pi-s the same-- for example, if you need many Pi-s configured identically with software, git clones, etc. for a demo.

For further status, see TODO / DONE sections below for features desired, planned or completed.

# GETTING STARTED #

## Assumptions

 - git clone of this repository!
 - ansible installed, control machine platform: _macOS_
   - may work from Linux control machine, but hasn't been tested
 - Raspbian `jessie` SD card image flashed onto an SD/µSD card
   - SSH server enabled on Pi (see following)
 - wired Ethernet network with DHCP

### Enable SSH on image

So you've flashed a Raspbian-compatible image to an SD card?

In November 2016, there was a change made to defaults so that SSH server is not enabled by default. This would cause problems with "headless" installs via ansible on first-boot. Fortunately there is a simple way available to override.

 https://www.raspberrypi.org/blog/a-security-update-for-raspbian-pixel/

> The boot partition on a Pi should be accessible from any machine with an SD
> card reader, on Windows, Mac, or Linux. If you want to enable SSH, all you
> need to do is to put a file called ssh in the /boot/ directory. The contents
> of the file don’t matter: it can contain any text you like, or even nothing
> at all.

We will be changing the password from the default soon in our ansible bootstrap role, so it will be OK enable the SSH server.

OK. On macOS, the SD card in its reader will usually automount the image's `/boot` partition at `/Volumes/boot`. `touch` will create an empty file if none exist at that location (`sudo` is not necessary in this example, but shouldn't hurt).

``` bash
sudo touch /Volumes/boot/ssh
```

Tell macOS to safely Eject the SD card once you have written the `/boot/ssh` file, to make sure it gets written to the card. One way to do this is right-click on the "boot" volume in Finder/Desktop and use the `[Eject]` item. Another way is (assuming it's where the `/boot` partition on SD card was mounted):

``` bash
diskutil eject /Volumes/boot
```



## Connect to LAN and power up

Now you've updated the µSD card image to enable `ssh` server and inserted into your Raspberry Pi(s)?

Power up Pi with Ethernet cable attached and connected to your LAN. Your ansible control machine (where you _run_ ansible) needs to be able to access the network the Pi is on in order to SSH to it.

The first boot in `jessie` Raspbian in recent image updates automatically performs the expand-fs step and reboots, so there is usually no need to explicitly do it any more. Your Pi will attempt to use DHCP to get an IP address. It should also appear on network with a Zeroconf address like `raspberrypi.local`. If that works, you should be able to reveal IP address with something like:

``` bash
$ ping raspberrypi.local
PING raspberrypi.local (10.0.1.87): 56 data bytes
64 bytes from 10.0.1.87: icmp_seq=0 ttl=64 time=0.859 ms
64 bytes from 10.0.1.87: icmp_seq=1 ttl=64 time=0.796 ms
```

The IP address (`10.0.1.87` in the example) is needed for your ansible `inventory`, described below.

If you need to troubleshoot anything, you should be able to connect a USB mouse+keyboard and HDMI monitor cable to access the console.

## Customize inventory ##

Copy example `inventory.cfg` file to your own:

```
cp inventory/inventory.cfg.example inventory/inventory.cfg
```

 Edit this new copy to customize. In the inventory file, the `ansible_host` IP must be SSH-able from the computer you are running ansible on. The inventory name (`rpi2` in example below) will be assigned on the Pi as its hostname when you run the playbook.

```
[raspberrypi-headless]
rpi2	ansible_host=10.0.1.87
```

You will need the IP address that get assigned to your Pi when it is booted up. Typically this is assigned by DHCP server on your network. Your Raspberry Pi _may also_ be addressable at a Zeroconf-assigned address like `raspberrypi.local`, since this is how a fresh Raspbian install image boots. However, a non-changing IP address for your Pi-s is a basic requirement for least trouble with ansible.

**RECOMMENDED**: On your network, use a router feature, sometimes called DHCP reservations or similar, to assign a DHCP IP "statically" to your Pi(s), especially if you have more than one Pi. These work by associating an Ethernet MAC address with an IP assignment. The ansible playbooks will work with as many Pi's as you have available, but if their IP addresses assigned are constantly changing, then there are going to be problems.

Something else to consider: set up **`~/.ssh/config`** to have the same matching hostnames and IP address as you used in your inventory file. Here's a matching example from my `~/.ssh/config` file:

``` ini
Host rpi2
  HostName 10.0.1.87
  User pi
```

That way a command like `ssh rpi2` will connect to the Pi.

## SSH keys for password-less  ###

Update SSH key config in `bootstrap-playbook.yaml` if necessary. First, will need to ensure an SSH identity is available. And identity may be  copied to **`keys/`** subdirectory and pointed to for ansible. The SSH keys are used to configure password-less SSH access.  Search google for how to make an ssh identity using `ssh-keygen` if you do not already have one. The default location of `~/.ssh/id_rsa` is also the default in the ansible config.

**FILES**

- `roles/raspbian_bootstrap/vars/main.yml`
``` ini
#ansible_ssh_private_key_file: "keys/id_rsa"
#dbs_ssh_pubkey: "{{ lookup('file', 'keys/id_rsa.pub') }}"
dbs_ssh_pubkey: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```

### `ssh-copy-id` - to copy to hosts in advance

`ssh-copy-id` available as macOS [Homebrew](http://brew.sh) package is another useful tool. If you have SSH identities it can be used to copy them to another host (host: `raspberrypi.local` in following example). You will need to provide ssh login password:

``` bash
ssh-copy-id -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null pi@raspberrypi.local
```

This is how I recommend to get your keys to the target host (_before_ running ansible), as ansible will sometimes get confused when you change the password out from under it, and error out.  the `--ask-pass` option below can be skipped if you can already "password-less" SSH to target Pi using ssh keys.


## Customize other config variables  ###

Many ansible control variables for the **`raspbian_bootstrap`** role should be edited to reflect your environment. The ansible files for these variable can be found at:

**FILES**

- `inventory/group_vars/all.yml`
- `roles/raspbian_bootstrap/defaults/main.yml`
- `roles/raspbian_bootstrap/vars/main.yml`

Take a look to customize defaults, update values for your environment, and other tweaks.

## Deploy!

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
  -e "enable_apt_cacher_ng_proxy=true"  \
  bootstrap-playbook.yaml

```

Explained above, recent images of Raspbian `jessie` automatically perform the filesystem expansion on first boot, so you shouldn't need `-e "dbs_expand_filesystem=true"`. Once you have successfully transferred your public SSH key (bootstrap-playbook does this), the `--ask-pass` option in later invocations is not required.

```
ansible-playbook -i inventory/inventory.cfg \
  -e "enable_apt_cacher_ng_proxy=true"  \
  bootstrap-playbook.yaml
```

## Optional: Wi-Fi

You can configure wifi. You will need to update some configuration files to point to your network and use an ansible vault. After having done these steps, when you run the **maintenance playbook** (described below), it will also configure your wi-fi network for the designated hosts.


### Add wifi hosts to inventory

First add nodes you want to have wifi to your `inventory.cfg` file, in the `[raspberrypi-wifi]` section. Note they will be in this section in _addition_ to one of the headless/GUI sections.

### Point `ansible.cfg` to vault password file

Uncomment this line

``` ini
#vault_password_file = ~/.ansible_vault_password.txt
```

### Use vault file for Wi-Fi network and password

The file checked into the repository has a default vault password of `raspberry`. You'll want to update the vault password as well as the files it protects.

The steps in the following example will set the vault password to this default value, and open editor (likely `vi`) on decrypted version of file to allow edits, then re-encrypt file when you have made your edits and exited.

``` bash
echo "raspberry" > ~/.ansible_vault_password.txt
ansible-vault edit    inventory/group_vars/raspberrypi-wifi/vault.yml \
  --vault-password-file ~/.ansible_vault_password.txt
```

There are other modes to use `ansible-vault` for manipulating vault files.

```
ansible-vault decrypt inventory/group_vars/raspberrypi-wifi/vault.yml \
  --vault-password-file ~/.ansible_vault_password.txt

ansible-vault encrypt inventory/group_vars/raspberrypi-wifi/vault.yml \
  --vault-password-file ~/.ansible_vault_password.txt
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

# LICENSE #

MIT

# CREDITS #


Borrowed parts or ideas from

- https://galaxy.ansible.com/geerlingguy/raspberry-pi/ - Configures a Raspberry Pi.
  - https://github.com/geerlingguy/ansible-role-raspberry-pi (License: BSD, MIT)

- https://github.com/rhietala/raspberry-ansible - Example Ansible scripts for setting up Raspberry Pi with Raspbian OS
  - http://www.hietala.org/automating-raspberry-pi-setup-with-ansible.html
  - `roles/raspbian_bootstrap` and its subdirs under GPLv2 LICENSE

- https://github.com/thomo/ansible-raspi_wifi - Raspberry Pi Wi-Fi role

# TODO #

Add ansible tags such as `maintenance`, `wifi`.

- System config
- [ ] GUI mode support vs. "headless"
- [ ] Determine generation (Pi 0, B, B+, 2, 3, etc.)
- Servers / Developer
- [ ] MQTT server for IoT
  - [ ] IoT related (node pkgs, python pkgs, IoT/cloud frameworks)
- [ ] python virtualenv
- [ ] shairport-sync
- [ ] swiftlang http://swift-lite.org/installation/
- [ ] golang
- [ ] xrdp (remote desktop)
- [ ] csharp (mono)
- [ ] k8s, docker


## TODO - Low priority

- `raspi-config`
  - Others
  - [ ] overscan
  - Dangerous:
  - [ ] overclock
  - [ ] serial

# DONE #

- System config
- [x] wi-fi config
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
