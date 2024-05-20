# eltex1.18.1
**IP**  
isp 1.1.1.1/30 (2001:1::1/64) 2.2.2.1/30 (2001:2::1/64) 3.3.3.1/30 (2001:3::1/64)
cli 3.3.3.2/30 (2001:3::2/64)  
hq-r gi1/0/1 1.1.1.2/30 (2001:1::2/64) gi1/0/2 192.168.100.1/26 (2000:192::1/122) tun1 10.10.10.1/30 (2001:10::1/64)  
hq-srv dhcp  
br-r 2.2.2.2/30 (2001:2::2/64) 172.16.100.1/28 (2000:172::1/124) tun1 10.10.10.2/30 (2001:10::2/64)   
br-srv 172.16.100.2/28 (2000:172::2/124)  
hq-cli dhcp - hq-ad 192.168.100.3/28 (2000:192::3/122)  
**BR-SRV**  
hostnamectl set-hostname BR-SRV  
exec bash .echo 172.16.100.2/28 > /etc/net/ifaces/ens192/ipv4address  
echo default via 172.16.100.1 > /etc/net/ifaces/ens192/ipv4route  
echo nameserver 192.168.100.2 > /etc/net/ifaces/ens192/resolf.conf  
systemctl restart network  
ip a - ping 172.16.100.1   
**NAT**   
hq-r - object-group network LOCAL_NET  
ip address-range 192.168.100.1-192.168.100.62 (172.16.100.1-14)  
exit  
object-group network PUBLIC_POOL  
ip address-range 1.1.1.2 (2.2.2.2)  
exit  
nat source  
pool TRANSLATE_ADDRESS  
ip address-range 1.1.1.2 (2.2.2.2)  
exit  
ruleset SNAT   
to interface gi1/0/1   
rule 1  
match source-address LOCAL_NET  
action source-nat pool TRANSLATE_ADDRESS  
enable - exit - exit - commit - confirm - ip route 0.0.0.0/0 1.1.1.1 (2.2.2.1) - check ping 8.8.8.8 source ip 192.168.100.1  
**DHCP**    
hq-srv - vim /etc/ifaces/ens192/options (dhcp) -esc-:wq .
dhcp hq-r   
ip dhcp-server    
ip dhcp-server pool SIMPLE  
network 192.168.100.0/26   
address-range 192.168.100.1-192.168.100.62  
excluded-address-range 192.168.100.1  
address 192.168.100.2 mac-address (ip a на hq-srv посмотреть)  
default-router 192.168.100.1  
dns-server 192.168.100.2  
exit     
(again ip dhcp server) - show ip dhcp binding  
dhcp hq-srv - systemctl restart network  - ip a  
**GRE**     
tunnel gre 10  
ttl 18 - mtu 1426 - ip firewall disable - local address 1.1.1.2 (2.2.2.2) - remote address 2.2.2.2 (1.1.1.2) - ip address 10.10.10.1/30 (2) - enable - exit - show tunnel status (-- ip ospf instance 10 - ip ospf area 1.1.1.1 - ip ospf - enable - exit) .  
**OSPF**  
router ospf log-adjacency-changes - router ospf 10 - router-id - 1.1.1.1 (2.2.2.2) - area 1.1.1.1 - network 192.168.100.0/26 (172.16.100.0/28) - enable - exit - enable   
**USER**   
account eltex - username admin - password P@ssw0rd - privilege 15 (linux - useradd admin - passwd admin )  
**IPERF3**  
iperf3 - cli dns 8.8.8.8 - apt-get update - apt-get install iperf3 - iperf (-R) -c 3.3.3.1 (cli-isp)  
**BACKUP**   
backup eltex - archive - time-period 1440 - type local - by-commit - count-backup 30 - exit - dir flash:backup/  
**SHH**  
hq-r - object-group service SRV_SSH - port range 2222 - exit - object-group network NETWORK_POOL - ip address-range 192.168.100.2 - exit - nat destination - pool SERVER_POOL - ip address 192.168.100.2 - ip port 22 - exit - ruleset DNAT - from interface gi1/0/1 - rule 1 - match protocol tcp - match destination-address PUBLIC_POOL - match destination-port SRV_SSH - action destination-nat pool SERVER_POOL - enable - exit -exit .
br-srv - ssh@user1.1.1.2 -p 2222   
**ACL**   
hq-r - ip access-list extended HQ
rule 1  
action deny  
match protocol tcp  
match source-address 3.3.3.2 255.255.255.252  
match destination-port 2222  
enable   
exit  
rule 2  
action permit  
enable  
exit  
end - commit - confirm

**DNS**  
vim /etc/bind/options.conf  
![image](https://github.com/karterbratihka/eltexrus/assets/154001162/580ef92b-7673-4ddd-8a9d-ac0c7441ffd6)  
systemctl enable --now bind  
vim /etc/bind/local.conf     
![image](https://github.com/karterbratihka/eltexrus/assets/154001162/3ca151d5-cfda-4af8-b488-818fef4bbc8e)   
![image](https://github.com/karterbratihka/eltexrus/assets/154001162/f3ace640-a198-49ce-9943-abcaaf3e1d71)
Прямые зоны   
 ![image](https://github.com/karterbratihka/eltexrus/assets/154001162/605920ce-9c7b-4f7d-a8e5-1eb515371c47)  
![image](https://github.com/karterbratihka/eltexrus/assets/154001162/ba2e4366-3cfb-4e52-b6f8-7f8f69dc6d31)  

Обратные зоны
 
 ![image](https://github.com/karterbratihka/eltexrus/assets/154001162/7557e1a0-593a-4737-a56c-3778ca54ffdd)  
 ![image](https://github.com/karterbratihka/eltexrus/assets/154001162/48a93c3c-4ca2-4c00-a6bc-3ffc52224ed9)     

Выдаём права  
 ![image](https://github.com/karterbratihka/eltexrus/assets/154001162/49e60767-13eb-42d3-ad7a-63b6d55b9911)   
 **NTP**  
 HQ-R  
 ![image](https://github.com/karterbratihka/eltexrus/assets/154001162/93183d5b-6770-406d-959a-9ce1d526b3f4)   
 BR-R  
 ![image](https://github.com/karterbratihka/eltexrus/assets/154001162/bcfd84bf-0bad-4411-b0aa-948605e4ccd1)   
 Linux   
 ![image](https://github.com/karterbratihka/eltexrus/assets/154001162/60295a6e-6c04-40fa-86f5-78e70eba6d8e)  


 



