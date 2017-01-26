# raspberry-pi-rotary-phone

Ansible role for Raspberry Pi deployment and configuration


**STATUS**: Very much a Work-In-Progress! Not yet ready for using.


# Getting started

## Customize inventory




### test (and copy SSH keys)

Update config in `bootstrap.yml` if necessary, after ensuring SSH identity is available and/or copying identity to `keys/`.



```
ansible-playbook -i inventory/inventory.cfg  bootstrap-playbook.yaml --ask-pass
```

## Deploy


Custom deployment

```
ansible-playbook -i inventory/inventory.cfg -v  -e "enable_apt_cacher_ng_proxy=true" \
	configure-playbook.yaml
```


# Credits


Borrowed parts from

- https://galaxy.ansible.com/geerlingguy/raspberry-pi/ - Configures a Raspberry Pi.
  - https://github.com/geerlingguy/ansible-role-raspberry-pi (License: BSD, MIT)

- https://github.com/rhietala/raspberry-ansible - Example Ansible scripts for setting up Raspberry Pi with Raspbian OS
  - http://www.hietala.org/automating-raspberry-pi-setup-with-ansible.html
  - `roles/raspbian_bootstrap` and its subdirs under GPLv2 LICENSE

# TODO

- [ ] `raspi-config`
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
- [ ] Assert systemd (systemctl, etc. assume Raspbian jessie)
  - [ ] Install NTP
- [ ] Change user login information
  - [ ] password from default
  - [ ] groups (serial, e.g.)
  - [ ] copy an ssh key
- [ ] Timezone, locale
- [ ] bashrc, emacs, vimrc, etc.
- [ ] wi-fi config
- [ ] GUI vs. headless
- [ ] Package management
   - [ ] custom package lists (dselect, screen, etc.)
   - [ ] maintenance (apt update cache + upgrade + reboot, e.g.)
   - [ ] Custom apt repositories config
   - [ ] k8s, docker
   - [ ] python/pip related
   - [ ] nodejs/nvm related
   - [ ] IoT related (GPIO, node pkgs, python pkgs, IoT/cloud frameworks)
