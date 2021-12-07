# devops-netology

## Домашнее задание к занятию "3.7. Компьютерные сети, лекция 2"

1) Для просмотра доступных сетевых интерфейсов, на примере Linux, можно использовать команду ip a |awk '/state UP/{print $2}'. В Windows сетевые интерфейсы можно просмотреть в "Центр управления сетями и общим доступом".
```bash
vagrant@vagrant:~$ ip a |awk '/state UP/{print $2}' 
eth0:

```

2) Для распознавания соседа по устройству используются протоколы LLDP и CDP (первый открытый, последний является проприетарный протоколом, разработанным Cisco Systems и поддерживаемый только ими).
Для подключения поддержки LLDP используется пакет lldpd (sudo apt install lldpd). Для включения LLDPD используем systemctl enable lldpd, для запуска сервиса - service lldpd start. 
Для проверки соседей используется команда lldpctl.

```bash
vagrant@vagrant:~$ systemctl enable lldpd
Synchronizing state of lldpd.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable lldpd
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ===
Authentication is required to reload the systemd state.
Authenticating as: vagrant,,, (vagrant)
Password: 
==== AUTHENTICATION COMPLETE ===
vagrant@vagrant:~$ lldpctl
-------------------------------------------------------------------------------
LLDP neighbors:
-------------------------------------------------------------------------------
```

3) Для этого используется технология VLAN. Для добавления поддержки VLAN в Linux используется одноименный пакет VLAN (apt-get install vlan). Настройка интерфейсов может происходить как вручную (используя vconfig)
или при помощи добавления конфигурации подынтерфейса в файл /etc/network/interfaces (на примере Debian-семейства, для CentOS и Gentoo конфигурации немного отличаются) 

```bash
auto vlan1400
iface vlan1400 inet static
        address 192.168.1.1
        netmask 255.255.255.0
        vlan_raw_device eth0
```
Номер 1400 в данном случае указывает на то, какой VLAN ID должен использоваться. vlan_raw_device указывает, на каком сетевом интерфейсе должны создаваться новый интерфейс vlan1400.

4) Существует 7 типов агрегации интерфейсов:

* mode=0 (balance-rr)
Этот режим используется по-умолчанию, если в настройках не указано другое. balance-rr обеспечивает балансировку нагрузки и отказоустойчивость. В данном режиме пакеты отправляются "по кругу" от первого интерфейса к последнему и сначала. Если выходит из строя один из интерфейсов, пакеты отправляются на остальные оставшиеся. При подключении портов к разным коммутаторам, требует их настройки.

* mode=1 (active-backup)
При active-backup один интерфейс работает в активном режиме, остальные в ожидающем. Если активный падает, управление передается одному из ожидающих. Не требует поддержки данной функциональности от коммутатора.

* mode=2 (balance-xor)
Передача пакетов распределяется между объединенными интерфейсами по формуле ((MAC-адрес источника) XOR (MAC-адрес получателя)) % число интерфейсов. Один и тот же интерфейс работает с определённым получателем. Режим даёт балансировку нагрузки и отказоустойчивость.

* mode=3 (broadcast)
Происходит передача во все объединенные интерфейсы, обеспечивая отказоустойчивость.

* mode=4 (802.3ad)
Это динамическое объединение портов. В данном режиме можно получить значительное увеличение пропускной способности как входящего так и исходящего трафика, используя все объединенные интерфейсы. Требует поддержки режима от коммутатора, а так же (иногда) дополнительную настройку коммутатора.

* mode=5 (balance-tlb)
Адаптивная балансировка нагрузки. При balance-tlb входящий трафик получается только активным интерфейсом, исходящий - распределяется в зависимости от текущей загрузки каждого интерфейса. Обеспечивается отказоустойчивость и распределение нагрузки исходящего трафика. Не требует специальной поддержки коммутатора.

* mode=6 (balance-alb)
Адаптивная балансировка нагрузки (более совершенная). Обеспечивает балансировку нагрузки как исходящего (TLB, transmit load balancing), так и входящего трафика (для IPv4 через ARP). Не требует специальной поддержки коммутатором, но требует возможности изменять MAC-адрес устройства.

В Linux объединение интерфейсов делается при помощи модуля bonding утилиты ifenslave. 

Если я правильно понял, опции балансировки это те режимы, которые относятся к балансировке нагрузки? Нигде не нашел отдельных опций. Но, если говорить про режимы (параметр mode), то можно выделить следующие:

* balance-rr или 0 
* balance-xor or 2
* balance-tlb or 5
* balance-alb or 6

Пример конфига:

```bash
        auto bond1
        allow-hotplug bond1
        iface bond1 inet static
        address 10.0.0.11
        netmask 255.255.255.0
        gateway 10.0.0.254
        # определяем подчиненные (объединяемые) интерфейсы
        bond-slaves eth0 eth1
        # задаем тип бондинга
        bond-mode balance-alb
        # интервал проверки линии в миллисекундах
        bond-miimon 100
        # Задержка перед установкой соединения в миллисекундах
        bond-downdelay 200
        # Задержка перед обрывом соединения в миллисекундах
        bond-updelay 200
```

5) Для маски /29 всего доступно 8 адресов и 6 хостов, т.к. нулевой адрес IP резервируется для идентификации подсети, последний — в качестве широковещательного адреса, 
таким образом в реально действующих сетях возможное количество узлов на два меньше количества адресов. 
Из сети с маской /24 можно получить 31 подсеть с маской /29, 254 хоста (256 минус 2 служебных) - количество адресов для /24 и 8 (6 доступных хостов и 2 служебных для каждой подсети) 
для /29, 254/8 ~ 31.

```bash
vagrant@vagrant:~$ ipcalc 10.10.10.0/24 -b -s 6 6
Address:   10.10.10.0           
Netmask:   255.255.255.0 = 24   
Wildcard:  0.0.0.255            
=>
Network:   10.10.10.0/24        
HostMin:   10.10.10.1           
HostMax:   10.10.10.254         
Broadcast: 10.10.10.255         
Hosts/Net: 254                   Class A, Private Internet

1. Requested size: 6 hosts
Netmask:   255.255.255.248 = 29 
Network:   10.10.10.0/29        
HostMin:   10.10.10.1           
HostMax:   10.10.10.6           
Broadcast: 10.10.10.7           
Hosts/Net: 6                     Class A, Private Internet

2. Requested size: 6 hosts
Netmask:   255.255.255.248 = 29 
Network:   10.10.10.8/29        
HostMin:   10.10.10.9           
HostMax:   10.10.10.14          
Broadcast: 10.10.10.15          
Hosts/Net: 6                     Class A, Private Internet

Needed size:  16 addresses.
Used network: 10.10.10.0/28
Unused:
10.10.10.16/28
10.10.10.32/27
10.10.10.64/26
10.10.10.128/25

```

6) Допустимо взять частные IP-адреса из 100.64.0.0/10 Carrier-Grade NAT. Маску возьмем 255.255.255.192, в итоге выбираем сеть 100.64.0.0/26.

```bash
vagrant@vagrant:~$ ipcalc 100.64.0.0/24 -b -s 40 50
Address:   100.64.0.0           
Netmask:   255.255.255.0 = 24   
Wildcard:  0.0.0.255            
=>
Network:   100.64.0.0/24        
HostMin:   100.64.0.1           
HostMax:   100.64.0.254         
Broadcast: 100.64.0.255         
Hosts/Net: 254                   Class A

1. Requested size: 40 hosts
Netmask:   255.255.255.192 = 26 
Network:   100.64.0.0/26        
HostMin:   100.64.0.1           
HostMax:   100.64.0.62          
Broadcast: 100.64.0.63          
Hosts/Net: 62                    Class A

2. Requested size: 50 hosts
Netmask:   255.255.255.192 = 26 
Network:   100.64.0.64/26       
HostMin:   100.64.0.65          
HostMax:   100.64.0.126         
Broadcast: 100.64.0.127         
Hosts/Net: 62                    Class A

Needed size:  128 addresses.
Used network: 100.64.0.0/25
Unused:
100.64.0.128/25
```

7) В Linux и Windows можно проверить ARP таблицу при помощи утилиты arp (для Linux, например art -i eth0, для Windows arp -a)

```bash
vagrant@vagrant:~$ arp -i eth0
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.2.3                 ether   52:54:00:12:35:03   C                     eth0
_gateway                 ether   52:54:00:12:35:02   C                     eth0

```

Для полной очистки кэша нужно использовать ip -statistics neigh flush all

```bash
vagrant@vagrant:~$ sudo ip -statistics neigh flush all

*** Round 1, deleting 2 entries ***
*** Flush is complete after 1 round ***

```
Для удаления кэша для определенного IP-адреса используем флаг -d (например arp -d 192.168.1.1)