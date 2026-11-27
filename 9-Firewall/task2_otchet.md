# Открываем firewalld

1) Установите и запустите `firewalld`.
2) Определите активную зону и интерфейсы, относящиеся к ней.
3) Проверьте возможность подключения по SSH.
4) Если SSH не разрешён — добавьте соответствующий сервис в нужную зону.
5) Выведите список разрешённых сервисов и портов с помощью `firewall-cmd`.
6) Можно ли добавлять доступ по названию сервиса, а не только номеру порта?
7) На вашей локальной виртуальной машине подключитесь к Samba-серверу из предыдущего задания.
8) Если firewall блокирует Samba — разрешите нужный сервис.
9) Добавьте один тестовый TCP-порт по номеру, покажите его в конфигурации и удалите.
10) Сделайте требуемые изменения постоянными.
11) Перезагрузите VM или firewalld и убедитесь, что постоянная конфигурация сохранилась.
12) Определите, какой backend использует firewalld в вашей системе.

## Критерий приёмки

В отчёте должны быть:
- активная зона firewalld;
- интерфейс, привязанный к этой зоне;
- список разрешённых сервисов и портов до изменений;
- команда добавления SSH-сервиса;
- команда добавления Samba-сервиса;
- пример добавления TCP-порта по номеру;
- вывод runtime-конфигурации;
- вывод permanent-конфигурации;
- команда применения постоянных изменений;
- вывод после reload или reboot, подтверждающий сохранение правил;
- информация о backend, используемом firewalld.


## Отчёт по практической работе: Открываем firewalld

## 1. Установка и запуск `firewalld`

Состояние перед началом работы:

```bash
rpm -q firewalld iptables nftables samba
systemctl is-active iptables firewalld smb
systemctl is-enabled iptables firewalld
sudo iptables -S
```

```text
package firewalld is not installed
iptables-1.8.13-alt1.x86_64
package nftables is not installed
samba-4.23.13-alt1.x86_64
active
inactive
active
enabled
not-found
-P INPUT DROP
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
```

До установки `firewalld` не было, а служба `iptables` (правила из прошлой работы с политикой INPUT DROP) была активна и включена. Обе службы управляют одним файрволом, поэтому перед переходом сохранены резервные копии правил:

```bash
sudo iptables-save | tee ~/iptables_before_firewalld.rules
sudo cp -a /etc/sysconfig/iptables ~/sysconfig_iptables.bak_fw
```

Установка пакета (в фоне через `nohup`, чтобы обрыв SSH не прервал её):

```bash
sudo apt-get update
sudo sh -c 'nohup apt-get install -y firewalld > /tmp/fw_install.log 2>&1 &'
```

```bash
rpm -q firewalld
```

```text
firewalld-2.5.2-alt1.noarch
```

Перед переключением проверено, что SSH будет разрешён в зоне по умолчанию (чтобы не потерять доступ):

```bash
sudo firewall-offline-cmd --get-default-zone
sudo firewall-offline-cmd --zone=public --list-services
sudo firewall-offline-cmd --zone=public --list-ports
```

```text
public
dhcpv6-client ssh
```

(список портов пуст)

Переключение: `iptables` остановлен и отключён, `firewalld` запущен и включён:

```bash
sudo systemctl disable --now iptables && sudo systemctl enable --now firewalld
sudo firewall-cmd --state
systemctl is-active firewalld; systemctl is-enabled firewalld
systemctl is-active iptables; systemctl is-enabled iptables
```

```text
running
active
enabled
inactive
disabled
```

Пояснение: `firewalld` установлен, работает и включён в автозагрузку. Старая служба `iptables` отключена, чтобы не конфликтовать с ним.

## 2. Активная зона и интерфейсы

```bash
sudo firewall-cmd --get-active-zones
ip -br addr
sudo firewall-cmd --get-zone-of-interface=eth0
sudo firewall-cmd --list-all
```

```text
public (default)
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             192.168.6.113/24 fe80::21:d4ff:fe25:c446/64
no zone
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces:
  sources:
  services: dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

Сразу после запуска `eth0` не был привязан к зоне явно (`no zone`) и обслуживался зоной по умолчанию `public`. Для наглядности привязал его к `public` явно:

```bash
sudo firewall-cmd --zone=public --change-interface=eth0
sudo firewall-cmd --get-zone-of-interface=eth0
sudo firewall-cmd --get-active-zones
```

```text
success
public
public (default)
  interfaces: eth0
```

Пояснение: активная зона — `public`, интерфейс — `eth0` (192.168.6.113/24).

## 3. Проверка возможности подключения по SSH

```bash
sudo ss -tlnp | grep ':22 '
```

```text
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=696,fd=3))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=696,fd=4))
```

Пояснение: `sshd` слушает порт 22. После запуска `firewalld` выполнено новое SSH-подключение со второй консоли (`ssh -i ... lab13`), оно прошло успешно, то есть SSH через файрвол доступен.

## 4. Добавление сервиса SSH

```bash
sudo firewall-cmd --zone=public --add-service=ssh
```

```text
Warning: ALREADY_ENABLED: 'ssh' already in 'public'
success
```

Пояснение: сервис `ssh` уже был разрешён в зоне `public`, поэтому добавлять его не потребовалось.

## 5. Список разрешённых сервисов и портов

```bash
sudo firewall-cmd --zone=public --list-services
sudo firewall-cmd --zone=public --list-ports
```

```text
dhcpv6-client ssh
```

(список портов пуст)

Пояснение: до изменений в зоне `public` разрешены сервисы `dhcpv6-client` и `ssh`, открытых портов нет.

## 6. Можно ли добавлять доступ по названию сервиса

```bash
sudo firewall-cmd --info-service=ssh
sudo firewall-cmd --get-services | tr ' ' '\n' | grep -xE 'ssh|samba|samba-client|http|https'
```

```text
ssh
  ports: 22/tcp
  protocols:
  source-ports:
  modules:
  destination:
  includes:
  helpers:
http
https
samba
samba-client
ssh
```

Пояснение: да, доступ можно открывать по имени сервиса. Сервис в `firewalld` — это именованный набор портов и протоколов (например, `ssh` — это `22/tcp`), поэтому не нужно помнить номера портов.

## 7. Подключение к Samba-серверу (до разрешения сервиса)

Шары на сервере:

```bash
sudo testparm -s 2>/dev/null | grep -E '^\[|path|guest ok|valid users|read only|writable'
sudo ss -tlnp | grep -E ':(139|445) '
```

```text
[global]
[homes]
        read only = No
        valid users = %S %D%w%S
[printers]
        path = /var/tmp
[print$]
        path = /var/lib/samba/drivers
[public]
        guest ok = Yes
        path = /srv/samba/public
[private]
        path = /srv/samba/private
        read only = No
        valid users = smbuser
[team]
        path = /srv/samba/team
        read only = No
        valid users = @smbteam
[school]
        path = /srv/samba/school
        valid users = @teachers @students
LISTEN 0      50           0.0.0.0:139       0.0.0.0:*    users:(("smbd",pid=699,fd=30))
LISTEN 0      50           0.0.0.0:445       0.0.0.0:*    users:(("smbd",pid=699,fd=29))
LISTEN 0      50              [::]:139          [::]:*    users:(("smbd",pid=699,fd=28))
LISTEN 0      50              [::]:445          [::]:*    users:(("smbd",pid=699,fd=27))
```

Отдельной локальной ВМ-клиента нет, а проверка с самого сервера не подходит: трафик к собственному адресу идёт через интерфейс `lo`, который файрвол пропускает. Попытка пробросить порт через jump-хост (`ssh -W 192.168.6.113:445 jump13`) закончилась ошибкой `administratively prohibited`: проброс запрещён на самом jump-хосте, это не результат работы файрвола, поэтому её в проверке не используем.

Поэтому на сервере создан временный сетевой namespace `smbtest`, соединённый с хостом парой `veth`. Запросы из него приходят на сервер через `veth-h` как с внешнего клиента, то есть проходят через `firewalld`:

```bash
sudo ip netns add smbtest
sudo ip link add veth-h type veth peer name veth-c
sudo ip link set veth-c netns smbtest
sudo ip addr add 10.99.0.1/24 dev veth-h && sudo ip link set veth-h up
sudo ip netns exec smbtest ip addr add 10.99.0.2/24 dev veth-c
sudo ip netns exec smbtest ip link set veth-c up
sudo ip netns exec smbtest ip link set lo up
sudo firewall-cmd --get-zone-of-interface=veth-h
for p in 139 445; do sudo ip netns exec smbtest timeout 3 bash -c "</dev/tcp/10.99.0.1/$p" 2>/dev/null && echo "$p open" || echo "$p blocked/timeout"; done
```

```text
no zone
139 blocked/timeout
445 blocked/timeout
```

Пояснение: `smbd` слушает порты 139 и 445, но из внешней сети они недоступны. Интерфейс `veth-h` не привязан к зоне, поэтому его обслуживает зона по умолчанию `public`, где сервиса `samba` нет. Значит, подключение блокирует файрвол.

## 8. Разрешение Samba

```bash
sudo firewall-cmd --info-service=samba
sudo firewall-cmd --zone=public --add-service=samba
sudo firewall-cmd --zone=public --list-services
for p in 139 445; do sudo ip netns exec smbtest timeout 3 bash -c "</dev/tcp/10.99.0.1/$p" 2>/dev/null && echo "$p open" || echo "$p blocked/timeout"; done
```

```text
samba
  ports: 139/tcp 445/tcp
  protocols:
  source-ports:
  modules:
  destination:
  includes: samba-client
  helpers:
success
dhcpv6-client samba ssh
139 open
445 open
```

Подключение к Samba по протоколу SMB (клиент запущен в namespace, то есть через файрвол):

```bash
sudo ip netns exec smbtest smbclient -N -L //10.99.0.1
sudo ip netns exec smbtest smbclient -N //10.99.0.1/public -c 'ls'
```

```text
        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        public          Disk      Public read-only share
        private         Disk      Private share with authentication
        team            Disk      Share for group smbteam
        school          Disk      Teachers (rw) and students (ro)
        IPC$            IPC       IPC Service (Samba 4.23.13-alt1)
        nobody          Disk      Home Directories
SMB1 disabled -- no workgroup available
  .                                   D        0  Thu Nov 26 14:03:48 2026
  ..                                  D        0  Thu Nov 26 14:03:48 2026
  readme.txt                          N       22  Thu Nov 26 14:03:48 2026

                15454048 blocks of size 1024. 13016112 blocks available
```

Пояснение: после добавления сервиса `samba` в зону `public` порты 139 и 445 открылись, подключение по SMB работает и показывает список шар и содержимое гостевой шары `public`.

## 9. Тестовый TCP-порт по номеру

```bash
sudo firewall-cmd --zone=public --add-port=8080/tcp
sudo firewall-cmd --zone=public --list-ports
sudo firewall-cmd --zone=public --list-all
sudo firewall-cmd --zone=public --remove-port=8080/tcp
sudo firewall-cmd --zone=public --list-ports
```

```text
success
8080/tcp
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: dhcpv6-client samba ssh
  ports: 8080/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
success
```

(последний `--list-ports` вернул пустой список)

Пояснение: порт `8080/tcp` добавлен по номеру, отображается в конфигурации зоны и затем удалён.

## 10. Постоянные изменения

Permanent-конфигурация до сохранения:

```bash
sudo firewall-cmd --permanent --zone=public --list-all
```

```text
public (default)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces:
  sources:
  services: dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

Применение постоянных изменений: текущая runtime-конфигурация записана в permanent.

```bash
sudo firewall-cmd --runtime-to-permanent
sudo firewall-cmd --permanent --zone=public --list-all
sudo firewall-cmd --zone=public --list-all
```

```text
success
public (default)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: dhcpv6-client samba ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: dhcpv6-client samba ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

Пояснение: изменения, сделанные без `--permanent`, действуют только до перезагрузки или `reload`. После `--runtime-to-permanent` в permanent появились сервис `samba` и привязка `eth0` к `public`. Тестовый порт 8080 в сохранённую конфигурацию не попал, так как был удалён заранее. Runtime- и permanent-конфигурации совпадают.

## 11. Проверка после перезагрузки конфигурации

Выполнен `firewall-cmd --reload` (сбрасывает runtime и заново загружает permanent). Перезагрузка виртуальной машины не выполнялась.

```bash
sudo firewall-cmd --reload
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --zone=public --list-all
for p in 139 445; do sudo ip netns exec smbtest timeout 3 bash -c "</dev/tcp/10.99.0.1/$p" 2>/dev/null && echo "$p open" || echo "$p blocked/timeout"; done
```

```text
success
public (default)
  interfaces: eth0
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: dhcpv6-client samba ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
139 open
445 open
```

Пояснение: после `reload` сервис `samba` и привязка `eth0` к `public` сохранились, порты Samba по-прежнему доступны. Значит, постоянная конфигурация применилась.

## 12. Backend firewalld

```bash
sudo grep -i '^FirewallBackend' /etc/firewalld/firewalld.conf
rpm -q libnftables1
```

```text
FirewallBackend=nftables
libnftables1-1.1.7-alt2.x86_64
```

Пояснение: `firewalld` использует backend **nftables** (параметр `FirewallBackend=nftables`). Работает он через библиотеку `libnftables1`, утилита `nft` (пакет `nftables`) для этого не требуется и в системе не установлена.

## Очистка тестового окружения

```bash
sudo ip netns del smbtest
sudo ip link del veth-h 2>/dev/null; ip netns list; ip -br link
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --permanent --zone=public --list-services
```

```text
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             02:21:d4:25:c4:46 <BROADCAST,MULTICAST,UP,LOWER_UP>
running
public (default)
  interfaces: eth0
dhcpv6-client samba ssh
```

Временный namespace и интерфейс `veth` удалены, остались только `lo` и `eth0`.
