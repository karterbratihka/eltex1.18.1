# eltexrus
cli 3.3.3.2/30 - isp 1.1.1.1/30 2.2.2.1 3.3.3.1 - hq-r gi1/0/1 1.1.1.2 gi1/0/2 192.168.100.1/26 10.10.10.1/30 - hq-srv 192.168.100.2/26 - br-r 2.2.2.2 172.16.100.1/28 10.10.10.2/30 - br-srv 172.16.100.2/28 - hq-cli dhcp - hq-ad 192.168.100.3/28
hostnamectl set-hostname BR-SRV - exec bach
echo 172.16.100.2/28 > /etc/net/ifaces/ens192/ipv4address - echo default via 172.16.100.1 > /etc/net/ifaces/ens192/ipv4route - echo nameserver 192.168.100.2 > /etc/net/ifaces/ens192/resolf.conf - systemctl restart network - ip a - ping 172.16.100.1
nat hq-r - object-group network LOCAL_NET ip address-range 192.168.100.1-192.168.100.62 - exit - object-group network PUBLIC_POOL - ip address-range 1.1.1.2 - exit - nat source 

