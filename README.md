Ansible Role: grub
=========

An Ansible Role that configure grub on RHEL/CentOS, Fedora and Debian/Ubuntu.

Requirements
------------

Operating system running on bare metal or on a hypervisor virtualization. Grub does not work on conteinerized systems.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

    grub_cmdline_var_name: GRUB_CMDLINE_LINUX

Choose the variable name to be edited.
Valid values are: 
- GRUB_CMDLINE_LINUX
- GRUB_CMDLINE_LINUX_DEFAULT


    grub_default_path: /etc/default/grub

Full path of default grub settings.

    grub_timeout: 5

Grub menu timeout in seconds.

    grub_recordfail_timeout: "{{ grub_timeout }}"

Menu timeout if "recordfail" condition is true.

    grub_allow_reboot: no

Should role perform a reboot after setting up grub?

    #grub_options_present:
    #  - cgroup_enable=memory
    #  - quiet
    #  - some.option=complex,off

Options to be added to GRUB_CMDLINE_LINUX. Value is optional.

    #grub_options_absent:
    #  - splash
    #  - rd.driver.pre

Options to be removed from GRUB_CMDLINE_LINUX. Notice that only the key should be listed. For example, to remove audit=0, just add audit to the list. Use grub_options_present to ensure the proper value is present.
Note that if the options to be added won´t be deleted because the key is listed in this variable.


    #grub_superuser: myuser
    #grub_password: grub.pbkdf2.sha512.10000.65AA561A865A2CA878473E9080A65E9F0614AEB11BE9BC08DA8E48FF51A4B285B68C299908E75256C992104265C6C9A46A418C889FC5975DD183C501B4998BEA.E050D8AE711A6424E48A946D95C7D10C12A56BE1270939455D676ED7B07FA0307371EF835FB1C8E4B3EF78A78E62AE1F582908355296259C744DDE7E78D5AB19

Sets grub user and password.


    #grub_boot_path: /boot/grub2

Optional variable. The role will check if /boot/grub or /boot/grub2 are present in the target system IF this variable is not defined.

    #grub_cfg_path: /boot/grub2/grub.cfg

Optional variable. The role will check if it is located at grub_boot_path or at /boot/efi/EFI... if EFI is enabled on the system. Checking will happen only IF this variable is not defined.

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    grub_packages:

Packages that need to be installed to provide grub functionality.

    grub_update_grub_command:

Command to be used to issue grub update.

    grub_cmdline_var_name:

Represents the cmdline variable that needs to be changed in order to add or remove kernel options.

    grub_d_path:

Grub configuration directory.

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
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
        grub_password: grub.pbkdf2.sha512.10000.65AA561A865A2CA878473E9080A65E9F0614AEB11BE9BC08DA8E48FF51A4B285B68C299908E75256C992104265C6C9A46A418C889FC5975DD183C501B4998BEA.E050D8AE711A6424E48A946D95C7D10C12A56BE1270939455D676ED7B07FA0307371EF835FB1C8E4B3EF78A78E62AE1F582908355296259C744DDE7E78D5AB19 
      roles:
         - { guidugli.grub }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
