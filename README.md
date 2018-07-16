# netctl

Role for deploy profiles for systemd-based network-manager
[netctl](//github.com/joukewitteveen/netctl).

## Requirements for usage

* Ansible 2.5+;

## Link types supported

* ethernet (inc. pseudo, like [IMQ](//github.com/imq/linuximq));
* vlan (802.1q);
* pppoe;
* mobile_ppp (USB modem support);
* tunnel (ipip, gre, etc);
* bonding;
* bridge;

## Extra

* ip addresses, before deploy will be checked by Jinja with python-netaddr lib;
* Support 'static' and 'dhcp' connections;
* Support arrays of addresses, DNS;
* DHCP client can be reassigned and accept options;
* Any `ip` commands can be executed with profile;
* Support for link options (run `ip link add type bond help` to see the
  available options);
* Support PostUp and PreDown command execution;

## Example configuration

```yaml
---
netctl_restart: 'false'
netctl_enable: 'false'
netctl_drop_exists_interfaces: 'true'
netctl_drop_exists_hooks: 'true'
netctl_deploy_interfaces: 'true'
netctl_deploy_interface_hooks: 'true'
netctl_deploy_global_hooks: 'true'

# Whenever a profile is read, all executable scripts in '/etc/netctl/hooks' and any
# executable script in '/etc/netctl/interfaces' with the name of the interface for
# the profile are sourced. Declarations in an interface script override declarations
# in a profile, which override declarations in hooks.
netctl_hooks:
- name: 'hook1'
  action:
  - 'echo 1 > /proc/net/super'
  - 'echo 2 > /proc/net/super'

netctl_interfaces:
# Interface name
- interface: 'lag0'
# A description of the profile.
  description: 'Bonded interface'
# The connection type used by the profile. Available connection types:
#
# ethernet - For wired connections.
# bond - For bonded interfaces.
# bridge - For bridge interfaces.
# pppoe - For PPPoE connections.
# mobile_ppp - For mobile broadband PPP connections that use a USB modem.
# tunnel - For tunnel interfaces.
# tuntap - For TUN/TAP interfaces.
# vlan - For VLANs on ethernet-like connections.
  connection: 'bond'
# Bonding slaves, one or more devices.
  physdev:
  - 'eth0'
  - 'eth1'
# One of 'static' or 'dhcp'.
  ip: 'static'
# One nameserver or list of nameservers.
  dns: '127.0.0.1'
# Whether or not the absence of a carrier (plugged-in cable) is acceptable.
# Defaults to 'no'.
  skip_no_carrier: 'yes'
# Set to 'yes' to force connecting even if the interface is up. Do not use this
# unless you know what you are doing.
  force_connect: 'yes'
# Maximum time, in seconds, to wait for an interface to get up. Defaults to '5'.
  timeout_up: '5'
# Set to 'yes' to consider the profile activated only when it is online.
  wait_online: 'no'
# Additional options to be passed to ip link. Run `ip link add type bond help`
# to see the available options.
  link_options:
  - 'mode 4'
  - 'xmit_hash_policy 2'
  - 'lacp_rate 1'
  - 'miimon 100'
# A command that is executed after a connection is established. If the specified
# command returns anything other than 0 (success), netctl will abort and stop
# the profile. If the command should be allowed to fail, add '|| true' to the
# end of it.
  ifup_local:
  - 'ethtool -K eth0 tso off'
  - 'ethtool -K eth1 tso off'
# A command that is executed before a connection is brought down. Similar precautions
# should be taken as with 'ifup_local'.
  ifdown_local:
  - 'ethtool -K eth0 tso on'
  - 'ethtool -K eth1 tso on'
# An list to pass to ip. This can be used to achieve complicated configurations
# within the framework of netctl.
  ip_custom:
  - 'link set dev eth0 txqueuelen 10000'
  - 'link set dev eth1 txqueuelen 10000'
  hook:
  - 'systemctl restart application'
- interface: 'vlan2'
  connection: 'vlan'
# The underlying physical interface.
  physdev: 'lag0'
# vid
  vlan_id: '2'
  ip: 'static'
# Example of list of addresses.
  address:
  - '10.7.0.6/24'
  - '10.8.0.6/24'
  gateway: '10.7.0.1'
# Example of list of DNS'es.
  dns:
  - '8.8.8.8'
  - '8.8.4.4'
- interface: 'vlan3'
  connection: 'vlan'
  physdev: 'lag0'
  vlan_id: '3'
  ip: 'static'
- interface: 'ppp0'
  connection: 'pppoe'
# The interface to dial peer-to-peer over ethernet.
  physdev: 'vlan3'
# The username to connect with.
  ppp_user: 'user123'
# The password to connect with.
  ppp_password: 'secret'
# This option specifies how a connection should be established, and may take either
# 'persist' or 'demand' as its argument.
  ppp_mode: 'persist'
# The number of consecutive failed connection attempts to tolerate. A value of '0'
# means no limit. Defaults to '5'.
  ppp_max_fail: '0'
# Use the default route provided by the peer (defaults to 'true').
  ppp_default_route: 'false'
# Use the DNS provided by the peer (defaults to 'true').
  ppp_peer_dns: 'false'
# Set the ppp unit number in the interface name (ppp0, ppp1, etc.).
  ppp_unit: '0'
  force_connect: 'yes'
- interface: 'ppp1'
  connection: 'mobile_ppp'
  physdev: '/dev/ttyUSB0'
  ppp_user: ''
  ppp_password: ''
# This option is used to specify the connection mode. Can be one of '3Gpref',
# '3Gonly', 'GPRSpref', 'GPRSonly', 'None'. This generates AT commands specific
# to certain Huawei modems, all other devices should use 'None'.
  ppp_mode: 'None'
# The access point (apn) to connect on. This is specific to your ISP.
  ppp_apn: 'internet.yota'
# The number to dial. Defaults to '*99#'.
  ppp_number: '*99#'
# An initialization string sent to the modem before dialing. This string is sent
# after sending 'ATZ'. Defaults to 'ATQ0 V1 E1 S0=0 &C1&D2 +FCLASS=0'.
  ppp_init: ''
  ppp_default_route: 'true'
  ppp_peer_dns: 'true'
  ppp_max_fail: '0'
  ppp_unit: '1'
- interface: 'Tunnel0'
  connection: 'tunnel'
# The tunnel type (e.g. 'gre'). See ip-tunnel(8) for available modes.
  tun_mode: 'ipip'
# The address of the local end of the tunnel.
  tun_local: '5.128.220.100'
# The address of the remote end of the tunnel.
  tun_remote: '5.128.220.50'
- interface: 'Tunnel1'
  connection: 'tunnel'
  tun_mode: 'gre'
  tun_local: '5.128.220.150'
  tun_remote: '5.128.220.250'
  ip: 'static'
  address: '10.17.17.1/24'
```
