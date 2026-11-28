# Григорьев Сергей 241 группа
# Сети

1) Выведите список сетевых интерфейсов минимум двумя способами.
2) Определите текущий IPv4-адрес и маску интерфейса.
3) Временно добавьте на сетевой интерфейс второй IPv4-адрес, не удаляя основной.
4) Выведите таблицу маршрутизации.
5) Определите маршрут до указанного преподавателем IP-адреса.
6) Выведите таблицу соседей ARP/neighbor.
7) Определите MAC-адрес вашего шлюза.
8) Что такое IP-адрес?
9) Для чего нужны маршруты?
10) Что такое ARP?
11) Что такое DHCP?
12) Что такое DNS?
13) Как называется один из протоколов синхронизации времени?
14) Что такое широковещательный адрес и зачем он используется?
15) Определите широковещательный адрес вашей подсети.
16) Какие ещё параметры, кроме IP-адреса, обычно задаются сетевому интерфейсу?
17) Что такое маска подсети и зачем она нужна?
18) Определите default gateway и используемые DNS-серверы.
19) Выведите список TCP/UDP-портов, которые слушает ваша VM.
20) Проверьте доступность шлюза, DNS-разрешение имени и соединение с SSH-сервером как три отдельные проверки.

## Критерий приёмки

В отчёте должны быть:
- список сетевых интерфейсов;
- IPv4-адрес, маска и MAC-адрес рабочего интерфейса;
- команда добавления второго IPv4-адреса и вывод после добавления;
- команда удаления дополнительного адреса;
- таблица маршрутизации;
- результат определения маршрута до заданного IP;
- таблица ARP/neighbor;
- MAC-адрес шлюза;
- default gateway;
- используемые DNS-серверы;
- рассчитанный широковещательный адрес подсети;
- список слушающих TCP/UDP-портов;
- отдельные результаты проверки доступности шлюза, DNS-разрешения и SSH-соединения;
- краткий вывод, позволяющий отличить проблему IP-связности от проблемы DNS.


# Отчёт по практической работе: Сети

## 1. Список сетевых интерфейсов (двумя способами)

Использованные команды:

```bash
ip link show
ls /sys/class/net
```

Полученный результат:

```bash
[student@student-13 ~]$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 02:21:d4:25:c4:46 brd ff:ff:ff:ff:ff:ff
[student@student-13 ~]$ ls /sys/class/net
eth0  lo
```

Пояснение: в системе два интерфейса. `lo` — служебный loopback, `eth0` — рабочий интерфейс (состояние UP, MTU 1500, MAC `02:21:d4:25:c4:46`). Оба способа показали одинаковый набор интерфейсов.

## 2. IPv4-адрес и маска интерфейса

Использованные команды:

```bash
ip -4 addr show dev eth0
ip -br addr
```

Полученный результат:

```bash
[student@student-13 ~]$ ip -4 addr show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.6.113/24 brd 192.168.6.255 scope global eth0
       valid_lft forever preferred_lft forever
[student@student-13 ~]$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             192.168.6.113/24 fe80::21:d4ff:fe25:c446/64
```

Пояснение: IPv4-адрес `eth0` — `192.168.6.113`, префикс `/24`, то есть маска `255.255.255.0`. MAC-адрес рабочего интерфейса — `02:21:d4:25:c4:46` (из пункта 1).

## 3. Временный второй IPv4-адрес

Добавление адреса (основной адрес не удаляется):

```bash
sudo ip addr add 10.10.10.13/24 dev eth0
ip -4 addr show dev eth0
```

Полученный результат:

```bash
[student@student-13 ~]$ sudo ip addr add 10.10.10.13/24 dev eth0
[student@student-13 ~]$ ip -4 addr show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.6.113/24 brd 192.168.6.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 10.10.10.13/24 scope global eth0
       valid_lft forever preferred_lft forever
```

Пояснение: на `eth0` одновременно два адреса: основной `192.168.6.113/24` и временный `10.10.10.13/24`. Временный адрес существует только до перезагрузки или ручного удаления.

Удаление дополнительного адреса (выполнено после остальных проверок):

```bash
sudo ip addr del 10.10.10.13/24 dev eth0
ip -4 addr show dev eth0
ip route
```

```bash
[student@student-13 ~]$ sudo ip addr del 10.10.10.13/24 dev eth0
[student@student-13 ~]$ ip -4 addr show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.6.113/24 brd 192.168.6.255 scope global eth0
       valid_lft forever preferred_lft forever
[student@student-13 ~]$ ip route
default via 192.168.6.1 dev eth0 proto static
192.168.6.0/24 dev eth0 proto kernel scope link src 192.168.6.113
```

Пояснение: дополнительный адрес удалён, вместе с ним исчез и маршрут `10.10.10.0/24`. Основной адрес остался без изменений.

## 4. Таблица маршрутизации

Использованная команда:

```bash
ip route
```

Результат (при добавленном втором адресе):

```bash
[student@student-13 ~]$ ip route
default via 192.168.6.1 dev eth0 proto static
10.10.10.0/24 dev eth0 proto kernel scope link src 10.10.10.13
192.168.6.0/24 dev eth0 proto kernel scope link src 192.168.6.113
```

Пояснение:
- `default via 192.168.6.1 dev eth0` — маршрут по умолчанию через шлюз `192.168.6.1`;
- `192.168.6.0/24 dev eth0` — основная подсеть, доступна напрямую;
- `10.10.10.0/24 dev eth0` — маршрут, который ядро создало автоматически после добавления временного адреса (после удаления адреса он пропал, см. пункт 3).

## 5. Маршрут до указанного IP-адреса

Преподаватель конкретный IP-адрес не указал, поэтому для демонстрации выбраны три адреса: внешний `8.8.8.8`, шлюз `192.168.6.1` и адрес из подсети временного адреса `10.10.10.5`.

```bash
ip route get 8.8.8.8
ip route get 192.168.6.1
ip route get 10.10.10.5
```

```bash
[student@student-13 ~]$ ip route get 8.8.8.8
8.8.8.8 via 192.168.6.1 dev eth0 src 192.168.6.113 uid 1000
    cache
[student@student-13 ~]$ ip route get 192.168.6.1
192.168.6.1 dev eth0 src 192.168.6.113 uid 1000
    cache
[student@student-13 ~]$ ip route get 10.10.10.5
10.10.10.5 dev eth0 src 10.10.10.13 uid 1000
    cache
```

Пояснение:
- до `8.8.8.8` пакет идёт через шлюз (`via 192.168.6.1`), источник — `192.168.6.113`;
- до `192.168.6.1` слова `via` нет, шлюз доступен напрямую в локальной подсети;
- до `10.10.10.5` также без `via`, но в качестве источника система выбрала `10.10.10.13` — адрес из подходящей подсети.

## 6. Таблица ARP/neighbor

Использованная команда:

```bash
ip neigh show
```

```bash
[student@student-13 ~]$ ip neigh show
192.168.6.1 dev eth0 lladdr 6e:63:f8:83:fb:47 STALE
192.168.6.104 dev eth0 lladdr 02:cc:7d:64:a7:ea STALE
192.168.6.83 dev eth0 lladdr bc:24:11:38:ef:2a REACHABLE
```

Пояснение: таблица соответствия IP-адресов и MAC-адресов соседей по локальной сети. Состояние `REACHABLE` — запись недавно подтверждена, `STALE` — запись есть, но давно не подтверждалась и будет перепроверена при следующем обмене.

## 7. MAC-адрес шлюза

Использованная команда:

```bash
ip neigh show 192.168.6.1
```

```bash
[student@student-13 ~]$ ip neigh show 192.168.6.1
192.168.6.1 dev eth0 lladdr 6e:63:f8:83:fb:47 STALE
```

MAC-адрес шлюза `192.168.6.1`: **`6e:63:f8:83:fb:47`**.

## 8. Что такое IP-адрес?

IP-адрес — это логический адрес узла в сети на сетевом уровне, по которому пакеты находят получателя. В IPv4 он состоит из 32 бит (например, `192.168.6.113`) и делится на сетевую часть и часть узла с помощью маски подсети.

## 9. Для чего нужны маршруты?

Маршрут сообщает системе, через какой интерфейс и какой шлюз отправлять пакеты для конкретной сети. Если сеть назначения подключена напрямую, пакет отправляется получателю в этой подсети; иначе используется маршрут через шлюз, а при отсутствии более точного — маршрут по умолчанию (`default`).

## 10. Что такое ARP?

ARP (Address Resolution Protocol) — протокол, который определяет MAC-адрес узла по его IPv4-адресу в пределах локальной сети. Узел рассылает широковещательный запрос «у кого этот IP?», владелец адреса отвечает своим MAC-адресом, и результат сохраняется в таблице соседей (пункт 6).

## 11. Что такое DHCP?

DHCP (Dynamic Host Configuration Protocol) — протокол автоматической выдачи сетевых параметров узлам: IP-адреса, маски, шлюза по умолчанию, DNS-серверов и срока аренды адреса. Избавляет от ручной настройки на каждой машине.

## 12. Что такое DNS?

DNS (Domain Name System) — распределённая система, которая преобразует доменные имена (например, `ya.ru`) в IP-адреса и обратно.

## 13. Один из протоколов синхронизации времени

NTP (Network Time Protocol). Его упрощённая версия — SNTP.

## 14. Что такое широковещательный адрес и зачем он используется?

Широковещательный (broadcast) адрес — специальный адрес подсети, пакет на который получают все узлы этой подсети. В нём все биты узловой части равны 1. Используется там, где отправителю неизвестен конкретный получатель, например для ARP-запросов и DHCP-обнаружения.

## 15. Широковещательный адрес подсети

Расчёт для адреса `192.168.6.113/24`:

- маска `/24` = `255.255.255.0`;
- адрес сети: `192.168.6.113 AND 255.255.255.0 = 192.168.6.0`;
- в узловой части (последний октет) все биты ставятся в 1: `11111111` = `255`.

Широковещательный адрес: **`192.168.6.255`**. Это совпадает со значением `brd 192.168.6.255` в выводе `ip -4 addr show dev eth0` (пункт 2).

## 16. Какие ещё параметры, кроме IP-адреса, задаются сетевому интерфейсу?

- маска подсети (префикс);
- шлюз по умолчанию (default gateway);
- DNS-серверы и домены поиска;
- MTU;
- способ получения настроек: статически или по DHCP;
- дополнительные маршруты и их метрики.

## 17. Что такое маска подсети и зачем она нужна?

Маска подсети определяет, какая часть IP-адреса относится к сети, а какая — к узлу. По ней система решает, находится ли получатель в локальной подсети (пакет отправляется напрямую) или за её пределами (пакет отправляется через шлюз). Здесь маска `/24` (`255.255.255.0`): сеть `192.168.6.0`, узлов до `.254`, broadcast `192.168.6.255`.

## 18. Default gateway и используемые DNS-серверы

Использованные команды:

```bash
ip route show default
cat /etc/resolv.conf
```

```bash
[student@student-13 ~]$ ip route show default
default via 192.168.6.1 dev eth0 proto static
[student@student-13 ~]$ cat /etc/resolv.conf
# This is /run/systemd/resolve/resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
#
# This file might be symlinked as /etc/resolv.conf. If you're looking at
# /etc/resolv.conf and seeing this text, you have followed the symlink.
#
# This is a dynamic resolv.conf file for connecting local clients directly to
# all known uplink DNS servers. This file lists all configured search domains.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 192.168.6.1
search .
```

- Default gateway: **`192.168.6.1`** (через `eth0`).
- DNS-сервер: **`192.168.6.1`**. Файл управляется `systemd-resolved`, доменов поиска нет (`search .`).

## 19. Слушающие TCP/UDP-порты

Использованная команда:

```bash
sudo ss -tulpn
```

```bash
[student@student-13 ~]$ sudo ss -tulpn
Netid  State   Recv-Q  Send-Q     Local Address:Port     Peer Address:Port  Process
udp    UNCONN  0       0             127.0.0.54:53            0.0.0.0:*      users:(("systemd-resolve",pid=529,fd=21))
udp    UNCONN  0       0          127.0.0.53%lo:53            0.0.0.0:*      users:(("systemd-resolve",pid=529,fd=19))
udp    UNCONN  0       0           10.10.10.255:137           0.0.0.0:*      users:(("nmbd",pid=656,fd=20))
udp    UNCONN  0       0            10.10.10.13:137           0.0.0.0:*      users:(("nmbd",pid=656,fd=19))
udp    UNCONN  0       0          192.168.6.255:137           0.0.0.0:*      users:(("nmbd",pid=656,fd=15))
udp    UNCONN  0       0          192.168.6.113:137           0.0.0.0:*      users:(("nmbd",pid=656,fd=14))
udp    UNCONN  0       0                0.0.0.0:137           0.0.0.0:*      users:(("nmbd",pid=656,fd=12))
udp    UNCONN  0       0           10.10.10.255:138           0.0.0.0:*      users:(("nmbd",pid=656,fd=22))
udp    UNCONN  0       0            10.10.10.13:138           0.0.0.0:*      users:(("nmbd",pid=656,fd=21))
udp    UNCONN  0       0          192.168.6.255:138           0.0.0.0:*      users:(("nmbd",pid=656,fd=17))
udp    UNCONN  0       0          192.168.6.113:138           0.0.0.0:*      users:(("nmbd",pid=656,fd=16))
udp    UNCONN  0       0                0.0.0.0:138           0.0.0.0:*      users:(("nmbd",pid=656,fd=13))
udp    UNCONN  0       0                0.0.0.0:5355          0.0.0.0:*      users:(("systemd-resolve",pid=529,fd=11))
udp    UNCONN  0       0                   [::]:5355             [::]:*      users:(("systemd-resolve",pid=529,fd=13))
tcp    LISTEN  0       50               0.0.0.0:139           0.0.0.0:*      users:(("smbd",pid=699,fd=30))
tcp    LISTEN  0       4096             0.0.0.0:5355          0.0.0.0:*      users:(("systemd-resolve",pid=529,fd=12))
tcp    LISTEN  0       128              0.0.0.0:22            0.0.0.0:*      users:(("sshd",pid=696,fd=3))
tcp    LISTEN  0       50               0.0.0.0:445           0.0.0.0:*      users:(("smbd",pid=699,fd=29))
tcp    LISTEN  0       4096       127.0.0.53%lo:53            0.0.0.0:*      users:(("systemd-resolve",pid=529,fd=20))
tcp    LISTEN  0       4096          127.0.0.54:53            0.0.0.0:*      users:(("systemd-resolve",pid=529,fd=22))
tcp    LISTEN  0       50                  [::]:139              [::]:*      users:(("smbd",pid=699,fd=28))
tcp    LISTEN  0       4096                [::]:5355             [::]:*      users:(("systemd-resolve",pid=529,fd=14))
tcp    LISTEN  0       128                 [::]:22               [::]:*      users:(("sshd",pid=696,fd=4))
tcp    LISTEN  0       50                  [::]:445              [::]:*      users:(("smbd",pid=699,fd=27))
```

Пояснение (вывод снят, пока временный адрес `10.10.10.13` ещё был добавлен, поэтому `nmbd` слушает и на нём):

| Протокол | Порт | Служба |
|----------|------|--------|
| TCP | 22 | sshd (SSH) |
| TCP | 139, 445 | smbd (Samba/SMB) |
| TCP/UDP | 53 | systemd-resolved (локальный DNS-кэш, только `127.0.0.53` и `127.0.0.54`) |
| TCP/UDP | 5355 | systemd-resolved (LLMNR) |
| UDP | 137, 138 | nmbd (NetBIOS) |

## 20. Три отдельные проверки: шлюз, DNS, SSH

Использованные команды:

```bash
ping -c 3 192.168.6.1
getent hosts ya.ru
timeout 3 bash -c 'exec 3<>/dev/tcp/192.168.6.113/22; head -n1 <&3'
```

```bash
[student@student-13 ~]$ ping -c 3 192.168.6.1
PING 192.168.6.1 (192.168.6.1) 56(84) bytes of data.
64 bytes from 192.168.6.1: icmp_seq=1 ttl=64 time=1.35 ms
64 bytes from 192.168.6.1: icmp_seq=2 ttl=64 time=0.511 ms
64 bytes from 192.168.6.1: icmp_seq=3 ttl=64 time=0.734 ms

--- 192.168.6.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 0.511/0.865/1.350/0.354 ms
[student@student-13 ~]$ getent hosts ya.ru
2a02:6b8::2:242 ya.ru
[student@student-13 ~]$ timeout 3 bash -c 'exec 3<>/dev/tcp/192.168.6.113/22; head -n1 <&3'
SSH-2.0-OpenSSH_9.6
```

Пояснение:
- **доступность шлюза**: получено 3 из 3 пакетов, потерь 0%, среднее время 0,865 мс — IP-связность в локальной сети работает;
- **DNS-разрешение**: имя `ya.ru` разрешилось в `2a02:6b8::2:242` (`getent` вернул IPv6-адрес) — DNS-сервер отвечает;
- **SSH-соединение**: TCP-порт 22 открыт, сервер вернул баннер `SSH-2.0-OpenSSH_9.6`.

## Вывод: как отличить проблему IP-связности от проблемы DNS

Проверки независимы друг от друга, поэтому по ним можно локализовать неисправность:
- если не проходит `ping` по IP-адресу (шлюз или внешний адрес), проблема в IP-связности: адрес, маска, маршрут, шлюз или физическое соединение;
- если `ping` по IP проходит, а имя (`getent hosts ya.ru`) не разрешается, IP-связность в порядке, а проблема в DNS: неверный или недоступный DNS-сервер, неправильный `resolv.conf`;
- если и IP-связность, и DNS работают, но нет соединения с портом (SSH), проблема на уровне службы или фильтрации: служба не запущена, порт закрыт файрволом.

В нашем случае все три проверки успешны, значит IP-связность, DNS и SSH работают.
