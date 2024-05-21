# eltex1.18.1
**IP**  
isp 1.1.1.1/30 (2001:1::1/64) 2.2.2.1/30 (2001:2::1/64) 3.3.3.1/30 (2001:3::1/64)
cli 3.3.3.2/30 (2001:3::2/64)  
hq-r gi1/0/1 1.1.1.2/30 (2001:1::2/64) gi1/0/2 192.168.100.1/27 (2000:192::1/123) tun1 10.10.10.1/30 (2001:10::1/64)  
hq-srv dhcp  
br-r 2.2.2.2/30 (2001:2::2/64) 172.16.100.1/29 (2000:172::1/125) tun1 10.10.10.2/30 (2001:10::2/64)   
br-srv 172.16.100.2/29 (2000:172::2/125)  
hq-cli dhcp - hq-ad 192.168.100.3/27 (2000:192::3/123)  
**BR-SRV**  
hostnamectl set-hostname BR-SRV  
exec bash .echo 172.16.100.2/29 > /etc/net/ifaces/ens192/ipv4address  
echo default via 172.16.100.1 > /etc/net/ifaces/ens192/ipv4route  
echo nameserver 192.168.100.2 > /etc/net/ifaces/ens192/resolf.conf  
systemctl restart network  
ip a - ping 172.16.100.1   
**NAT**   
hq-r - object-group network LOCAL_NET  
ip address-range 192.168.100.1-192.168.100.30 (172.16.100.1-14)  
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
network 192.168.100.0/27     
address-range 192.168.100.1-192.168.100.30    
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
router ospf log-adjacency-changes - router ospf 10 - router-id - 1.1.1.1 (2.2.2.2) - area 1.1.1.1 - network 192.168.100.0/27 (172.16.100.0/29) - enable - exit - enable   
**USER**   
account eltex - username admin - password P@ssw0rd - privilege 15 (linux - useradd admin - passwd admin )  
**IPERF3**  
iperf3 - cli dns 8.8.8.8 - apt-get update - apt-get install iperf3 - iperf (-R) -c 3.3.3.1 (cli-isp)  
**BACKUP**   
backup eltex - archive - time-period 1440 - type local - by-commit - count-backup 30 - exit - dir flash:backup/  
**SHH**  
hq-r - object-group service SRV_SSH - port range 3035 - exit - object-group network NETWORK_POOL - ip address-range 192.168.100.2 - exit - nat destination - pool SERVER_POOL - ip address 192.168.100.2 - ip port 22 - exit - ruleset DNAT - from interface gi1/0/1 - rule 1 - match protocol tcp - match destination-address PUBLIC_POOL - match destination-port SRV_SSH - action destination-nat pool SERVER_POOL - enable - exit -exit .
br-srv - ssh@user1.1.1.2 -p 3035   
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


**DOCKER**   
control bind-chroot disabled   
grep -**q KRB5RCACHETYPE /etc/sysconfig/bind || echo 'KRB5RCACHETYPE="none"' >> /etc/sysconfig/bind   
grep -q 'bind-dns' /etc/bind/named.conf || echo 'include "/var/lib/samba/bind-dns/named.conf";' >> /etc/bind/named.conf   

Добавить в файл  /etc/bind/options.conf:   
tkey-gssapi-keytab "/var/lib/samba/bind-dns/dns.keytab";   
        minimal-responses yes;     
category lame-servers {null;};   

systemctl stop bind   
rm -f /etc/samba/smb.conf    
rm -rf /var/lib/samba    
rm -rf /var/cache/samba   
mkdir -p /var/lib/samba/sysvol   
Меняем хостнейм   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/db02a5d2-1d99-434b-81aa-f26245ce1bfb)   

![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/96658d6b-8019-4140-b5f0-cc93dca0431d)   

samba-tool domain provision --realm=demo.chrt --domain demo --adminpass='P@ssw0rd' --dns-backend=BIND9_DLZ --server-role=dc   
systemctl restart samba   
systemctl restart bind   


**MOODLE**  

 Меняем интерфейс BR-SRV на  вм нетворк и удаляем маршрут по умолчанию  
 ![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/5fa86441-6495-4b5f-a1d8-b68bb66107c1)   
 Через PuTTY   
apt-get install -y apache2 apache2-base apache2-httpd-prefork apache2-mod_php8.1 apache2-mods   

![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/229a46f4-58f1-4354-b62f-c066f8a1cc73)   

apt-get -y install php8.1 php8.1-curl php8.1-fileinfo php8.1-fpm-fcgi php8.1-gd php8.1-intl php8.1-ldap php8.1-mbstring php8.1-mysqlnd php8.1-mysqlnd-mysqli php8.1-opcache php8.1-soap php8.1-sodium php8.1-xmlreader php8.1-xmlrpc php8.1-zip php8.1-openssl   
apt-get install MySQL-server -y (https://docs.moodle.org/404/en/MySQL)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/d2bc7153-b610-4084-90ac-e8ef2bf741b5)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/12a055d2-e519-472f-a9f7-9095edc0f1f1)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/c28d3074-146e-4a4c-bfb9-7e679674ad2e)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/7af7c0b3-afe1-4520-b93f-988e60533dcb)  
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/5e021e8c-2efa-45be-9bac-28a6d0caf7bd)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/9da62022-038d-4ea5-b307-c2e36dd616a4)  
Скачать этот пакет через wget     
https://download.moodle.org/download.php/direct/stable404/moodle-4.4.tgz   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/03d02064-ca7a-48ab-a340-e88e3a7143ef)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/caae0063-4f42-48e7-807a-ce00f5464bd5)  
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/9dfcf851-8449-406a-88b0-615045d9bbc0)  
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/f17360e7-cf82-47c7-8eb0-b6e2403bf5af)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/c5a36801-68a8-4dac-85b8-067102c5cc51)  
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/64e8759c-b7f0-4050-b68e-d7df1f9390ef)  
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/ef229343-64ac-4ff5-b4b2-452bf43c6053)    
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/c3e5b8a3-cfec-4d50-9f45-bdf142b024d4)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/7053d6d7-2364-4af4-895f-5266cf0c1ad1)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/f8cfcf31-eb5f-4e61-8c1c-56fe5d962b42)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/6014e44a-422e-4f9a-b44d-3537675607da)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/0b3bc2a7-a4bd-4770-8e70-722a50209cde)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/a26683df-4236-480e-a01a-68ab27273e45)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/a7d07b5b-7ced-4e41-9993-941e48c486e5)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/cf1011fd-a329-48a2-9466-e1f135866d8f)   
Идём на HQ-CLI и нажимаем обновить   
Добавить пользователей - Администрирование - Добавить пользователей   
Добавить группы - Администрирование - Глобальные группы   
Для исправления ошибки с докером   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/a2f4b713-0ea8-4c07-ba78-7c2a0947bb31)   




![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/f50eb250-eb10-4c42-8466-47bc4b59699b)   
![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/7f22f571-1318-446a-9e4b-586b2b85e0ba)  
2. Выбор OSPF в качестве протокола динамической маршрутизации для растущей сети обоснован его способностью эффективно управлять большими и сложными сетями, обеспечивая быструю сходимость, гибкость в использовании адресного пространства, поддержку различных топологий и устройств от разных производителей. Эти преимущества делают OSPF отличным выбором для сетей, которые планируется масштабировать в будущем.    

5.   
 ![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/bb91ecb9-8d6b-4488-a700-63d84bab17ec)   
 

В приведённом скриншоте показан результат работы утилиты iperf3, которая используется для измерения пропускной способности сети. Команда была выполнена на локальной машине CLI (адрес 3.3.3.2), соединённой с удалённым сервером ISP (адрес 3.3.3.1) по порту 5201.   

Описание результатов:   
-	Общий объём переданных данных за 10 секунд составил 8.29 Гбайт.  
-	Средняя пропускная способность составила 7.12 Гбит/сек.  

 ![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/92ddd611-b2d9-4d78-bd76-300ac126b318)    


На предоставленном скриншоте показан результат работы утилиты iperf3 в режиме реверса (-R), что означает, что сервер ISP (хост 3.3.3.1) отправляет данные клиенту CLI (хост 3.3.3.2). Это позволяет измерить пропускную способность канала от сервера к клиенту.    

Описание результатов:   
-	Общий объём переданных данных за 10 секунд составил 18.1 Гбайт.   
-	Средняя пропускная способность составила 15.5 Гбит/сек.    

6. Резервное копирование выполняется ежедневно, а также по команде COMMIT.    
HQ-R
 ![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/5fc3034e-74db-446b-afe6-8e101845ff4d)   


BR-R
 ![image](https://github.com/karterbratihka/eltex1.18.1/assets/154001162/4494f10e-6a18-4148-adba-b57af6c02f5d)   


Модуль 2   

3. Выбор Samba DC в качестве контроллера домена представляет собой рациональное и экономически выгодное решение, которое обеспечивает высокую совместимость, безопасность и гибкость. Это позволяет организациям эффективно управлять своими сетевыми ресурсами без значительных затрат на программное обеспечение и лицензирование.   
  
4. Использование файлового сервера на базе SMB является оптимальным решением для большинства корпоративных и домашних сетей благодаря его совместимости, простоте настройки, высоким показателям безопасности и производительности. Эти преимущества делают SMB подходящим выбором для эффективного и безопасного обмена файлами в разнообразных сетевых средах.   






 
  














 



