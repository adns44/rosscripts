# pppoe-dynamic-ipv6

Compatibility: The script refactored in 2023-11-26 and to work well, the required minimum RouterOS is 7.9
Because MikroTik does not have native support for Dynamic IPv6 prefixes, you can use this scripts primarily on PPPoE networks, where the IPv6 prefix is dynamic. The prefix length (/48, /56 or /64) does not matter for the script. Clients can get address wit RA on your network.
To use these scripts, do the following
- Install RouterOS v7.
- Add a DHCPv6 client on your PPPoE IF, request prefix and / or address (depends on your ISP)
```
/ipv6 dhcp-client
add disabled=no interface=pppoe-if-name pool-name=pppoe-pool request=prefix
```
- Add an address from the pool to your LAN interface.
```
/ipv6 address
add address=::1/64 eui-64=no from-pool=pool1 interface=lan advertise=yes
```
If you turn on IPv6, because no NAT on this type of connection, you need to use a few firewall rules. Whit these, no new connections allowed outside, only your clients can go to the internet (over pppoe1 IF). A minimum of these above.
```
/ipv6 firewall filter
add action=accept chain=forward comment="related,established to clients" \
    connection-state=established,related,untracked
add action=accept chain=forward comment="all out on pppoe" out-interface=\
    pppoe1
add action=accept chain=forward comment="allow ICMPv6 on forward" protocol=\
    icmpv6
add action=accept chain=input comment="related,established to router" \
    connection-state=established,related,untracked
add action=accept chain=input comment="ICMP to router" protocol=icmpv6
add action=accept chain=input comment="DHCPv6 to router" dst-port=546 \
    protocol=udp src-port=547
add action=drop chain=forward
add action=drop chain=input
```

## files
This folder contains the following

- ipv6-up.txt
You can run this script after PPPoE connects. It restarts the DHCP v6 client, turns on the IPv6 LAN interface and starts RA. Don't forget to set the variables!

- dynamic-ipv6-firewall.txt
IF you're using IPv6 firewall, and if you want access specific devices from outside, you're seeing the best script! Fill the variables on the top of the script.
In the machines array, you can set a machine name (it will be an IPv6 address-list), and an IPv6 suffix. IMPORTANT! Suffix must be fix. On Windows, turn off randomize identifiers (with Netsh), on Linux add these lines to sysctl.conf
```
net.ipv6.conf.default.addr_gen_mode = 0
net.ipv6.conf.eth0.addr_gen_mode = 0
```
(change eth0 if required)
After IPv6 up, this script gets the prefix, and adds the address-lists. you can set rules on firewall for address lists. E.G. drop all new incoming connections, except your home server. it is important if you want protect your devices, because IPv6 does not use NAT by default, so clients are accessible directly from outside.
In some situations the script might not work because ROS does not have native support to strip the prefix only from an IPv6 address.
If you want to access client1 and client2 from outside, add these rules to firewall
```
/ipv6 firewall filter
add place-before=3 chain=forward dst-address-list=client1 in-interface=pppoe1
add place-before=3 chain=forward dst-address-list=client2 in-interface=pppoe1
```
If you use my previous rules, place-before places rules in the best position to keep the performance.
Exec this script in the end of up script if you need to use it.

- ipv6-down.txt
Fill the variables! When PPPoE disconnect, this script can force expire RAs. It is important to the old addresses do not stuck in the devices. so after reconnect, devices can use new IPv6 prefix.

## Set router to use scripts
Add the scripts pppoe-up and pppoe-down in system/scripts, add a profile 
```
/ppp/profile
add copy-from=default name=pppoe-custom on-down=pppoe-down on-up=pppoe-up
```
And set your interface to use the pppoe-custom profile in interface/pppoe-client. Finaly do a reconnect.

These scripts are tested on DIGI Hungary network, but it might be work on Magyar Telekom and on others.