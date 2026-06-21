[![CI](https://github.com/guidugli/ansible-role-grub/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-grub/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-grub?label=release)](https://github.com/guidugli/ansible-role-grub/releases)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.grub-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/grub/)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-grub)](https://github.com/guidugli/ansible-role-grub/blob/main/LICENSE)

# Ansible Role: grub

Install and configure GRUB on Debian/Ubuntu and RedHat-family systems.

## Requirements

- A Linux host with GRUB available on the target operating system.
- This role is intended for bare metal or virtualized hosts. Container execution may be useful for limited Molecule validation, but full GRUB behavior does not apply in containerized environments.
- External privilege escalation must be provided by the calling play when required.

## Variables

Primary role variables, with defaults from `defaults/main.yml`:

```yaml
grub_cmdline_var_name: GRUB_CMDLINE_LINUX
grub_default_path: /etc/default/grub
grub_timeout: 5
grub_recordfail_timeout: "{{ grub_timeout }}"
grub_allow_reboot: false
# grub_options_present: []
# grub_options_absent: []
# grub_superuser: myuser
# yamllint disable-line rule:line-length
# grub_password: grub.pbkdf2.sha512.10000.65AA561A865A2CA878473E9080A65E9F0614AEB11BE9BC08DA8E48FF51A4B285B68C299908E75256C992104265C6C9A46A418C889FC5975DD183C501B4998BEA.E050D8AE711A6424E48A946D95C7D10C12A56BE1270939455D676ED7B07FA0307371EF835FB1C8E4B3EF78A78E62AE1F582908355296259C744DDE7E78D5AB19
```

Platform-derived variables from `vars/main.yml`:

- `grub_packages`
- `grub_d_path`
- `grub_boot_path`
- `grub_conf_path`
- `grub_update_grub_command`
- `_container_types`

## Example playbook

```yaml
---
- name: Configure grub
  hosts: all
  become: true
  vars:
    grub_timeout: 5
    grub_recordfail_timeout: "{{ grub_timeout }}"
    grub_options_present:
      - cgroup_enable=memory
      - quiet
      - some.option=complex,off
    grub_options_absent:
      - splash
      - rd.driver.pre
    grub_superuser: testuser
    # yamllint disable-line rule:line-length
    grub_password: grub.pbkdf2.sha512.10000.65AA561A865A2CA878473E9080A65E9F0614AEB11BE9BC08DA8E48FF51A4B285B68C299908E75256C992104265C6C9A46A418C889FC5975DD183C501B4998BEA.E050D8AE711A6424E48A946D95C7D10C12A56BE1270939455D676ED7B07FA0307371EF835FB1C8E4B3EF78A78E62AE1F582908355296259C744DDE7E78D5AB19
  roles:
    - role: guidugli.grub
```

## Molecule testing

The current role still contains legacy scenario-local Molecule files under `molecule/default/`. Those files were not modified in this modernization pass because they are outside the allowed edit scope.

## Execution notes

- **Privilege model**: the role does not define `become`. Callers must set privilege escalation externally when tasks touch privileged paths such as `/boot`, `/etc`, and package management.
- **Container behavior**: containerized runs may validate some file and option handling logic, but GRUB package behavior and bootloader updates are not representative in containers.
- **Systemd behavior**: this role does not currently manage systemd units directly.
