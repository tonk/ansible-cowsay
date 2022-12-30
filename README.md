# Ansible Cowsay, without Cowsay

When I was installing an ansible AAP cluster, I decided
that I wanted a nice message of the day, when you log into the system.
Problem was that the execution environment container does not have the `cowsay`
program installed. Trying to install it with `yum` and `dnf` failed because of
failing Python modules. This turned out to be a chicken and egg problem.
As I only need the Cowsay part and nothing fancy, I created an Ansible `cowsay`
module, which returns the text supplied into a Cowsay message.

An example playbook

```
---
- name: Set a nice message of the day
  hosts: cowhost
  become: false

  vars:
    motd_title: The bestest Mmhoo
    motd_alias: cn01
    motd_func: Ansible AAP Control node
    motd_location: Amsterdam
    motd_vmname: EXDTONKE01

  tasks:
    - name: Get cowsay
      cowsay:
        text: "{{ motd_title }}"
      register: cowsaid
      delegate_to: localhost

    - name: Generate system information
      ansible.builtin.template:
        src: motd.j2
        dest: /etc/motd
        owner: root
        group: root
        mode: 0644
      become: true
```

## Requirements

There are no requirements

## Variables for a nice motd

Variables are per host and identify the host
  - `motd_title` - The functionality of the host
    e.g. Member of the AAP cluster
    This is the text in the Cow balloon
  - `motd_alias` - A host alias
    e.g. `cn01`
  - `motd_func` - The specific functionality of this host
    e.g. Ansible AAP Execution node
  - `motd_loc` - The location of the machine
    e.g. Amsterdam
  - `motd_vmname` - The name of the VM as it is know in VMware
    e.g. EXDTONKE01


### motd template

```
{#

This template creates a motd file with contents like below.

   ___________________________
  < Member of the AAP Cluster >      | Host name   : thor
   ---------------------------       | Host alias  : cn01
              \   ^__^               | Function    : Ansible AAP Control node
               \  (oo)\_______       | Location    : Amsterdam
                  (__)\       )\/\   | IP address  : 192.168.63.194
                      ||----w |      | VMware name : EXDTONKE01
                      ||     ||

#}

{% set ns = namespace(l=15) %}
{% for line in cowsaid['message'] %}
{% set ns.l = [ns.l, line | length, 15] | max %}
{% endfor %}

  {{ "{1:<{0}}".format(ns.l+2, cowsaid['message'][0]) }}
  {{ "{1:<{0}}".format(ns.l+2, cowsaid['message'][1]) }} | Host name   : {{ ansible_facts['hostname'] }}
  {{ "{1:<{0}}".format(ns.l+2, cowsaid['message'][2]) }} | Host alias  : {{ motd_alias | default(inventory_hostname) }}
  {{ "{1:<{0}}".format(ns.l+2, cowsaid['message'][3]) }} | Function    : {{ motd_func }}
  {{ "{1:<{0}}".format(ns.l+2, cowsaid['message'][4]) }} | Location    : {{ motd_location }}
  {{ "{1:<{0}}".format(ns.l+2, cowsaid['message'][5]) }} | IP address  : {{ ansible_facts['default_ipv4']['address'] }}
  {{ "{1:<{0}}".format(ns.l+2, cowsaid['message'][6]) }} | VMware name : {{ motd_vmname }}
  {{ "{1:<{0}}".format(ns.l+2, cowsaid['message'][7]) }}

```

Example output

```
   ___________________________
  < Member of the AAP Cluster >      | Host name   : thor
   ---------------------------       | Host alias  : localhost
              \   ^__^               | Function    : Ansible AAP Control node
               \  (oo)\_______       | Location    : Groesbeek
                  (__)\       )\/\   | IP address  : 10.20.30.40
                      ||----w |      | VMware name : EXDTONKE01
                      ||     ||

```

## Author Information
- 2022-12-30 - Ton Kersten <@tonk>
  Fixed `check` mode. Just return a cowsaid message.

- 2022-11-18 - Ton Kersten <@tonk>
  Initial version
