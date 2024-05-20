# eltexrus
cli 3.3.3.2/30 - isp 1.1.1.1/30 2.2.2.1 3.3.3.1 - hq-r gi1/0/1 1.1.1.2 gi1/0/2 192.168.100.1/26 10.10.10.1/30 - hq-srv 192.168.100.2/26 - br-r 2.2.2.2 172.16.100.1/28 10.10.10.2/30 - br-srv 172.16.100.2/28 - hq-cli dhcp - hq-ad 192.168.100.3/28  
hostnamectl set-hostname BR-SRV - exec bash .
echo 172.16.100.2/28 >\  
  /etc/net/ifaces/ens192/ipv4address - echo default via 172.16.100.1 > /etc/net/ifaces/ens192/ipv4route - echo nameserver 192.168.100.2 > /etc/net/ifaces/ens192/resolf.conf - systemctl restart network - ip a - ping 172.16.100.1 .
nat hq-r - object-group network LOCAL_NET ip address-range 192.168.100.1-192.168.100.62 (172.16.100.1-14) - exit - object-group network PUBLIC_POOL - ip address-range 1.1.1.2 (2.2.2.2) - exit - nat source - pool TRANSLATE_ADDRESS - ip address-range 1.1.1.2 (2.2.2.2) - exit - ruleset SNAT - to interface gi1/0/1 - rule 1 - match source-address LOCAL_NET - action source-nat - pool TRANSLATE_ADDRESS - enable - exit - exit - commit - confirm - ip route 0.0.0.0/0 1.1.1.1 (2.2.2.1) - check ping 8.8.8.8 source ip 192.168.100.1 .
dhcp hq-srv - vim /etc/ifaces/ens192/options (dhcp) -esc-:wq .
dhcp hq-r - ip dhcp-server - ip dhcp-server pool SIMPLE - network 192.168.100.0/26 - address-range 192.168.100.1-192.168.100.62 - excluded-address-range 192.168.100.1 - address 192.168.100.2 mac-address (ip a) - default-router 192.168.100.1 - dns-server 192.168.100.2 - exit . (again ip dhcp server) - show ip dhcp binding .
dhcp hq-srv - systemctl restart network  - ip a .
gre - tunnel gre 10 - ttl ttl 18 - mtu 1426 - ip firewall disable - local address 1.1.1.2 (2.2.2.2) - remote address 2.2.2.2 (1.1.1.2) - ip address 10.10.10.1/30 (2) - enable - exit - show tunnel status (-- ip ospf instance 10 - ip ospf area 1.1.1.1 - ip ospf - enable - exit) .
ospf - router ospf log-adjacency-changes - router ospf 10 - router-id - 1.1.1.1 (2.2.2.2) - area 1.1.1.1 - network 192.168.100.0/26 (172.16.100.0/28) - enable - exit - enable .
account eltex - username admin - password P@ssw0rd - privilege 15 (linux - useradd admin - passwd admin ) .
iperf3 - cli dns 8.8.8.8 - apt-get update - apt-get install perf3 - iperf (-R) -c 3.3.3.1 (cli-isp) .
backup eltex - archive - time-period 1440 - type local - by-commit - count-backup 30 - exit - dir flash:backup/ .
ssh hq-r - object-group service SRV_SSH - port range 2222 - exit - object-group network NETWORK_POOL - ip address-range 192.168.100.2 - exit - nat destination - pool SERVER_POOL - ip address 192.168.100.2 - ip port 22 - exit - ruleset DNAT - from interface gi1/0/1 - rule 1 - match protocol tcp - match destination-address PUBLIC_POOL - match destination-port SRV_SSH - action destination-nat pool SERVER_POOL - enable - exit -exit .
br-srv - ssh@user1.1.1.2 -p 2222
