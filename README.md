# Cisco Nexus 9k Configuration with Ansible

## Overview

This Ansible project automates the configuration of Cisco Nexus 9k switches. It includes enabling necessary features, setting up VLANs, SNMP, NTP, Syslog, Time Zone, TACACS+, Spanning Tree Protocol (STP), and Virtual Port Channel (VPC).

## Project Structure
```
ansible_project/
├── inventory
│   └── hosts
├── playbook.yml
└── roles/
    └── cisco_nxos/
        ├── tasks/
        │   └── main.yml
        ├── templates/
        │   └── config.j2
        └── vars/
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
Calls the cisco_nxos role to configure the switches.

**Path:** `playbook.yml`
```yaml
---
- name: Configure Cisco Nexus 9k Switches
  hosts: switches
  gather_facts: false
  roles:
    - cisco_nxos
```
## Variables File
Defines the configuration parameters for the switches.

**Path:** `roles/cisco_nxos/vars/main.yml`

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

## Tasks File
Generates the NX-OS configuration from the template and applies it to the switches.

**Path:** `roles/cisco_nxos/tasks/main.yml`
```yaml
---
- name: Generate NX-OS configuration
  template:
    src: config.j2
    dest: /tmp/config.cfg

- name: Apply NX-OS configuration
  cisco.nxos.nxos_config:
    src: /tmp/config.cfg
```

## Template File
Defines the commands to configure the switches.

**Path:** `roles/cisco_nxos/templates/config.j2`
```j2
!
! Enable necessary features
!
feature lacp
feature interface-vlan
feature vtp
feature vpc
!
! Configure VLANs
!
{% for vlan in vlans %}
vlan {{ vlan.id }}
  name {{ vlan.name }}
{% endfor %}
!
! Configure SNMP
!
snmp-server contact {{ snmp_contact }}
snmp-server location {{ snmp_location }}
{% for community in snmp_communities %}
snmp-server community {{ community.name }} {{ community.permission }} {{ community.group }}
{% endfor %}
!
! Configure NTP
!
ntp server {{ ntp_server }} use-vrf management
!
! Configure Syslog
!
logging server {{ syslog_server }} 5 use-vrf management
!
! Configure Time Zone
!
clock timezone {{ timezone }} {{ timezone_offset }}
!
! Configure TACACS+
!
tacacs-server host 10.32.15.100 key 0oT$v&2OabSe5Q
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
  peer-keepalive destination {{ hostvars['switch2']['ansible_host'] }} source {{ hostvars['switch1']['ansible_host'] }} vrf management
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
mkdir -p ansible_project/roles/cisco_nxos/tasks
mkdir -p ansible_project/roles/cisco_nxos/templates
mkdir -p ansible_project/roles/cisco_nxos/vars
```

## 2. Create the Necessary Files: create and populate the files as described above (or clone this repository)

## 3. Navigate to the Project Directory:
```bash
ansible-galaxy collection install cisco.nxos
```
## 4. Install the Cisco NX-OS Collection:
```bash
ansible-galaxy collection install cisco.nxos
```

## 5. Run the Playbook:
```bash
ansible-playbook -i inventory/hosts playbook.yml
```
This command will execute the playbook playbook.yml using the inventory defined in inventory/hosts. The configurations will be generated from the template and applied to the Cisco Nexus 9k switches.



