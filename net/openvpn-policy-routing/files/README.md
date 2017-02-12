# OpenVPN Policy-Based Routing

## Description
This service allows you to define rules (policies) for routing traffic via WAN or your OpenVPN tunnel(s). Policies can be set based on any combination of local/remote ports, local/remote IPv4 or IPv6 addresses/subnets or domains. This service supersedes the [VPN Bypass](https://github.com/openwrt/packages/blob/master/net/vpnbypass/files/README.md) service, by supporting IPv6 and by allowing you to set explicit rules not just for WAN gateway (bypassing OpenVPN tunnel), but for multiple OpenVPN tunnels as well.

## Features

#### Gateways/Tunnels
- Any policy can target either WAN or an OpenVPN tunnel.
- Multiple OpenVPN interfaces/tunnels supported (with device names tun\* or tap\*).

#### IPv4/IPv6/Port-Based Policies
- Policies based on local IPs/subnets. You can specify a single IP (as in ```192.168.1.70```) or a local subnet (as in ```192.168.1.81/29```). IPv6 addresses are also supported.
- Policies based on local ports numbers. Can be set as an individual port number (```32400```), comma-separated list (```80,8080```) or a range (```5060-5061```). Limited to 15 ports per policy.
- Policies based on remote IPs/subnets. Same format/syntax as local IPs/subnets.
- Policies based on remote ports numbers. Same format/syntax and restrictions as local ports.

#### Domain-Based Policies
- Policies can be comprised of lists of domains which are accessed via WAN (```wanroute```) or specific VPN tunnel (```tun0route```, ```tun1route```, etc.); useful if you want to access some web-sites (like Netflix, Hulu, etc.) via specific gateway/tunnel.
- Lists should be in following format:  ```/domain1.com/domain2.com/route``` where ```route``` must be either of ```wanroute```, ```tun0route```, ```tun1route``` etc.
- If ```dnsmasq``` is installed and running, supports ```ipset``` feature (as of now, only ```dnsmasq-full``` package supports ```ipset``` and not the basic ```dnsmasq``` package) and detected as active name-resolution service, OpenVPN Policy-Based Routing Web UI lets you edit the ```/etc/config/dhcp``` file for domain-based routing. Service will create required ipsets, but will rely on ```dnsmasq``` (please see [Requirements](#requirements) for details) to handle domain-based policies. If you want to add domain-based policies manually, please either edit ```/etc/config/dhcp``` file or run commands like:
```sh
uci add_list dhcp.@dnsmasq[0].ipset='/hulu.com/netflix.com/nhl.com/wanroute'
uci add_list dhcp.@dnsmasq[0].ipset='/github.com/google.com/tun0route'
uci commit dhcp
```
- Otherwise, OpenVPN Policy-Based Routing Web UI lets you edit internal domain-based policy configuration (which is not as efficient or elegant as one in ```dnsmasq-full```), please see [Notes/Known Issues](#notesknown-issues) for details.

#### DSCP-tag Based Policies
You can also set policies for traffic with specific DSCP tag. Instructions for tagging specific app traffic in Windows 10 can be found [here](http://serverfault.com/questions/769843/cannot-set-dscp-on-windows-10-pro-via-group-policy).

#### Strict enforcement
- Supports strict policy enforcement, even if the policy gateway is down -- resulting in network being unreachable for specific policy (enabled by default).

#### Customization
- Can be fully configured with uci commands or by editing ```/etc/config/openvpn-policy-routing``` file.
- Has a companion package (luci-app-openvpn-policy-routing) so everything can be configured with Web UI.

#### Other Features
- Doesn't stay in memory, modifies ```dnsmasq``` configuration and creates the ip tables/iptables rules which are automatically updated on WAN or OpenVPN tunnels changes.
- Proudly made in Canada, using locally-sourced electrons.

## Screenshot (luci-app-openvpn-policy-routing)
![screenshot](https://raw.githubusercontent.com/stangri/screenshots/master/openvpn-policy-routing/screenshot01.png "screenshot")

## Requirements
This service requires the following packages to be installed on your router: ```ipset``` and ```iptables```. Additionally, if you want to use [Domain-Based Policies](#domainbased-policies), you are strongly encouraged to install ```dnsmasq-full``` (```dnsmasq-full``` requires you uninstall ```dnsmasq``` first).

To satisfy the requirements for both [IPv4/IPv6/Port-Based Policies](#ipv4ipv6portbased-policies) and [Domain-Based Policies](#domainbased-policies) connect to your router via ssh and run the following commands:
```sh
opkg update; opkg remove dnsmasq; opkg install ipset iptables dnsmasq-full
```

To satisfy the requirements for just [IPv4/IPv6/Port-Based Policies](#ipv4ipv6portbased-policies) connect to your router via ssh and run the following commands:
```sh
opkg update; opkg install ipset iptables
```

#### Unmet dependencies
If you are running a development (trunk/snapshot) build of OpenWrt/LEDE Project on your router and your build is outdated (meaning that packages of the same revision/commit hash are no longer available and when you try to satisfy the [requirements](#requirements) you get errors), please flash either current LEDE release image or current development/snapshot image.

## How to install
Please make sure that the [requirements](#requirements) are satisfied and install ```openvpn-policy-routing``` and ```luci-app-openvpn-policy-routing``` from Web UI or connect to your router via ssh and run the following commands:
```sh
opkg update
opkg install openvpn-policy-routing luci-app-openvpn-policy-routing
```
If these packages are not found in the official feed/repo for your version of OpenWrt/LEDE Project, you will need to [add a custom repo to your router](#add-custom-repo-to-your-router) first.

#### Add custom repo to your router
If your router is not set up with the access to repository containing these packages you will need to add custom repository to your router by connecting to your router via ssh and running the following commands:

###### OpenWrt CC 15.05.1
```sh
opkg update; opkg install wget libopenssl
echo -e -n 'untrusted comment: public key 7ffc7517c4cc0c56\nRWR//HUXxMwMVnx7fESOKO7x8XoW4/dRidJPjt91hAAU2L59mYvHy0Fa\n' > /tmp/stangri-repo.pub && opkg-key add /tmp/stangri-repo.pub
! grep -q 'stangri_repo' /etc/opkg/customfeeds.conf && echo 'src/gz stangri_repo https://raw.githubusercontent.com/stangri/openwrt-repo/master' >> /etc/opkg/customfeeds.conf
opkg update
```

###### LEDE Project and OpenWrt DD trunk
```sh
opkg update; opkg install uclient-fetch libustream-mbedtls
echo -e -n 'untrusted comment: public key 7ffc7517c4cc0c56\nRWR//HUXxMwMVnx7fESOKO7x8XoW4/dRidJPjt91hAAU2L59mYvHy0Fa\n' > /tmp/stangri-repo.pub && opkg-key add /tmp/stangri-repo.pub
! grep -q 'stangri_repo' /etc/opkg/customfeeds.conf && echo 'src/gz stangri_repo https://raw.githubusercontent.com/stangri/openwrt-repo/master' >> /etc/opkg/customfeeds.conf
opkg update
```

## Default Settings
Default configuration has service disabled (use Web UI to enable/start service or run ```uci set openvpn-policy-routing.config.enabled=1```) and has the following example policies which can be safely deleted:
- Routes local Plex Media Server traffic (port 32400) via WAN gateway.
- Routes LogmeIn Hamachi traffic (25.0.0.0/8) via WAN gateway.
- Routes internet traffic from local IPs 192.168.1.70 and 192.168.1.81-192.168.1.86 via WAN gateway.

## Discussion
Please head to [LEDE Project Forum](https://forum.lede-project.org/t/openvpn-policy-based-routing-web-ui/1422) or [OpenWrt Forum](https://forum.openwrt.org/viewtopic.php?pid=351193) for discussions of this service.

## What's New
4.1.4
- No longer depends on specific WAN interface name (```wan```) can be used with custom WAN interface names (like ```wwan```).
- Include WAN interface information in ```support``` command.
- Starting table number, FW_MARK and FW_MASK can be defined in the config.
- Return to config-file based service enable/disable.

4.1.3
- More reliable creation/destruction of iptables OVPBR_MARK chain.

4.1.2
- Better output logic/format.

4.1.1
- New extra command: ```support```. Run ```/etc/init.d/openvpn-policy-routing``` for details.

4.1.0
- More elegant handling of iptables (thanks [@hnyman](https://github.com/hnyman) and [@tohojo](https://github.com/tohojo)!).

4.0.0
- Return to dnsmasq-full for domain-based policy routing (enabled by updated luci-app-openvpn-policy-routing directly editing dnsmasq config).
- README overhaul in preparation for official OpenWrt feed PR.

3.3.5
- Attempt to implement internal (dnsmasq-full independent) domain-based policy routing.
- More reliable way of obtaining WAN gateway on boot (thanks [@dibdot](https://github.com/dibdot) for the hint!).
- Changed format of config file for domains-based routing.
- Fixed bugs:
  - Duplicate ipset entries after using luci-app-openvpn-policy-routing.
  - Domains-based routing only targeting last dnsmasq config.
- Added support for routed domains in format ```/domain1.com/domain2.com/gwroute``` where ```gwroute``` could be ```wanroute```, ```tun0route```, etc.
- Support for automatic reload of openvpn-policy-routing service on OpenVPN tunnels changes.
- Support for strict policy enforcement, even if the policy gateway is down -- resulting in network being unreachable for specific policy (enabled by default).
- Support for multiple OpenVPN tunnels.

2.0.0:
- Proper support for local/remote IP addresses/ranges and ports.

1.0.0:
- Initial release.

## Notes/Known Issues
- In order to select specific OpenVPN tunnel in Web UI, the appropriate tunnel must be running. You can assign an inactive tunnel to any policy with the uci command or by manually editing ```/etc/config/openvpn-policy-routing``` file.

- Service does not alter the default routing. Depending on your OpenVPN settings (and settings of the OpenVPN server you are connecting to), the default routing might be set to go via WAN or via OpenVPN tunnel. This service affects only routing of the traffic matching the policies. If you want to override default routing, consider adding the following to your OpenVPN tunnel(s) configs:
```
option route_nopull '1'
option route '0.0.0.0 0.0.0.0'
```

#### Internal domain-based policies handling
- Preferred way for the domain-based policies is to rely on ```dnsmasq-full``` to work. If, for some reason, you don't have (or can't have) ```dnsmasq-full``` installed and running, you can configure openvpn-policy-routing to use its own handling of domain-based policies instead.

- The format/syntax for domain-based policies is the same. You can add/edit the domain policies to be handled internally using Web UI (it detects wherever domain-based policies can rely on ```dnsmasq``` or not and lets you edit the proper config file accordingly), by editing ```/etc/config/openvpn-policy-routing``` or by running commands like:
```sh
uci add openvpn-policy-routing domain-policy
uci add_list openvpn-policy-routing.@domain-policy[0].ipset='/whatismyip.com/wanroute'
uci add_list openvpn-policy-routing.@domain-policy[0].ipset='/showip.net/tun0route'
uci commit openvpn-policy-routing
```
- By default, with internal handling of domain-based policies, only a first resolved IP address for domain will be used. If you want to ensure reliable operation, please configure openvpn-policy-routing to use full name resolution by running  ```uci set openvpn-policy-routing.@domain-policy[0].resolve=1; uci commit openvpn-policy-routing;```.

- If you want to use internal handling of domains-based policies with full name resolution and *large* lists of domains, please consider installing ```bind-dig``` (it requires ```libopenssl``` which is quite heavy), openvpn-policy-routing will recognize that dig is installed and will use it to speed up name resolution during boot/start/reload.

## Thanks
I'd like to thank everyone who helped create, test and troubleshoot this service. Without contributions from [@hnyman](https://github.com/hnyman), [@dibdot](https://github.com/dibdot), [@danrl](https://github.com/danrl), [@tohojo](https://github.com/tohojo), [@cybrnook](https://github.com/cybrnook), [@nidstigator](https://github.com/nidstigator), [@AndreBL](https://github.com/AndreBL) and rigorous testing by [@dziny](https://github.com/dziny), [@bluenote73](https://github.com/bluenote73) and [@buckaroo](https://github.com/pgera) it wouldn't have been possible.
