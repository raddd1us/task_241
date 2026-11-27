# Открываем iptables

1) Установите `iptables`.
2) Посмотрите текущие правила и политики цепочек.
3) Перед изменением firewall убедитесь, что у вас есть текущая рабочая SSH-сессия и дополнительный способ доступа к консоли VM.
4) Настройте правила так, чтобы:
   - loopback-трафик был разрешён;
   - уже установленные соединения продолжали работать;
   - входящий SSH был разрешён;
   - ICMP echo-request был разрешён;
   - остальные новые входящие соединения были запрещены.
5) Проверьте SSH из второй консоли.
6) Какой транспортный протокол использует SSH — TCP или UDP?
7) Временно разрешите дополнительный TCP-порт, проверьте правило и затем удалите его.

# Сохраняем

8) Сохраняются ли созданные вами правила после перезагрузки автоматически?
9) Настройте сохранение правил способом, подходящим для вашей системы.
10) Перезагрузите VM и убедитесь, что нужные правила сохранились.

## Критерий приёмки

В отчёте должны быть:
- исходный список правил и политик iptables;
- итоговый список правил;
- правило для loopback;
- правило для `ESTABLISHED,RELATED`;
- правило разрешения SSH;
- правило разрешения ICMP echo-request;
- политика или правило, запрещающее остальные новые входящие соединения;
- пример временного разрешения дополнительного TCP-порта и его последующего удаления;
- способ сохранения правил;
- вывод после перезагрузки, подтверждающий сохранение конфигурации.


# Григорьев Сергей 241 группа
# Открываем iptables

## 1. Установка iptables

Перед установкой проверено, что пакета нет в системе:

```bash
[student@student-13 ~]$ cat /etc/os-release | head -3
NAME="ALT Linux"
VERSION="20250612"
ID=altlinux
[student@student-13 ~]$ ls -l /sbin/iptables /usr/sbin/iptables 2>&1
ls: cannot access '/sbin/iptables': No such file or directory
ls: cannot access '/usr/sbin/iptables': No such file or directory
[student@student-13 ~]$ rpm -q iptables 2>&1
package iptables is not installed
[student@student-13 ~]$ systemctl list-unit-files 2>&1 | grep -i -E 'iptables|nftables|firewalld'
[student@student-13 ~]$
```

Установка:

```bash
sudo apt-get update
sudo apt-get install -y iptables
```

Проверка после установки:

```bash
[student@student-13 ~]$ rpm -q iptables
iptables-1.8.13-alt1.x86_64
[student@student-13 ~]$ ls -l /sbin/iptables /usr/sbin/iptables 2>&1
lrwxrwxrwx 1 root root 20 Jul 22 05:00 /sbin/iptables -> xtables-legacy-multi
lrwxrwxrwx 1 root root 20 Jul 22 05:00 /usr/sbin/iptables -> xtables-legacy-multi
[student@student-13 ~]$ systemctl list-unit-files 2>&1 | grep -i iptables
iptables.service                             disabled        disabled
```

Пакет `iptables-1.8.13-alt1` установлен, бинарник находится в `/sbin/iptables`. Для обычного пользователя он не в `PATH` (`iptables: command not found`), поэтому команды запускаются через `sudo`. В пакете есть служба `iptables.service`, она пригодится для сохранения правил.

## 2. Текущие правила и политики (до изменений)

```bash
[student@student-13 ~]$ sudo iptables --version
iptables v1.8.13 (legacy)
[student@student-13 ~]$ sudo iptables -S
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
[student@student-13 ~]$ sudo iptables -L -n -v --line-numbers
Chain INPUT (policy ACCEPT 79 packets, 6538 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 70 packets, 6410 bytes)
num   pkts bytes target     prot opt in     out     source               destination
```

Исходное состояние: политики всех цепочек `ACCEPT`, правил нет, то есть пропускается весь трафик.

## 3. Подготовка перед изменением firewall

Порт, на котором слушает SSH:

```bash
[student@student-13 ~]$ sudo ss -tlnp | grep sshd
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=11268,fd=3))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=11268,fd=4))
```

Открыта вторая SSH-сессия в качестве запасной:

```bash
[student@student-13 ~]$ who
student  pts/0        2026-11-27 12:48 (192.168.6.83)
student  pts/1        2026-11-27 12:56 (192.168.6.83)
```

`sshd` слушает порт 22. Две сессии (`pts/0` и `pts/1`) были открыты до изменения правил. Если бы правила отрезали новые подключения, запасная сессия осталась бы рабочей.

## 4. Настройка правил

Сначала добавлены разрешающие правила (политика оставалась `ACCEPT`, доступ не пропадал):

```bash
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
```

Проверка добавленных правил:

```bash
[student@student-13 ~]$ sudo iptables -S
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
```

После этого запрещены все остальные новые входящие соединения: политика цепочки `INPUT` переведена в `DROP`.

```bash
sudo iptables -P INPUT DROP
```

Итоговый список правил:

```bash
[student@student-13 ~]$ sudo iptables -S
-P INPUT DROP
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
```

Назначение правил:

| Правило | Назначение |
|---------|------------|
| `-A INPUT -i lo -j ACCEPT` | разрешает loopback-трафик |
| `-m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT` | пропускает пакеты уже установленных соединений и связанных с ними (в том числе открытые SSH-сессии) |
| `-p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT` | разрешает новые входящие SSH-подключения |
| `-p icmp --icmp-type echo-request -j ACCEPT` | разрешает входящий ping (в выводе `icmp-type 8`) |
| `-P INPUT DROP` | политика: всё остальное входящее отбрасывается |

Правило `ESTABLISHED,RELATED` добавлено до смены политики на `DROP`, поэтому открытые сессии не прервались.

## 5. Проверка SSH из второй консоли

Из нового окна PowerShell выполнено новое подключение (то есть через правило с `ctstate NEW`):

```bash
[student@student-13 ~]$ who
student  pts/0        2026-11-27 12:48 (192.168.6.83)
student  pts/1        2026-11-27 12:56 (192.168.6.83)
student  pts/2        2026-11-27 13:01 (192.168.6.83)
[student@student-13 ~]$ sudo iptables -L INPUT -n -v --line-numbers
Chain INPUT (policy DROP 13 packets, 416 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0
2      890 77029 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
3        2   164 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22 ctstate NEW
4        0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0            icmptype 8
```

Новое SSH-подключение прошло при политике `INPUT DROP`. Счётчик правила 3 (`tcp dpt:22 ctstate NEW`) вырос с 0 до 2, значит, подключение пропустило именно оно. Политика `DROP` отбросила 13 пакетов.

## 6. Транспортный протокол SSH

SSH использует **TCP** (порт 22). Это подтверждается тем, что `ss -tlnp` показал `sshd` в состоянии `LISTEN` на `0.0.0.0:22` (ключ `-t` — TCP), а новое подключение пропустило правило `tcp dpt:22`.

## 7. Временное разрешение дополнительного TCP-порта

Добавление правила для порта 8080:

```bash
[student@student-13 ~]$ sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
[student@student-13 ~]$ sudo iptables -S
-P INPUT DROP
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 8080 -j ACCEPT
[student@student-13 ~]$ sudo iptables -L INPUT -n -v --line-numbers
Chain INPUT (policy DROP 33 packets, 1056 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0
2     1273  109K ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
3        2   164 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22 ctstate NEW
4        0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0            icmptype 8
5        0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8080
```

Правило для `tcp/8080` добавлено пятым, до срабатывания политики `DROP`. Счётчик 0: к порту никто не обращался, на нём ничего не слушает, правило создано только для демонстрации.

Удаление правила:

```bash
[student@student-13 ~]$ sudo iptables -D INPUT -p tcp --dport 8080 -j ACCEPT
[student@student-13 ~]$ sudo iptables -S
-P INPUT DROP
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
```

Временное правило удалено, остались четыре основных правила и политика `INPUT DROP`.

# Сохраняем

## 8. Сохраняются ли правила после перезагрузки автоматически

**Нет.** Правила `iptables` хранятся в памяти ядра и при перезагрузке теряются. В нашей системе это подтверждается состоянием службы и файла правил:

```bash
[student@student-13 ~]$ systemctl cat iptables.service
# /usr/lib/systemd/system/iptables.service
[Unit]
Description=IPv4 firewall with iptables
After=syslog.target
ConditionFileNotEmpty=/etc/sysconfig/iptables

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/etc/init.d/iptables start
ExecStop=/etc/init.d/iptables stop
Environment=BOOTUP=serial
Environment=CONSOLETYPE=serial
StandardOutput=syslog
StandardError=syslog

[Install]
WantedBy=basic.target
[student@student-13 ~]$ systemctl is-enabled iptables.service
disabled
[student@student-13 ~]$ sudo cat /etc/sysconfig/iptables
# iptables firewall config file in iptables-save format.
# DO NOT USE THE -t (table) OPTION IN THIS FILE!
```

Служба `iptables.service` при старте загружает правила из `/etc/sysconfig/iptables`, но она была `disabled`, а сам файл содержал только комментарии. Значит, без дополнительной настройки после перезагрузки правил бы не было.

## 9. Настройка сохранения правил

В скрипте службы есть команда `save`, которая вызывает `iptables-save` и записывает результат в `/etc/sysconfig/iptables`:

```bash
[student@student-13 ~]$ sudo grep -n -i -E 'save|iptables-save' /etc/rc.d/init.d/iptables
20:IPTABLES_SAVE="$IPTABLES-save"
21:IPTABLES_SAVE_ARGS=
23:IPTABLES_SAVE_ON_STOP=no
24:IPTABLES_SAVE_ON_RESTART=no
208:save()
215:               $IPTABLES_SAVE $IPTABLES_SAVE_ARGS >"$tmp"; then
217:                            ln -f "$IPTABLES_DATA" "$IPTABLES_DATA.save"
232:            passed "$IPTABLES save"
278:    save)
279:            save
288:            is_yes "$IPTABLES_SAVE_ON_STOP" && save
292:            is_yes "$IPTABLES_SAVE_ON_RESTART" && save
302:            msg_usage "${0##*/} {condrestart|condstop|panic|restart|save|start|status|stop}"
```

Перед перезаписью сделана резервная копия заводского файла, правила сохранены, служба включена в автозагрузку:

```bash
[student@student-13 ~]$ sudo cp -a /etc/sysconfig/iptables /etc/sysconfig/iptables.orig
[student@student-13 ~]$ sudo /etc/init.d/iptables save
Saving current rules to /etc/sysconfig/iptables:                           [ DONE ]
[student@student-13 ~]$ sudo cat /etc/sysconfig/iptables
# Generated by iptables-save v1.8.13 on Fri Nov 27 13:05:39 2026
*filter
:INPUT DROP [136:9792]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [2823:269469]
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
COMMIT
# Completed on Fri Nov 27 13:05:39 2026
[student@student-13 ~]$ sudo systemctl enable iptables.service
Synchronizing state of iptables.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable iptables
Created symlink '/etc/systemd/system/basic.target.wants/iptables.service' → '/usr/lib/systemd/system/iptables.service'.
[student@student-13 ~]$ systemctl is-enabled iptables.service
enabled
```

Способ сохранения: правила записаны командой `/etc/init.d/iptables save` в файл `/etc/sysconfig/iptables` (формат `iptables-save`), а служба `iptables.service` включена в автозагрузку и при старте системы восстанавливает правила из этого файла.

## 10. Перезагрузка и проверка

```bash
sudo reboot
```

После перезагрузки выполнено новое подключение по SSH и проверено:

```bash
[student@student-13 ~]$ uptime
 13:24:19 up 16 min,  1 user,  load average: 0.07, 0.03, 0.05
[student@student-13 ~]$ systemctl is-enabled iptables.service
enabled
[student@student-13 ~]$ systemctl is-active iptables.service
active
[student@student-13 ~]$ sudo iptables -S
-P INPUT DROP
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
[student@student-13 ~]$ sudo iptables -L INPUT -n -v --line-numbers
Chain INPUT (policy DROP 456 packets, 35119 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0
2      424 37365 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
3        1    60 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22 ctstate NEW
4        0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0            icmptype 8
```

После перезагрузки (`up 16 min`) служба `iptables.service` включена и активна, политика `INPUT DROP` и все четыре правила восстановлены автоматически. Счётчик правила 3 (1 пакет) показывает, что SSH-подключение после перезагрузки прошло через сохранённое правило.
