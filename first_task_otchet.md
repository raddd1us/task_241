# Григорьев Сергей 241 группа
# Первое самостоятельное задание с пары

1) Установить пакеты `samba`, `bind`, `bind-utils` и любой Python-модуль.

2) Для каждого установленного пакета вывести:
   2.1) версию;
   2.2) список файлов, принадлежащих пакету;
   2.3) зависимости пакета.

3) Проверить службы, относящиеся к установленным пакетам. Если нужная служба не запущена — запустить её и добавить в автозагрузку.
   3.1) Можно ли включить службу и добавить её в автозагрузку одной командой?

4) Найти и вывести конфигурационные файлы пакетов, у которых они есть.

5) Для одной из служб:
   5.1) показать её текущий статус;
   5.2) показать последние записи из журнала;
   5.3) определить, под каким пользователем она работает.

## Критерий приёмки

В отчёте должны быть:
- перечень установленных пакетов;
- команды и вывод, подтверждающие версии пакетов;
- команды и вывод со списком файлов, принадлежащих пакетам;
- команды и вывод с зависимостями пакетов;
- статус соответствующих служб до и после запуска;
- подтверждение включения служб в автозагрузку;
- путь и фрагмент конфигурационного файла хотя бы одной службы;
- фрагмент журнала выбранной службы;
- краткий вывод о различии между установкой пакета, запуском службы и включением автозапуска.


# Отчёт по практической работе: Первое самостоятельное задание с пары

## 1. Установка пакетов `samba`, `bind`, `bind-utils` и Python-модуля

Сначала проверим, какие из пакетов уже установлены и какие версии доступны в репозитории:

```bash
[student@student-13 ~]$ rpm -q samba bind bind-utils
samba-4.23.13-alt1.x86_64
package bind is not installed
package bind-utils is not installed
[student@student-13 ~]$ apt-cache policy samba bind bind-utils | grep -E '^[a-z]|Candidate|Installed'
samba:
  Installed: 4.23.13-alt1:sisyphus+408845.164.18.2@1794596972
  Candidate: 4.23.13-alt1:sisyphus+408845.164.18.2@1794596972
bind:
  Installed: (none)
  Candidate: 9.18.50-alt3.8c198236bc:sisyphus+438574.100.1.1@1793022303
bind-utils:
  Installed: (none)
  Candidate: 9.18.50-alt3.8c198236bc:sisyphus+438574.100.1.1@1793022303
[student@student-13 ~]$ apt-cache search python3-module-requests
python3-module-requests+socks - Extra "socks" for python3-module-requests
python3-module-requests - HTTP library, written in Python, for human beings
...
```

(в выводе `apt-cache search` показаны первые строки.)

Результат: `samba` уже установлен, `bind` и `bind-utils` отсутствуют. В качестве Python-модуля выбран `python3-module-requests` (HTTP-библиотека).

Установка выполнена в фоне через `nohup`, чтобы обрыв SSH-соединения не прервал установку:

```bash
[student@student-13 ~]$ nohup sudo apt-get install -y bind bind-utils python3-module-requests > ~/install_pkgs.log 2>&1 &
[student@student-13 ~]$ tail -n 30 ~/install_pkgs.log
The following packages will be upgraded
  python3-module-requests
The following NEW packages will be installed:
  bind  bind-control  bind-utils  libbind  libjemalloc2  libnghttp2  libuv
1 upgraded, 7 newly installed, 0 removed and 160 not upgraded.
Need to get 2283kB of archives.
After unpacking 6988kB of additional disk space will be used.
Get:1 http://ftp.altlinux.org Sisyphus/noarch/classic bind-control 1.3-alt1:sisyphus+252433.440.3.1@1591341818 [3431B]
Get:2 http://ftp.altlinux.org Sisyphus/x86_64/classic libjemalloc2 5.4.0-alt1:sisyphus+434807.100.1.1@1790696435 [191kB]
Get:3 http://ftp.altlinux.org Sisyphus/x86_64/classic libnghttp2 1.70.0-alt1:sisyphus+428572.100.1.1@1786002363 [63.7kB]
Get:4 http://ftp.altlinux.org Sisyphus/x86_64/classic libuv 1.53.0-alt1:sisyphus+435668.40.3.1@1791320248 [80.6kB]
Get:5 http://ftp.altlinux.org Sisyphus/x86_64/classic libbind 9.18.50-alt3.8c198236bc:sisyphus+438574.100.1.1@1793022303 [1202kB]
Get:6 http://ftp.altlinux.org Sisyphus/x86_64/classic bind-utils 9.18.50-alt3.8c198236bc:sisyphus+438574.100.1.1@1793022303 [186kB]
Get:7 http://ftp.altlinux.org Sisyphus/noarch/classic python3-module-requests 2.34.2-alt2:sisyphus+439475.100.1.1@1793608601 [160kB]
Get:8 http://ftp.altlinux.org Sisyphus/x86_64/classic bind 9.18.50-alt3.8c198236bc:sisyphus+438574.100.1.1@1793022303 [396kB]
Fetched 2283kB in 0s (4939kB/s)
Committing changes...
Preparing...                            ################################# [100%]
Updating / installing...
1: libnghttp2-1.70.0-alt1               ################################# [ 11%]
2: libjemalloc2-5.4.0-alt1              ################################# [ 22%]
3: libuv-1.53.0-alt1                    ################################# [ 33%]
4: libbind-9.18.50-alt3.8c198236bc      ################################# [ 44%]
5: bind-control-1.3-alt1                ################################# [ 56%]
6: bind-9.18.50-alt3.8c198236bc         ################################# [ 67%]
7: bind-utils-9.18.50-alt3.8c198236bc   ################################# [ 78%]
8: python3-module-requests-2.34.2-alt2  ################################# [ 89%]
Cleaning up / removing...
9: python3-module-requests-2.32.5-alt1  ################################# [100%]
Done.
```

Проверка результата:

```bash
[student@student-13 ~]$ rpm -q samba bind bind-utils python3-module-requests
samba-4.23.13-alt1.x86_64
bind-9.18.50-alt3.8c198236bc.x86_64
bind-utils-9.18.50-alt3.8c198236bc.x86_64
python3-module-requests-2.34.2-alt2.noarch
```

**Перечень установленных пакетов:**

| Пакет | Состояние |
|-------|-----------|
| `samba` | уже был установлен до работы, версия не менялась |
| `bind` | установлен (вместе с `bind-control`, `libbind`, `libjemalloc2`, `libnghttp2`, `libuv`) |
| `bind-utils` | установлен |
| `python3-module-requests` | в системе уже была версия 2.32.5, обновлён до 2.34.2 |

## 2. Версия, файлы и зависимости пакетов

### 2.1. Версии пакетов

```bash
[student@student-13 ~]$ rpm -q --qf '%{NAME} %{VERSION}-%{RELEASE}\n' samba bind bind-utils python3-module-requests
samba 4.23.13-alt1
bind 9.18.50-alt3.8c198236bc
bind-utils 9.18.50-alt3.8c198236bc
python3-module-requests 2.34.2-alt2
```

### 2.2. Список файлов, принадлежащих пакетам

Для каждого пакета выведено количество файлов (`rpm -ql ... | wc -l`) и первые 15 строк списка:

```bash
[student@student-13 ~]$ for p in samba bind bind-utils python3-module-requests; do echo "== $p: файлов $(rpm -ql $p | wc -l)"; rpm -ql $p | head -n 15; done
== samba: файлов 92
/etc/alternatives/packages.d/samba-mit
/etc/rc.d/init.d/nmb
/etc/rc.d/init.d/smb
/etc/samba/smbusers
/usr/lib/samba/samba-bgqd
/usr/lib/systemd/system/nmb.service
/usr/lib/systemd/system/samba-bgqd.service
/usr/lib/systemd/system/smb.service
/usr/lib64/samba/auth
/usr/lib64/samba/auth/unix.so
/usr/lib64/samba/sbin/eventlogadm
/usr/lib64/samba/sbin/nmbd
/usr/lib64/samba/sbin/smbd
/usr/lib64/samba/vfs
/usr/lib64/samba/vfs/acl_tdb.so
== bind: файлов 91
/etc/bind
/etc/bind.keys
/etc/named.conf
/etc/rc.d/init.d/bind
/etc/rndc.conf
/etc/sysconfig/bind
/etc/syslog.d/bind
/run/named
/usr/bin/arpaname
/usr/bin/dnssec-cds
/usr/bin/dnssec-dsfromkey
/usr/bin/dnssec-importkey
/usr/bin/dnssec-keyfromlabel
/usr/bin/dnssec-keygen
/usr/bin/dnssec-revoke
== bind-utils: файлов 12
/usr/bin/delv
/usr/bin/dig
/usr/bin/host
/usr/bin/mdig
/usr/bin/nslookup
/usr/bin/nsupdate
/usr/share/man/man1/delv.1.xz
/usr/share/man/man1/dig.1.xz
/usr/share/man/man1/host.1.xz
/usr/share/man/man1/mdig.1.xz
/usr/share/man/man1/nslookup.1.xz
/usr/share/man/man1/nsupdate.1.xz
== python3-module-requests: файлов 81
/usr/lib/python3/site-packages/requests
/usr/lib/python3/site-packages/requests-2.34.2.dist-info
/usr/lib/python3/site-packages/requests-2.34.2.dist-info/METADATA
/usr/lib/python3/site-packages/requests/__init__.py
/usr/lib/python3/site-packages/requests/__pycache__
/usr/lib/python3/site-packages/requests/__pycache__/__init__.cpython-314.opt-1.pyc
/usr/lib/python3/site-packages/requests/__pycache__/__init__.cpython-314.opt-2.pyc
/usr/lib/python3/site-packages/requests/__pycache__/__init__.cpython-314.pyc
/usr/lib/python3/site-packages/requests/__pycache__/__version__.cpython-314.opt-1.pyc
/usr/lib/python3/site-packages/requests/__pycache__/__version__.cpython-314.opt-2.pyc
/usr/lib/python3/site-packages/requests/__pycache__/__version__.cpython-314.pyc
/usr/lib/python3/site-packages/requests/__pycache__/_internal_utils.cpython-314.opt-1.pyc
/usr/lib/python3/site-packages/requests/__pycache__/_internal_utils.cpython-314.opt-2.pyc
/usr/lib/python3/site-packages/requests/__pycache__/_internal_utils.cpython-314.pyc
/usr/lib/python3/site-packages/requests/__pycache__/_types.cpython-314.opt-1.pyc
```

| Пакет | Файлов | Что входит |
|-------|--------|------------|
| `samba` | 92 | юниты `smb.service`, `nmb.service`, `samba-bgqd.service`, демоны `smbd` и `nmbd`, `/etc/samba/smbusers` |
| `bind` | 91 | сервер имён, `/etc/named.conf`, `/etc/bind`, `/etc/sysconfig/bind`, init-скрипт `/etc/rc.d/init.d/bind`, утилиты `dnssec-*` |
| `bind-utils` | 12 | клиентские утилиты `dig`, `host`, `nslookup`, `nsupdate`, `delv`, `mdig` и их man-страницы; службы нет |
| `python3-module-requests` | 81 | модуль в `/usr/lib/python3/site-packages/requests`; службы нет |

### 2.3. Зависимости пакетов

Для каждого пакета выведено количество зависимостей (`rpm -qR ... | wc -l`) и первые 15 строк:

```bash
[student@student-13 ~]$ for p in samba bind bind-utils python3-module-requests; do echo "== $p: зависимостей $(rpm -qR $p | wc -l)"; rpm -qR $p | head -n 15; done
== samba: зависимостей 25
tdb-utils
/bin/sh
/bin/kill
/bin/sh
/etc/alternatives/packages.d
/etc/rc.d/init.d
/etc/rc.d/init.d(SourceIfNotEmpty)
/etc/rc.d/init.d(is_yes)
/etc/rc.d/init.d(msg_reloading)
/etc/rc.d/init.d(msg_starting)
/etc/rc.d/init.d(msg_stopping)
/etc/rc.d/init.d(msg_usage)
/etc/rc.d/init.d(start_daemon)
/etc/rc.d/init.d(status)
/etc/rc.d/init.d(stop_daemon)
== bind: зависимостей 42
bind-control >= 1.3
/bin/sh
/bin/sh
/etc/rc.d/init.d
/etc/rc.d/init.d(SourceIfNotEmpty)
/etc/rc.d/init.d(action)
/etc/rc.d/init.d(failure)
/etc/rc.d/init.d(is_yes)
/etc/rc.d/init.d(msg_reloading)
/etc/rc.d/init.d(msg_usage)
/etc/rc.d/init.d(start_daemon)
/etc/rc.d/init.d(status)
/etc/rc.d/init.d(stop_daemon)
/etc/rc.d/init.d(success)
/etc/rc.d/init.d/functions
== bind-utils: зависимостей 6
/lib64/ld-linux-x86-64.so.2
libidn2.so.0()(64bit) >= set:ifaF4MxNtNuGyt21
libidn2.so.0(IDN2_0.0.0)(64bit)
libkrb5.so.3()(64bit) >= set:nk6RMYKtexZFP7MfOcRD55Qc
libbind = 9.18.50-alt3.8c198236bc:sisyphus+438574.100.1.1
rpmlib(PayloadIsLzma)
== python3-module-requests: зависимостей 6
python3-module-charset-normalizer
python3-module-idna
python3-module-urllib3
rpmlib(FileDigests) <= 4.6.0-1
/usr/lib/python3/site-packages
rpmlib(PayloadIsLzma)
```

У `bind` много служебных зависимостей от функций init-скриптов, поэтому дополнительно выведены только его пакетные и библиотечные зависимости (без путей и `rpmlib`):

```bash
[student@student-13 ~]$ rpm -qR bind | grep -v -E '^(/|rpmlib)' | sort -u
bind-control >= 1.3
chkconfig
coreutils
grep
libbind = 9.18.50-alt3.8c198236bc:sisyphus+438574.100.1.1
libcap.so.2()(64bit) >= set:kgzEnqa243uKaZoevXMvRH1
libcap.so.2(LIBCAP_1.10)(64bit)
libcap.so.2(LIBCAP_2.21)(64bit)
libcrypto.so.3()(64bit) >= set:qpdXyh
libjemalloc.so.2()(64bit) >= set:jgo00eJYaC870
libjson-c.so.5()(64bit) >= set:lhaJLNNHpDhhnQpG3JrrIq7k7gBMslQ
libnghttp2.so.14()(64bit) >= set:lk6bf0
sed
service
```

| Пакет | Зависимостей | Ключевые |
|-------|--------------|----------|
| `samba` | 25 | `tdb-utils`, `/bin/sh`, `/bin/kill`, функции init-скриптов |
| `bind` | 42 | `bind-control >= 1.3`, `libbind` той же версии, `libcap`, `libcrypto`, `libjemalloc`, `libjson-c`, `libnghttp2`, `chkconfig`, `service` |
| `bind-utils` | 6 | `libbind` той же версии, `libidn2`, `libkrb5`, `ld-linux-x86-64.so.2` |
| `python3-module-requests` | 6 | `python3-module-charset-normalizer`, `python3-module-idna`, `python3-module-urllib3` |

## 3. Службы установленных пакетов

### Состояние до запуска

Какие юниты принесли пакеты и в каком они состоянии (в `systemctl list-unit-files` первая колонка — состояние, вторая — preset):

```bash
[student@student-13 ~]$ systemctl list-unit-files | grep -E 'smb|nmb|samba|bind|named'
bind.service                                 disabled        disabled
nmb.service                                  enabled         disabled
samba-bgqd.service                           disabled        disabled
smb.service                                  enabled         disabled
systemd-hostnamed.service                    static          -
systemd-hostnamed.socket                     static          -
rpcbind.target                               static          -
[student@student-13 ~]$ for u in smb nmb bind; do echo "== $u: is-active=$(systemctl is-active $u) is-enabled=$(systemctl is-enabled $u)"; done
== smb: is-active=active is-enabled=enabled
== nmb: is-active=active is-enabled=enabled
== bind: is-active=inactive is-enabled=disabled
```

Службы `smb` и `nmb` (пакет `samba`) уже работали и были включены в автозагрузку. Служба `bind` после установки пакета оказалась остановленной и не включённой в автозагрузку. `samba-bgqd.service` — вспомогательная служба Samba, отдельно не включалась. У пакетов `bind-utils` и `python3-module-requests` служб нет.

### Запуск `bind` и включение в автозагрузку (п. 3.1)

Служба запущена и добавлена в автозагрузку одной командой:

```bash
[student@student-13 ~]$ sudo systemctl enable --now bind
```

Состояние после:

```bash
[student@student-13 ~]$ systemctl status bind --no-pager
● bind.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/bind.service; enabled; preset: disabled)
     Active: active (running) since Sat 2026-11-28 09:52:04 UTC; 4s ago
 Invocation: fc12ff49fd944d27a0e8c367ca4e3f43
    Process: 7027 ExecStartPre=/etc/init.d/bind rndc_keygen (code=exited, status=0/SUCCESS)
    Process: 7033 ExecStartPre=/usr/bin/named-checkconf $CHROOT -z /etc/named.conf (code=exited, status=0/SUCCESS)
    Process: 7035 ExecStart=/usr/sbin/named -u named $CHROOT $RETAIN_CAPS $EXTRAOPTIONS (code=exited, status=0/SUCCESS)
      Tasks: 4 (limit: 1728)
     Memory: 30M (peak: 30.2M)
        CPU: 119ms
     CGroup: /system.slice/bind.service
             └─7036 /usr/sbin/named -u named
[student@student-13 ~]$ echo "is-active=$(systemctl is-active bind) is-enabled=$(systemctl is-enabled bind)"
is-active=active is-enabled=enabled
```

(из вывода `status` убраны последние строки журнала; они приведены в разделе 5.2.)

Итоговая проверка всех трёх служб после запуска:

```bash
[student@student-13 ~]$ for u in smb nmb bind; do echo "== $u: is-active=$(systemctl is-active $u) is-enabled=$(systemctl is-enabled $u)"; done
== smb: is-active=active is-enabled=enabled
== nmb: is-active=active is-enabled=enabled
== bind: is-active=active is-enabled=enabled
```

| Служба | До запуска | После запуска |
|--------|------------|---------------|
| `smb` | active, enabled | active, enabled |
| `nmb` | active, enabled | active, enabled |
| `bind` | inactive, disabled | active, enabled |

**Ответ на вопрос 3.1:** да, включить службу и добавить её в автозагрузку можно одной командой — `systemctl enable --now <служба>`. Ключ `enable` создаёт автозапуск при загрузке системы, а `--now` сразу запускает службу.

Перед стартом `bind` systemd выполнил проверки `rndc_keygen` и `named-checkconf` (обе завершились с кодом 0), затем запустил `named` от пользователя `named`.

## 4. Конфигурационные файлы пакетов

Файлы, помеченные в пакетах как конфигурационные:

```bash
[student@student-13 ~]$ for p in samba bind bind-utils python3-module-requests; do echo "== $p"; rpm -qc $p; done
== samba
/etc/samba/smbusers
== bind
/etc/rc.d/init.d/bind
/etc/rndc.conf
/etc/sysconfig/bind
/var/lib/bind/etc/local.conf
/var/lib/bind/etc/named.conf
/var/lib/bind/etc/options.conf
/var/lib/bind/etc/rfc1912.conf
/var/lib/bind/etc/rfc1918.conf
/var/lib/bind/etc/rndc.conf
/var/lib/bind/etc/rndc.key
/var/lib/bind/zone/127.in-addr.arpa
/var/lib/bind/zone/empty
/var/lib/bind/zone/localdomain
/var/lib/bind/zone/localhost
== bind-utils
== python3-module-requests
```

У `bind-utils` и `python3-module-requests` конфигурационных файлов нет.

Главные конфигурационные файлы служб:

```bash
[student@student-13 ~]$ ls -l /etc/named.conf /etc/samba/smb.conf
lrwxrwxrwx 1 root root   15 Oct 26 13:39 /etc/named.conf -> bind/named.conf
-rw-r--r-- 1 root root 1844 Nov 26 14:05 /etc/samba/smb.conf
[student@student-13 ~]$ rpm -qf /etc/named.conf /etc/samba/smb.conf
bind-9.18.50-alt3.8c198236bc.x86_64
samba-common-client-4.23.13-alt1.noarch
```

`/etc/named.conf` — символическая ссылка на `bind/named.conf`, принадлежит пакету `bind`. `/etc/samba/smb.conf` принадлежит не пакету `samba`, а пакету `samba-common-client`, поэтому `rpm -qc samba` его не показал.

**Фрагмент `/etc/samba/smb.conf`** (без комментариев и пустых строк, первые 20 строк):

```bash
[student@student-13 ~]$ grep -v -E '^\s*(//|#|;|$)' /etc/samba/smb.conf | head -n 20
[global]
        workgroup = SAMBA
        security = user
        map to guest = Bad User
        passdb backend = tdbsam
        printing = cups
        printcap name = cups
        load printers = yes
        cups options = raw
        include = /etc/samba/usershares.conf
[homes]
        comment = Home Directories
        valid users = %S, %D%w%S
        browseable = no
        read only = no
        inherit acls = yes
[printers]
        comment = All Printers
        path = /var/tmp
        printable = yes
```

**Фрагмент `/etc/named.conf`.** Без `sudo` файл обычному пользователю прочитать нельзя (`grep: /etc/named.conf: Permission denied`), поэтому команда выполнена с `sudo`:

```bash
[student@student-13 ~]$ sudo grep -v -E '^\s*(//|#|;|$)' /etc/named.conf | head -n 20
include "/etc/bind/options.conf";
include "/etc/bind/rndc.conf";
include "/etc/bind/local.conf";
```

Главный конфиг `bind` состоит из трёх подключаемых файлов в `/etc/bind/`.

## 5. Служба `bind`: статус, журнал, пользователь

Для п. 5 выбрана служба `bind.service`.

### 5.1. Текущий статус

```bash
[student@student-13 ~]$ systemctl status bind --no-pager -l
● bind.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/bind.service; enabled; preset: disabled)
     Active: active (running) since Sat 2026-11-28 09:52:04 UTC; 2min 48s ago
 Invocation: fc12ff49fd944d27a0e8c367ca4e3f43
    Process: 7027 ExecStartPre=/etc/init.d/bind rndc_keygen (code=exited, status=0/SUCCESS)
    Process: 7033 ExecStartPre=/usr/bin/named-checkconf $CHROOT -z /etc/named.conf (code=exited, status=0/SUCCESS)
    Process: 7035 ExecStart=/usr/sbin/named -u named $CHROOT $RETAIN_CAPS $EXTRAOPTIONS (code=exited, status=0/SUCCESS)
      Tasks: 5 (limit: 1728)
     Memory: 28.1M (peak: 32M)
        CPU: 120ms
     CGroup: /system.slice/bind.service
             └─7036 /usr/sbin/named -u named

Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './DNSKEY/IN': 2001:500:2f::f#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './NS/IN': 2001:500:2f::f#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './DNSKEY/IN': 2001:500:1::53#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './NS/IN': 2001:500:1::53#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './DNSKEY/IN': 2001:500:a8::e#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './NS/IN': 2001:500:a8::e#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './DNSKEY/IN': 2001:7fd::1#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './NS/IN': 2001:7fd::1#53
Nov 28 09:52:04 student-13 named[7036]: managed-keys-zone: Initializing automatic trust anchor management for zone '.'; DNSKEY ID 20326 is now trusted, waiving the normal 30-day waiting period.
Nov 28 09:52:04 student-13 named[7036]: managed-keys-zone: Initializing automatic trust anchor management for zone '.'; DNSKEY ID 38696 is now trusted, waiving the normal 30-day waiting period.
```

Служба активна (`active (running)`) и включена в автозагрузку (`enabled`).

### 5.2. Последние записи журнала

```bash
[student@student-13 ~]$ journalctl -u bind -n 15 --no-pager
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './NS/IN': 2001:dc3::35#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './DNSKEY/IN': 2001:500:9f::42#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './NS/IN': 2001:500:9f::42#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './DNSKEY/IN': 2001:500:12::d0d#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './NS/IN': 2001:500:12::d0d#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './DNSKEY/IN': 2001:500:2f::f#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './NS/IN': 2001:500:2f::f#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './DNSKEY/IN': 2001:500:1::53#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './NS/IN': 2001:500:1::53#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './DNSKEY/IN': 2001:500:a8::e#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './NS/IN': 2001:500:a8::e#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './DNSKEY/IN': 2001:7fd::1#53
Nov 28 09:52:04 student-13 named[7036]: network unreachable resolving './NS/IN': 2001:7fd::1#53
Nov 28 09:52:04 student-13 named[7036]: managed-keys-zone: Initializing automatic trust anchor management for zone '.'; DNSKEY ID 20326 is now trusted, waiving the normal 30-day waiting period.
Nov 28 09:52:04 student-13 named[7036]: managed-keys-zone: Initializing automatic trust anchor management for zone '.'; DNSKEY ID 38696 is now trusted, waiving the normal 30-day waiting period.
```

В журнале видно две группы записей: сообщения `network unreachable resolving './NS/IN'` и `'./DNSKEY/IN'` (сервер пытается связаться с корневыми DNS-серверами по IPv6, и до них нет маршрута) и инициализация доверенных ключей зоны `.`. Это не ошибка запуска: служба осталась в состоянии `active (running)`.

### 5.3. Пользователь, под которым работает служба

```bash
[student@student-13 ~]$ ps -o user,pid,ppid,cmd -C named
USER         PID    PPID CMD
named       7036       1 /usr/sbin/named -u named
```

Процесс `named` (PID 7036) работает от отдельного непривилегированного пользователя `named`, что соответствует ключу `-u named` в строке запуска. Родительский процесс — PID 1 (`systemd`).

## Вывод: установка пакета, запуск службы и включение автозапуска

Это три разных действия.

- **Установка пакета** только кладёт файлы на диск: программы, юниты, конфигурационные файлы. Службу она не запускает и в автозагрузку не добавляет: после установки `bind` оказался `inactive` и `disabled`.
- **Запуск службы** (`systemctl start`) создаёт работающий процесс прямо сейчас, но после перезагрузки службу придётся запускать снова.
- **Включение автозапуска** (`systemctl enable`) только создаёт настройку на будущее: служба будет стартовать при загрузке системы, но сама по себе сейчас не запускается.

Состояния независимы друг от друга: у службы может быть `active` без `enabled` и наоборот. Поэтому нужны обе проверки: `systemctl is-active` и `systemctl is-enabled`. Команда `systemctl enable --now` выполняет оба действия сразу.
