# raspberry-pi-rotary-phone #

Ansible role(s) for Raspberry Pi deployment, configuration, and maintenance

**STATUS**: Very much a Work-In-Progress! Not yet ready for using.

# Getting started #

## Customize inventory ##




### test (and copy SSH keys) ###

Update config in `bootstrap.yml` if necessary, after ensuring SSH identity is available and/or copying identity to `keys/`.



```
ansible-playbook -i inventory/inventory.cfg  bootstrap-playbook.yaml --ask-pass
```

## Deploy ##


Custom deployment

```
ansible-playbook -i inventory/inventory.cfg -v  -e "enable_apt_cacher_ng_proxy=true" \
	configure-playbook.yaml
```


# Credits #


Borrowed parts from

- https://galaxy.ansible.com/geerlingguy/raspberry-pi/ - Configures a Raspberry Pi.
  - https://github.com/geerlingguy/ansible-role-raspberry-pi (License: BSD, MIT)

- https://github.com/rhietala/raspberry-ansible - Example Ansible scripts for setting up Raspberry Pi with Raspbian OS
  - http://www.hietala.org/automating-raspberry-pi-setup-with-ansible.html
  - `roles/raspbian_bootstrap` and its subdirs under GPLv2 LICENSE

# TODO #

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
  - [ ] `noint do_hostname`
  - [ ] `noint do_boot_behavior {B1|B2|B3|B4}`
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
