# Cisco Nexus 9k Configuration with Ansible

## Overview

This Ansible project automates the configuration of Cisco Nexus 9k switches. It includes enabling necessary features, setting up VLANs, SNMP, NTP, Syslog, Time Zone, Spanning Tree Protocol (STP), and Virtual Port Channel (VPC).

## Project Structure
```
ansible_project/
├── inventory
│   └── hosts
├── playbook.yml
├── config.j2
└── roles/
    └── main.yml
```


## Inventory File

Defines the switches and their credentials.

**Path:** `inventory/hosts`
```ini
[switches]
switch1 ansible_host=10.32.14.26 ansible_user=admin ansible_password=admin_password
switch2 ansible_host=10.32.15.103 ansible_user=admin ansible_password=admin_password
```
## Playbook File
Applies the switch configuration directly using the playbook and the config.j2 template located in the project root directory.

**Path:** `playbook.yml`
```yaml
---
- name: Configure Cisco Nexus 9k Switches
  hosts: switches
  gather_facts: false

  vars:
    ansible_connection: ansible.netcommon.network_cli
    ansible_network_os: cisco.nxos.nxos
    ansible_become: true
    ansible_become_method: enable

  tasks:
    - name: Generate NX-OS configuration from template
      template:
        src: config.j2
        dest: /tmp/config.cfg

    - name: Apply NX-OS configuration
      cisco.nxos.nxos_config:
        src: /tmp/config.cfg
```
## Variables File
Defines the configuration parameters for the switches.

**Path:** `roles/main.yml`

```yaml
ntp_server: "10.216.25.226"
syslog_server: "10.150.145.76"
timezone: "CST"
timezone_offset: -6
snmp_communities:
  - name: "netview"
    permission: "RO"
    group: "SystemOwner"
  - name: "n3tc00lnm"
    permission: "RO"
    group: "SystemOwner"
  - name: "0SSN3TC00L"
    permission: "RO"
    group: "SystemOwner"
  - name: "NII0r10n3g"
    permission: "RO"
    group: "SystemOwner"
  - name: "Pr1m3Su1t3_2016"
    permission: "RW"
    group: ""
  - name: "3PnM4n4g3R_2017!"
    permission: "RO"
    group: "SystemOwner"
snmp_contact: "mx.noc.ipcore@mx.att.com"
snmp_location: "NGDC MTY APODACA"
vlans:
  - { id: 2732, name: "IPMI_iDRAC" }
  - { id: 2733, name: "BAREMETAL_mgmt" }
  - { id: 2748, name: "LB_Nginx" }
  - { id: 2749, name: "App" }
```



## Template File
Defines the commands to configure the switches.

**Path:** `config.j2`
```j2
!
! Enable necessary features
!
feature lacp
feature interface-vlan
feature vtp
feature vpc
{% if vlans is defined and vlans | length > 0 %}
!
! Configure VLANs
!
{% for vlan in vlans %}
vlan {{ vlan.id }}
  name {{ vlan.name }}
{% endfor %}
{% else %}
! No VLANs to configure
{% endif %}
{% if snmp_contact is defined and snmp_location is defined %}
!
! Configure SNMP
!
snmp-server contact {{ snmp_contact }}
snmp-server location {{ snmp_location }}
{% for community in snmp_communities %}
  {% if community.name is defined and community.permission is defined %}
snmp-server community {{ community.name }} {{ community.permission }}
  {% endif %}
{% endfor %}
{% else %}
! SNMP settings not configured
{% endif %}
{% if ntp_server is defined %}
!
! Configure NTP
!
ntp server {{ ntp_server }} use-vrf management
{% else %}
! NTP server not configured
{% endif %}
{% if syslog_server is defined %}
!
! Configure Syslog
!
logging server {{ syslog_server }} 5 use-vrf management
{% else %}
! Syslog server not configured
{% endif %}
{% if timezone is defined and timezone_offset is defined %}
!
! Configure Time Zone
!
clock timezone {{ timezone }} {{ timezone_offset }} 0
{% else %}
! Timezone not configured
{% endif %}
!
! Configure Spanning Tree
!
interface Ethernet1/1
  spanning-tree portfast
  spanning-tree bpduguard enable
interface Ethernet1/2
  spanning-tree portfast
  spanning-tree bpduguard enable
!
! Configure VPC
!
vpc domain 1
  peer-switch
  role priority 1
  system priority 2500
  {% for switch_name in groups['switches'] %}
    {% if switch_name != inventory_hostname %}
  peer-keepalive destination {{ hostvars[switch_name]['ansible_host'] }} source {{ hostvars[inventory_hostname]['ansible_host'] }} vrf management
    {% endif %}
  {% endfor %}
!
interface port-channel10
  switchport mode trunk
  switchport trunk allowed vlan 2732,2733,2748,2749
  vpc peer-link
!
```

# How to Use
## 1. Create the Project Structure:

```bash
mkdir -p ansible_project/inventory
mkdir -p ansible_project/roles
```

## 2. Create the Necessary Files: create and populate the files as described above (or clone this repository)

## 3. Navigate to the Project Directory:

## 4. Install the Cisco NX-OS Collection:
```bash
ansible-galaxy collection install cisco.nxos
```

## 5. Run the Playbook:
```bash
ansible-playbook -i inventory/hosts playbook.yml
```
This command will execute the playbook playbook.yml using the inventory defined in `inventory/hosts`. The configurations will be generated from the playbook and applied to the Cisco Nexus 9k switches.



