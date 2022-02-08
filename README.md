# ![logo](https://upload.wikimedia.org/wikipedia/commons/2/24/Ansible_logo.svg)
[//]: # (Ansible/Red Hat, Public domain, via Wikimedia Commons)

A simple repo to hold playbooks that I have written over the years for a variety of purposes


## License

[![GPLv3 License](https://img.shields.io/badge/License-GPL%20v3-yellow.svg)](https://opensource.org/licenses/)

## Allow Root SSH

[![ ](https://img.shields.io/badge/Ansible-v2.9-brightgreen)](https://docs.ansible.com/ansible/latest/roadmap/ROADMAP_2_9.html)  
[allowrootssh.yml](allowrootssh.yml)

- This playbook will:
  - Operate on all hosts in the Ansible inventory
  - The tasks this playbook will complete are:
    - Update the `/etc/ssh/sshd_config` file to allow root logins (`"PermitRootLogin yes"`)
    - It will restart the SSH Daemon with a `handler`

## Disable Root SSH

[![ ](https://img.shields.io/badge/Ansible-v2.9-brightgreen)](https://docs.ansible.com/ansible/latest/roadmap/ROADMAP_2_9.html)  
[disablerootssh.yml](disablerootssh.yml)

- This playbook will:
  - Operate on all hosts in the Ansible inventory
  - The tasks this playbook will complete are:
    - Update the `/etc/ssh/sshd_config` file to disable root logins (`"PermitRootLogin no"`)
    - It will restart the SSH Daemon with a `handler`

## Linux Patching

[![ ](https://img.shields.io/badge/Ansible-v2.9-brightgreen)](https://docs.ansible.com/ansible/latest/roadmap/ROADMAP_2_9.html)  
[linux-patching.yml](linux-patching.yml)

- This playbook will:
  - Operate on all hosts in the Ansible inventory
  - Achieve super user privelages by using `sudo`
  - Lookup the currently running `"$USER"` from the environment variables and assign them as the Ansible `remote_user`
  - The tasks this playbook will complete are:
      - Stop the `systemd` service `chef-client`, and disable it.
      - Find `/etc/yum.repos.d/` repo files, and register the results in `repofiles`
      - Loop through `repofiles` and replace all instances of `enabled=1` with `enabled=0` (_It also accounts for a possible space around the '=' symbol_)
      - Check to make sure cache exists, and register the results in `yumcache`
      - Remove YUM Cache, `when` `yumcache.stat.exists`
      - Run the command `yum clean all`
      - Update all RHEL Packages with `state: latest` and register the ouput in `packageupdate`
      - Get the kernel name-version-release, store the output in `rpm_output`
      - Loop through the `stdout_lines` of `rpm_output` and `reboot` the server when the kernel matches (_Prior to running the playbook, the admin should update the playbook with an expected kernel 'name-version-release'_)
      - Loop through `repofiles` and replace all instances of `enabled=0` with `enabled=1` (_It also accounts for a possible space around the '=' symbol_)
      - Restart the `systemd` service `chef-client`, and enable it.

## Secure SSH

[![ ](https://img.shields.io/badge/Ansible-v2.9-brightgreen)](https://docs.ansible.com/ansible/latest/roadmap/ROADMAP_2_9.html)  
[secure-ssh.yml](secure-ssh.yml)

- This playbook will:
  - Operate on all hosts in the Ansible inventory
  - Achieve super user privelages by using `sudo`
  - Gather facts
  - Set `vars`: `arcfour256`, `arcfour`, `aes192-cbc`, `aes256-cbc`, `diffie-hellman-group1-sha1`, `diffie-hellman-group-exchange-sha1`
  - The tasks this playbook will complete are:
    - Set a fact `sshchange` to `false`
    - Check the `/etc/ssh/sshd_config` for commented lines that contain the `Kex` Algorithm, and if found, uncomment them.
    - Check if KexAlgorithms is present
    - Add KexAlgorithms if missing on RHEL 7 or more
    - Add KexAlgorithms if missing on RHEL 6
    - Ciphers - Uncomment if commented
    - Check if Ciphers is present
    - Add Ciphers if missing on RHEL 7 or more
    - Add Ciphers if missing on RHEL 6
    - Loop on the `vars` established at the start, and if found, remove them from `/etc/ssh/sshd_config`
    - Disable root login over SSH
    - Restart SSH Daemon

## Update Sudo
###### This playbook was written to resolve CVE-2021-3156

[![ ](https://img.shields.io/badge/ALERT-CVE--2021--3156-red)](https://nvd.nist.gov/vuln/detail/CVE-2021-3156)  
[![ ](https://img.shields.io/badge/Ansible-v2.9-brightgreen)](https://docs.ansible.com/ansible/latest/roadmap/ROADMAP_2_9.html)  
[UpdateSudo.yml](UpdateSudo.yml)

- This playbook will:
  - Operate on all hosts in the Ansible inventory
  - Achieve super user privelages by using `sudo`
  - Gather facts
  - The tasks this playbook will complete are:
    - Gather package facts
    - Set sudo version from discovered package facts: (`"{{ ansible_facts.packages.sudo[0].version }}-{{ ansible_facts.packages.sudo[0].release }}"`)
    - Install rsync if needed
    - Sync an updated version of sudo to affected hosts, and notify handlers to both install and cleanup files
      - Oracle Enterprise Linux 6/7
      - RedHat Enterprise Linux 6/7
      - CentOS Linux 6/7

[//]: # (End Of Document)