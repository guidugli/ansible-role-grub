[![CI](https://github.com/guidugli/ansible-role-grub/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-grub/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-grub?label=release)](https://github.com/guidugli/ansible-role-grub/tags)
[![Galaxy](https://img.shields.io/ansible/role/d/guidugli/grub?label=galaxy)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/grub/)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-grub)](LICENSE)

## Ansible Role: grub

Configure GRUB defaults, kernel command line options, and optional bootloader authentication on Debian, Ubuntu,
and Red Hat family Linux systems.

### Requirements

- Ansible Core 2.14 or newer according to role metadata.
- A Linux host with GRUB tooling available through the target operating system package manager.
- Root-level permissions are required for package installation, `/etc/default/grub`, `/etc/grub.d`, `/boot/grub`,
  `/boot/grub2`, bootloader password files, GRUB configuration generation, and optional reboot handling.
- Supply privilege externally in your playbook, inventory, or automation platform.
- The `containers.podman` collection is required for Molecule container scenarios and is pinned in `requirements.yml`
  with a minimum version.

### Features

- Installs the platform-specific GRUB package set defined in `vars/main.yml` on non-container hosts.
- Uses `grub2-common` for Debian-family systems with apt metadata refresh guarded by `cache_valid_time`.
- Manages `GRUB_TIMEOUT` and `GRUB_RECORDFAIL_TIMEOUT` in the configured GRUB defaults file.
- Adds requested kernel command line options while preserving existing options that are not explicitly managed.
- Removes requested kernel option keys when the existing option is not also requested as present.
- Supports optional GRUB PBKDF2 bootloader authentication on Debian and Red Hat family systems.
- Edits `/etc/grub.d/10_linux` only when that package-managed script already exists.
- Runs GRUB configuration generation through a handler and skips that handler inside common container runtimes.
- Supports optional reboot handling when `grub_allow_reboot` is enabled.
- Includes shared Molecule converge and verify logic for default and systemd scenarios.

### Supported platforms

Role metadata lists Fedora, Ubuntu, and Debian. The bundled Molecule shared matrix includes Ubuntu 26.04 and 24.04,
Debian 13 and 12, and Fedora 44 and 43. Fedora metadata is rendered as `all` by the generator template, while
Ubuntu and Debian versions are rendered from the shared matrix.

### Variables

All public inputs are defined in `defaults/main.yml` and surfaced through `meta/argument_specs.yml`.

| Variable | Type | Default | Description |
| --- | --- | --- | --- |
| `grub_cmdline_var_name` | string | `GRUB_CMDLINE_LINUX` | GRUB defaults variable that contains kernel command line options. Supported values are `GRUB_CMDLINE_LINUX` and `GRUB_CMDLINE_LINUX_DEFAULT`. |
| `grub_default_path` | string | `/etc/default/grub` | Path to the GRUB defaults file. Use this when a distribution or site convention writes GRUB defaults elsewhere, such as under `/etc/default/grub.d`. |
| `grub_timeout` | integer | `5` | Value written to `GRUB_TIMEOUT`. Must be zero or greater. |
| `grub_recordfail_timeout` | string | `{{ grub_timeout }}` | Value written to `GRUB_RECORDFAIL_TIMEOUT`. Defaults to the same value as `grub_timeout`. |
| `grub_allow_reboot` | boolean | `false` | Allows the `Reboot` handler to reboot the host after GRUB changes. The reboot handler is skipped in common container runtimes. |
| `grub_options_present` | list of strings | undefined | Kernel command line options that must be present. Values may be bare flags such as `quiet` or key-value options such as `cgroup_enable=memory`. |
| `grub_options_absent` | list of strings | undefined | Kernel command line option keys that must be absent. Specify only the key, such as `splash` or `audit`. Existing options are not removed when the same key-value pair is also requested in `grub_options_present`. |
| `grub_superuser` | string | undefined | Optional GRUB superuser name used for bootloader authentication. |
| `grub_password` | string | undefined | Optional GRUB PBKDF2 password hash. Treat this value as sensitive and store it in Ansible Vault or another approved secret store. |

Platform-derived internal variables are defined in `vars/main.yml` and should normally not be overridden:

- `grub_packages`
- `grub_d_path`
- `grub_boot_path`
- `grub_conf_path`
- `grub_update_grub_command`
- `_container_types`

### Example playbook

```yaml
---
- name: Configure GRUB
  hosts: linux_hosts
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
    grub_superuser: bootadmin
    grub_password: !vault |
      $ANSIBLE_VAULT;1.1;AES256
      replace_with_your_vault_encrypted_grub_pbkdf2_hash
  roles:
    - role: guidugli.grub
```

### Molecule testing instructions

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

A convenience script is also present:

```bash
./scripts/run_local.sh
```

### Execution notes

- **Privilege model:** this role never declares `become`, `become_user`, or `become_method`. Use `become: true`
  at the play, inventory, or automation-controller level for real hosts that require elevated access to install
  packages, edit `/etc` or `/boot`, regenerate GRUB configuration, or reboot.
- **Container behavior:** Molecule containers normally execute as root and role tasks do not assume privilege
  escalation. Package installation is skipped in common container runtimes because bootloader package availability
  varies across minimal container images and GRUB bootloader behavior is not representative there. The role creates
  the container test boot path needed for file-management tasks. Real Debian-family hosts use apt metadata refresh
  with `cache_valid_time` so the idempotence phase does not repeatedly report cache changes. Default-file
  manipulation can be validated in containers, but bootloader installation, firmware interaction, and reboot
  behavior are not representative in containerized environments.
- **Package-managed GRUB scripts:** the role does not create `/etc/grub.d/10_linux`. It edits that script only when
  it already exists because the file is normally provided by the GRUB package.
- **Handler behavior:** update and reboot handlers are skipped when `ansible_facts['virtualization_type']` is one of
  the configured container runtimes.
- **Systemd behavior:** this role does not manage systemd services or unit files directly. The bundled systemd
  Molecule scenario remains useful for validating behavior in a systemd-capable container environment, especially
  handler and reboot safety boundaries.

### Release workflow

Generated repository metadata and inventories are refreshed through the shared generator scripts:

```bash
./scripts/update_release_metadata.sh
./scripts/release.sh --version v1.2.0 --message "Release v1.2.0"
```

### License

MIT

### Author

Carlos Guidugli
