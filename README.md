# Ansible Role: Kexec

[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![GitHub tag](https://img.shields.io/github/tag/OnkelDom/ansible-role-kexec.svg)](https://github.com/OnkelDom/ansible-role-kexec/tags)
[![GitHub issues](https://img.shields.io/github/issues/OnkelDom/ansible-role-kexec)](https://github.com/OnkelDom/ansible-role-kexec/issues)
[![GitHub license](https://img.shields.io/github/license/OnkelDom/ansible-role-kexec)](https://github.com/OnkelDom/ansible-role-kexec/blob/master/LICENSE)

## Description

Reinstall your CentOS System with kexec [kexec](https://github.com/OnkelDom/ansible-role-kexec) using ansible. I wrote this role for rented vps servers to setup a clean installation.

## Password generation

You can generate passwords with the following command.
```bash
python -c 'import crypt,getpass;pw=getpass.getpass();print(crypt.crypt(pw) if (pw==getpass.getpass("Confirm: ")) else exit())'
```

## Requirements

- Ansible >= 2.5 (It might work on previous versions, but we cannot guarantee it)

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below.

| Name           | Default Value | Description                        |
| -------------- | ------------- | -----------------------------------|
| `kexec_os_install` | centos | OS to install. |
| `kexec_os_centos_version` | "8.2.2004" | OS release to install |
| `kexec_os_arch` | "x86_64" | OS architecture |
| `kexec_sshd_port` | "22" | SSH Port to open |
| `kexec_ks_root_pass` | "$1$TTCkItYV$T9ynqTURsryC4APnVorZe/" | Root Password (MySuperR00tP4assw0rd) |
| `kexec_ks_timezone` | "Europe/Berlin" | Set system timezone |
| `kexec_ks_system_lang` | "en_US.UTF-8" | Define your system language |
| `kexec_ks_keyboard_layout` | "us" | Define keyboard layout |
| `kexec_ks_selinux_state` | "permissive" | Define SELinux state |
| `kexec_ks_services` | "{ enabled: "sshd,firewalld,chronyd", disabled: [] }" | Define enabled/disabled system services |
| `kexec_ks_kickstarturl` | "http://mirror.centos.org/centos/{{ kexec_os_centos_version }}/BaseOS/{{ kexec_os_arch }}/kickstart/isolinux/" | Define kickstart url to download vmlinuz and initrd.img |
| `kexec_ks_repo` | "[- name: epel, url: "http://dl.fedoraproject.org/pub/epel/8/{{ kexec_os_arch }}/", save_to_disk: true ]" | Define repositorys for installation. With save_to_disk: true you can add them to system permanently |
| `kexec_ks_url` | "{ url: "http://mirror.centos.org/centos/{{ kexec_os_centos_version }}/BaseOS/{{ kexec_os_arch }}/os/", skip_verify_ssl: true, proxy: {} }" | Define network installation url |
| `kexec_ks_users` | "[- username: svc_ansible, groups: wheel, password: "$6$biO4a29mqIVZT0Sk$/IAlwjlcVJaPsABYMm1SZ/UjGfUOdk3s3oOKjPZ/4Qfr8MAkZiVbEfSJeGOtle/J3/Fw0c3beb7PnCr0HHBGp1", id: 1000, ssh_key: "{{ lookup('file', '~/.ssh/id_rsa_ansible.pub') }}" ]" | Define system user (Pass: svc_ansible) |
| `kexec_ks_network` | "{ hostname: "{{ ansible_hostname }}.{{ ansible_domain }}", ipv4_address: "{{ ansible_default_ipv4.address }}" , ipv4_netmask: "{{ ansible_default_ipv4.netmask }}" , ipv4_gateway: "{{ ansible_default_ipv4.gateway }}" , ipv4_nameservers: "{{ ansible_dns.nameservers[0] }},{{ ansible_dns.nameservers[1] }}" , ipv6_address: "{{ ansible_default_ipv6.address }}" , ipv6_cidr: "{{ ansible_default_ipv6.prefix }}" , ipv6_gateway: "{{ ansible_default_ipv6.gateway }}" }" | Define network settings |
| `kexec_ks_pvs` | "[ - name: pv.1, size: 8192, grow: true, ondrive: sda, crypt: false, crypt_pass: "SecureMyDisc", volumegroups: [ - name: system, logical_volumes: [ - name: swap, size: 2048, path: swap, - name: root, size: 16384, fstype: xfs, path: /, - name: log, size: 5020, fstype: xfs, path: /var/log ]]]" | Define system lvm |
| `kexec_ks_packages` | "sudo, openssh-server, curl, python3, ca-certificats, openssl, vim-enhanced" | Define packages to install |

## Example

```yaml
# Kickstart installation url
kexec_ks_kickstarturl: "http://mirror.centos.org/centos/{{ kexec_os_centos_version }}/BaseOS/{{ kexec_os_arch }}/kickstart/isolinux/"

# Network installation url
kexec_ks_url:
  url: "http://mirror.centos.org/centos/{{ kexec_os_centos_version }}/BaseOS/{{ kexec_os_arch }}/os/"
  skip_verify_ssl: true
  proxy: {}

# Add repos for installation
kexec_ks_repo:
# EPEL CentOS 8
  - name: epel
    url: "http://dl.fedoraproject.org/pub/epel/8/{{ kexec_os_arch }}/"
    save_to_disk: true

kexec_ks_services:
  enabled: "sshd,firewalld,chronyd"
  disabled: {}

# define your users
kexec_ks_users:
  - username: svc_ansible # pass: svc_ansible
    groups: wheel
    password: "$6$biO4a29mqIVZT0Sk$/IAlwjlcVJaPsABYMm1SZ/UjGfUOdk3s3oOKjPZ/4Qfr8MAkZiVbEfSJeGOtle/J3/Fw0c3beb7PnCr0HHBGp1"
    id: 1000
    ssh_key: "{{ lookup('file', '~/.ssh/id_rsa_ansible.pub') }}"

# define your network
kexec_ks_network:
  hostname: "{{ ansible_hostname }}.{{ ansible_domain }}"
  ipv4_address: "{{ ansible_default_ipv4.address }}"
  ipv4_netmask: "{{ ansible_default_ipv4.netmask }}"
  ipv4_gateway: "{{ ansible_default_ipv4.gateway }}"
  ipv4_nameservers: "{{ ansible_dns.nameservers[0] }},{{ ansible_dns.nameservers[1] }}"
  ipv6_address: "{{ ansible_default_ipv6.address }}"
  ipv6_cidr: "{{ ansible_default_ipv6.prefix }}"
  ipv6_gateway: "{{ ansible_default_ipv6.gateway }}"

# define your volumes
kexec_ks_pvs:
  - name: pv.1
    size: 8192
    grow: true
    ondrive: sda
    crypt: true
    crypt_pass: "SuperSaveP4ss"
    volumegroups:
      - name: system
        logical_volumes:
          - name: swap
            size: 2048
            path: swap
          - name: root
            size: 65536
            fstype: xfs
            path: /
          - name: log
            size: 10240
            fstype: xfs
            path: /var/log
          - name: data
            size: 65536
            fstype: xfs
            path: /opt
```

### Playbook

```yaml
- hosts: all
  roles:
    - onkeldom.ansible-role-kexec
  vars:
```

## Contributing

See [contributor guideline](CONTRIBUTING.md).

## License

This project is licensed under MIT License. See [LICENSE](/LICENSE) for more details.
