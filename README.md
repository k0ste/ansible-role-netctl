ansible-role-netctl
======================

Role for deploy profiles for systemd-based network-manager - [netctl](//github.com/joukewitteveen/netctl).

Ansible versions
--------------------
Role is adapted for Ansible 2.0, should work on 1.9.

Requirements for usage
-----------------------------------

* ArchLinux based distro. Becasue I really don't know any other distro who use
this. In order not to break anything, will not touch them;
* netctl;
* ppp, if pppoe is used;
* [python-netaddr](//docs.ansible.com/ansible/playbooks_filters_ipaddr.html) library (on machine with Ansible);

Link types supported
-----------------------

* ethernet (inc. pseudo, like [IMQ](//github.com/imq/linuximq));
* vlan (802.1q);
* pppoe (not all options, only what I use);
* mobile_ppp (USB modem support);
* tunnel (ipip, gre, etc);
* bond (only with same parameters, for now netctl can't up multiple different bonds);

Extra
-----------

* ip addresses, before deploy will be checked by Jinja with python-netaddr lib;
* Support 'static' and 'dhcp' connections;
* Support arrays of addresses, DNS;
* DHCP client can be reassigned and accept options;
* Support assign options for Bonding Linux module;
* Any `ip` commands can be executed with profile;
* Support PostUp and PreDown command execution;

Example configuration
-------------------------

```yaml
---
netctl_restart: 'true'
netctl_enable: 'true'

netctl_address_array:
 - 10.7.0.6/24
 - 10.8.0.6/24
netctl_bond:
  - eth0
  - eth1
netctl_custom:
  - 'link set dev eth0 txqueuelen 10000'
  - 'link set dev eth1 txqueuelen 10000'
netctl_bonding:
  miimon: '100'
  mode: '4'
  xmit_hash_policy: '2'
netctl_exec:
  - 'ethtool -K eth0 tso off'
  - 'ethtool -K eth1 tso off'

netctl:
- {
description: 'BOND0',
connection: 'bond',
interface: 'bond0',
physdev: 'eth0',
ip: 'static',
dns: '127.0.0.1',
skip_no_carrier: 'true',
force_connect: 'true',
exec_up_post: "{{ hostvars[inventory_hostname]['netctl_exec'] }}",
ip_custom: "{{ hostvars[inventory_hostname]['netctl_custom'] }}"
}
- {
description: 'VLAN100',
connection: 'vlan',
interface: 'vlan100',
physdev: 'bond0',
vlan_id: '100',
ip: 'static',
address: "{{ hostvars[inventory_hostname]['netctl_address_array'] }}",
gateway: '10.7.0.1'
}
- {
description: 'VLAN111',
connection: 'vlan',
interface: 'vlan111',
physdev: 'bond0',
vlan_id: '111',
ip: 'static',
address: '172.16.15.1/24'
}
- {
description: 'VLAN1998',
connection: 'vlan',
interface: 'vlan1998',
physdev: 'bond0',
vlan_id: '1998',
ip: 'static',
address: '172.16.14.128/25'
}
- {
description: 'VLAN1999',
connection: 'vlan',
interface: 'vlan1999',
physdev: 'bond0',
vlan_id: '1999',
ip: 'static',
address: '172.16.14.1/25'
}
- {
description: 'VLAN2000',
connection: 'vlan',
interface: 'vlan2000',
physdev: 'bond0',
vlan_id: '2000',
ip: 'static',
address: '192.168.130.193/26'
}
- {
description: 'VLAN2048',
connection: 'vlan',
interface: 'vlan2048',
physdev: 'bond0',
vlan_id: '2048',
ip: 'static',
address: '192.168.2.1/24'
}
- {
description: 'VLAN66',
connection: 'vlan',
interface: 'vlan66',
physdev: 'bond0',
vlan_id: '66',
ip: 'static',
address: '172.16.66.1/24'
}
- {
description: 'VLAN666',
connection: 'vlan',
interface: 'vlan666',
physdev: 'bond0',
vlan_id: '666',
ip: 'static',
address: '198.18.2.1/24'
}
- {
description: 'VLAN777',
connection: 'vlan',
interface: 'vlan777',
physdev: 'bond0',
vlan_id: '777',
ip: 'static',
address: '172.16.17.254/24'
}
- {
description: 'VLAN888',
connection: 'vlan',
interface: 'vlan888',
physdev: 'bond0',
vlan_id: '888',
ip: 'static',
address: '172.16.18.2/24'
}
- {
description: 'PPP0',
connection: 'pppoe',
interface: 'ppp0',
physdev: 'vlan888',
ppp_user: 'ppp_USERQWFWF',
ppp_password: 'pspspdq',
ppp_mode: 'persist',
ppp_max_fail: '0',
ppp_default_route: 'false',
ppp_peer_dns: 'false',
ppp_unit: '0',
force_connect: 'true'
}
- {
connection: 'mobile_ppp',
interface: 'ppp1',
physdev: '/dev/ttyUSB0',
ppp_user: '',
ppp_password: '',
ppp_apn: 'internet.yota',
ppp_number: '*99#',
ppp_default_route: 'true',
ppp_peer_dns: 'true',
ppp_max_fail: '0',
ppp_unit: '1'
}
- {
connection: 'tunnel',
interface: 'Tunnel0',
tun_mode: 'ipip',
tun_local: '5.128.220.100',
tun_remote: '5.128.220.50'
}
- {
connection: 'tunnel',
interface: 'Tunnel1',
tun_mode: 'gre',
tun_local: '5.128.220.150',
tun_remote: '5.128.220.250',
ip: 'static',
address: '10.17.17.1/24'
}
```
