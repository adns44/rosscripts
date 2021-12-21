# pppoe-dynamic-ipv6

Because MikroTik does not have native support for Dynamic IPv6 prefixes, you can use this scripts primarily on PPPoE networks, where the IPv6 prefix is dynamic. The prefix length (/48, /56 or /64) does not matter for the script. Clients can get address wit RA on your network.
To use these scripts, do the following
- Install RouterOS v7.
- Add a DHCPv6 client on your PPPoE IF, request prefix and / or address (depends on your ISP)
- Add an address from the pool to your LAN interface.
- On IPv6 ND section set the ra-interval 3m-6m and ra-lifetime to 10m on LAN IF.
- For default prefix, set valid-lifetime 33m and preferred-lifetime 30m.
If you need more help (E.G. firewall config), search specific descriptions for these steps!

This folder contains the following

- ipv6-up.txt
You can run this script after PPPoE connects. It restarts the DHCP v6 client, turns on the IPv6 LAN interface and starts RA. Don't forget to set the variables!

- dynamic-ipv6-firewall.txt
IF you're using IPv6 firewall, and if you want access specific devices from outside, you're seeing the best script! Fill the variables on the top of the script.
In the machines array, you can set a machine name (it will be an IPv6 address-list), and an IPv6 suffix. IMPORTANT! Suffix must be fix. On Windows, turn off randomize identifiers (with Netsh), on Linux add these lines to sysctl.conf
net.ipv6.conf.default.addr_gen_mode = 0
net.ipv6.conf.eth0.addr_gen_mode = 0
(change eth0 if required)
After IPv6 up, this script gets the prefix, and adds the address-lists. you can set rules on firewall for address lists. E.G. drop all new incoming connections, except your home server. it is important if you want protect your devices, because IPv6 does not use NAT by default, so clients are accessible directly from outside.
In some situations the script might not work because ROS does not have native support to strip the prefix only from an IPv6 address.

- ipv6-down.txt
Fill the variables! When PPPoE disconnect, this script can force expire RAs. It is important to the old addresses do not stuck in the devices. so after reconnect, devices can use new IPv6 prefix.

These scripts are tested on DIGI Hungary network, but it might be work on Magyar Telekom.