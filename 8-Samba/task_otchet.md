# Шарим

1) Установите пакет Samba.
2) Что такое общий сетевой ресурс и зачем он может быть нужен?
3) Создайте общую папку без пароля с доступом только на чтение.
4) Создайте общую папку с аутентификацией и правами на чтение и запись.
5) Создайте общую папку с доступом для отдельной группы с полными правами.
6) Создайте общий ресурс, для которого:
   - группа `teachers` имеет чтение и запись;
   - группа `students` имеет только чтение;
   - пользователи, не входящие в эти группы, не имеют доступа.
7) Проверьте доступ к ресурсу минимум от имени трёх разных пользователей.
8) Проверьте конфигурацию Samba штатной командой до перезапуска службы.
9) Покажите, какие TCP/UDP-порты слушает Samba после запуска.

## Критерий приёмки

В отчёте должны быть:
- команда установки Samba;
- фрагменты `smb.conf` для всех созданных ресурсов;
- пути к каталогам, опубликованным по SMB;
- команды создания пользователей и групп, использованных в задании;
- результаты проверки доступа от пользователя с полными правами;
- результаты проверки доступа от пользователя только с чтением;
- результат отказа в доступе для пользователя без разрешений;
- вывод встроенной проверки конфигурации Samba;
- список слушающих портов Samba.


## Отчет по практической работе: Шарим

## 1. Установка пакета Samba

Проверка, что пакеты ещё не установлены, и поиск их в репозитории:

```bash
[student@student-13 ~]$ rpm -q samba samba-client cifs-utils
package samba is not installed
package samba-client is not installed
package cifs-utils is not installed
[student@student-13 ~]$ cat /etc/os-release | head -3
NAME="ALT Linux"
VERSION="20250612"
ID=altlinux
```

Команда установки (запущена в фоне через `nohup`, чтобы обрыв SSH не прервал установку):

```bash
sudo nohup apt-get install -y samba samba-client > ~/samba_install.log 2>&1 &
```

Конец журнала установки и проверка результата:

```bash
[student@student-13 ~]$ tail -n 25 ~/samba_install.log; echo "=== rpm ==="; rpm -q samba samba-client
 2: libtevent-0.17.1-alt1               ################################# [  8%]
 3: libtdb-1.4.14-alt1                  ################################# [ 12%]
 ...
24: samba-client-4.23.13-alt1           ################################# [ 96%]
25: samba-4.23.13-alt1                  ################################# [100%]
Done.
=== rpm ===
samba-4.23.13-alt1.x86_64
samba-client-4.23.13-alt1.x86_64
```

Установлены `samba` и `samba-client` версии 4.23.13. Клиент нужен для проверки доступа через `smbclient`.

Утилиты `pdbedit` и `testparm` в этих пакетах не оказались (`sudo: pdbedit: command not found`). Они входят в отдельный пакет `samba-common-tools`, поэтому он установлен дополнительно:

```bash
[student@student-13 ~]$ sudo apt-get install -y samba-common-tools 2>&1 | tail -n 8
...
1: samba-common-tools-4.23.13-alt1      ################################# [100%]
Done.
[student@student-13 ~]$ which testparm pdbedit
/usr/bin/testparm
/usr/bin/pdbedit
```

## 2. Что такое общий сетевой ресурс и зачем он нужен

**Общий сетевой ресурс (шара)** — это каталог (или принтер) на сервере, который опубликован по сети, так что другие компьютеры могут подключаться к нему и работать с файлами как с локальными. В Samba доступ к шаре обеспечивает протокол SMB/CIFS, и этот же протокол используют Windows-клиенты.

Зачем это нужно:

- хранить общие документы в одном месте, а не копировать их на каждый компьютер;
- совместно работать с файлами нескольких пользователей;
- разграничивать права: одним разрешить только чтение, другим чтение и запись, остальным закрыть доступ;
- централизованно делать резервные копии и раздавать данные (например, учебные материалы преподавателей студентам).

## Подготовка: состояние системы до настройки

Службы Samba после установки выключены, групп `teachers` и `students` в системе нет:

```bash
[student@student-13 ~]$ ls -l /etc/samba/
total 24
-rw-r--r-- 1 root root    20 Nov 11 02:56 lmhosts
-rw-r--r-- 1 root root   940 Nov 11 02:56 smb.conf
-rw-r--r-- 1 root root 11319 Nov 11 02:56 smb.conf.example
-rw-r--r-- 1 root root    97 Nov 11 02:56 smbusers
[student@student-13 ~]$ systemctl list-unit-files | grep -E '^(smb|nmb|winbind)'
nmb.service                                  disabled        disabled
smb.service                                  disabled        disabled
[student@student-13 ~]$ getent group teachers students
[student@student-13 ~]$
```

Перед изменением конфигурации сделана резервная копия:

```bash
sudo cp -a /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

## Создание групп и пользователей

Группы `teachers`, `students`, `smbteam` и пользователи для проверки: `teacher1` (группа `teachers`), `student1` (группа `students`), `smbuser` (группа `smbteam`) и `outsider1` (без групп, для проверки отказа). Домашние каталоги и вход в систему отключены, пользователи нужны только для Samba.

```bash
sudo groupadd teachers
sudo groupadd students
sudo groupadd smbteam
sudo useradd -M -s /sbin/nologin -G teachers teacher1
sudo useradd -M -s /sbin/nologin -G students student1
sudo useradd -M -s /sbin/nologin -G smbteam smbuser
sudo useradd -M -s /sbin/nologin outsider1
```

```bash
[student@student-13 ~]$ id teacher1; id student1; id smbuser; id outsider1
uid=1004(teacher1) gid=1009(teacher1) groups=1009(teacher1),1006(teachers)
uid=1005(student1) gid=1010(student1) groups=1010(student1),1007(students)
uid=1006(smbuser) gid=1011(smbuser) groups=1011(smbuser),1008(smbteam)
uid=1007(outsider1) gid=1012(outsider1) groups=1012(outsider1)
```

Samba хранит пароли отдельно от системных, поэтому пользователи добавлены в базу Samba (тестовый пароль `Lab13-pass`):

```bash
[student@student-13 ~]$ for u in teacher1 student1 smbuser outsider1; do printf 'Lab13-pass\nLab13-pass\n' | sudo smbpasswd -a -s $u; done
Added user teacher1.
Added user student1.
Added user smbuser.
Added user outsider1.
[student@student-13 ~]$ sudo pdbedit -L
teacher1:1004:
smbuser:1006:
student1:1005:
outsider1:1007:
```

## Каталоги для общих ресурсов

```bash
sudo mkdir -p /srv/samba/{public,private,team,school}
sudo chown root:root /srv/samba/public && sudo chmod 755 /srv/samba/public
sudo chown smbuser:smbuser /srv/samba/private && sudo chmod 770 /srv/samba/private
sudo chown root:smbteam /srv/samba/team && sudo chmod 2770 /srv/samba/team
sudo chown root:teachers /srv/samba/school && sudo chmod 2775 /srv/samba/school
echo "public read-only file" | sudo tee /srv/samba/public/readme.txt
echo "school shared file" | sudo tee /srv/samba/school/lesson.txt
```

```bash
[student@student-13 ~]$ ls -ld /srv/samba/*
drwxrwx--- 2 smbuser smbuser  4096 Nov 26 14:03 /srv/samba/private
drwxr-xr-x 2 root    root     4096 Nov 26 14:03 /srv/samba/public
drwxrwsr-x 2 root    teachers 4096 Nov 26 14:03 /srv/samba/school
drwxrws--- 2 root    smbteam  4096 Nov 26 14:03 /srv/samba/team
```

Опубликованные по SMB каталоги:

| Ресурс | Путь |
|--------|------|
| `[public]` | `/srv/samba/public` |
| `[private]` | `/srv/samba/private` |
| `[team]` | `/srv/samba/team` |
| `[school]` | `/srv/samba/school` |

## 3–6. Настройка общих ресурсов в `smb.conf`

В секцию `[global]` добавлен параметр `map to guest = Bad User`, без него анонимный доступ к ресурсу без пароля не работает. Затем в конец файла дописаны четыре ресурса:

```bash
sudo sed -i '/^[[:space:]]*security = user/a\        map to guest = Bad User' /etc/samba/smb.conf

sudo tee -a /etc/samba/smb.conf > /dev/null <<'EOF'

[public]
        comment = Public read-only share
        path = /srv/samba/public
        guest ok = yes
        read only = yes
        browseable = yes

[private]
        comment = Private share with authentication
        path = /srv/samba/private
        valid users = smbuser
        guest ok = no
        read only = no
        browseable = yes

[team]
        comment = Share for group smbteam
        path = /srv/samba/team
        valid users = @smbteam
        force group = smbteam
        read only = no
        create mask = 0660
        directory mask = 2770
        browseable = yes

[school]
        comment = Teachers (rw) and students (ro)
        path = /srv/samba/school
        valid users = @teachers @students
        write list = @teachers
        read only = yes
        create mask = 0664
        directory mask = 2775
        browseable = yes
EOF
```

Итоговое содержимое файла (фрагменты `smb.conf`):

```ini
[global]
        workgroup = SAMBA
        security = user
        map to guest = Bad User

[public]
        comment = Public read-only share
        path = /srv/samba/public
        guest ok = yes
        read only = yes
        browseable = yes

[private]
        comment = Private share with authentication
        path = /srv/samba/private
        valid users = smbuser
        guest ok = no
        read only = no
        browseable = yes

[team]
        comment = Share for group smbteam
        path = /srv/samba/team
        valid users = @smbteam
        force group = smbteam
        read only = no
        create mask = 0660
        directory mask = 2770
        browseable = yes

[school]
        comment = Teachers (rw) and students (ro)
        path = /srv/samba/school
        valid users = @teachers @students
        write list = @teachers
        read only = yes
        create mask = 0664
        directory mask = 2775
        browseable = yes
```

Пояснение по ресурсам:

- **`[public]`** (п. 3): доступ без пароля (`guest ok = yes`), только чтение.
- **`[private]`** (п. 4): вход только по паролю, доступ только у `smbuser`, чтение и запись.
- **`[team]`** (п. 5): доступ только группе `smbteam` с полными правами; новые файлы получают группу `smbteam` (`force group`).
- **`[school]`** (п. 6): `valid users` допускает только группы `teachers` и `students`, `write list` даёт запись только `teachers`, остальным (`students`) остаётся чтение, всем прочим доступ закрыт.

## 8. Проверка конфигурации до запуска службы

Службы `smb` и `nmb` на этот момент ещё не были запущены. Штатная проверка:

```bash
[student@student-13 ~]$ sudo testparm -s
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
Weak crypto is allowed by GnuTLS (e.g. NTLM as a compatibility fallback)

Server role: ROLE_STANDALONE

# Global parameters
[global]
        map to guest = Bad User
        printcap name = cups
        security = USER
        workgroup = SAMBA
        idmap config * : backend = tdb
        cups options = raw
        include = /etc/samba/usershares.conf


[homes]
        browseable = No
        comment = Home Directories
        inherit acls = Yes
        read only = No
        valid users = %S %D%w%S


[printers]
        browseable = No
        comment = All Printers
        create mask = 0600
        path = /var/tmp
        printable = Yes


[print$]
        comment = Printer Drivers
        create mask = 0664
        directory mask = 0775
        force group = @printadmin
        path = /var/lib/samba/drivers
        write list = @printadmin root


[public]
        comment = Public read-only share
        guest ok = Yes
        path = /srv/samba/public


[private]
        comment = Private share with authentication
        path = /srv/samba/private
        read only = No
        valid users = smbuser


[team]
        comment = Share for group smbteam
        create mask = 0660
        directory mask = 02770
        force group = smbteam
        path = /srv/samba/team
        read only = No
        valid users = @smbteam


[school]
        comment = Teachers (rw) and students (ro)
        create mask = 0664
        directory mask = 02775
        path = /srv/samba/school
        valid users = @teachers @students
        write list = @teachers
```

`Loaded services file OK.` означает, что ошибок синтаксиса нет. `testparm -s` выводит только параметры, отличающиеся от значений по умолчанию, поэтому `read only = yes` и `browseable = yes` в выводе не показаны (это значения по умолчанию).

## 9. Запуск служб и слушающие порты

```bash
[student@student-13 ~]$ sudo systemctl enable --now smb nmb
Synchronizing state of smb.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable smb
Synchronizing state of nmb.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable nmb
Created symlink '/etc/systemd/system/multi-user.target.wants/smb.service' → '/usr/lib/systemd/system/smb.service'.
Created symlink '/etc/systemd/system/multi-user.target.wants/nmb.service' → '/usr/lib/systemd/system/nmb.service'.
[student@student-13 ~]$ systemctl is-active smb nmb
active
active
[student@student-13 ~]$ sudo ss -tulpn | grep -E 'smbd|nmbd'
udp   UNCONN 0      0      192.168.6.255:137       0.0.0.0:*    users:(("nmbd",pid=13525,fd=15))
udp   UNCONN 0      0      192.168.6.113:137       0.0.0.0:*    users:(("nmbd",pid=13525,fd=14))
udp   UNCONN 0      0            0.0.0.0:137       0.0.0.0:*    users:(("nmbd",pid=13525,fd=12))
udp   UNCONN 0      0      192.168.6.255:138       0.0.0.0:*    users:(("nmbd",pid=13525,fd=17))
udp   UNCONN 0      0      192.168.6.113:138       0.0.0.0:*    users:(("nmbd",pid=13525,fd=16))
udp   UNCONN 0      0            0.0.0.0:138       0.0.0.0:*    users:(("nmbd",pid=13525,fd=13))
tcp   LISTEN 0      50           0.0.0.0:139       0.0.0.0:*    users:(("smbd",pid=13527,fd=30))
tcp   LISTEN 0      50           0.0.0.0:445       0.0.0.0:*    users:(("smbd",pid=13527,fd=29))
tcp   LISTEN 0      50              [::]:139          [::]:*    users:(("smbd",pid=13527,fd=28))
tcp   LISTEN 0      50              [::]:445          [::]:*    users:(("smbd",pid=13527,fd=27))
```

Samba слушает:

- **TCP 445** и **TCP 139** (`smbd`, файловый доступ SMB; на IPv4 и IPv6);
- **UDP 137** и **UDP 138** (`nmbd`, службы имён и датаграмм NetBIOS).

## 7. Проверка доступа от разных пользователей

Проверка выполнена локально клиентом `smbclient`.

### Ресурс `[school]`: teacher1, student1, outsider1

```bash
[student@student-13 ~]$ echo "from teacher1" > /tmp/teacher.txt
[student@student-13 ~]$ echo "=== teacher1 ==="; smbclient //localhost/school -U teacher1%Lab13-pass -c 'ls; put /tmp/teacher.txt teacher.txt; ls'
=== teacher1 ===
  .                                   D        0  Thu Nov 26 14:03:55 2026
  ..                                  D        0  Thu Nov 26 14:03:55 2026
  lesson.txt                          N       19  Thu Nov 26 14:03:55 2026

                15454048 blocks of size 1024. 13093916 blocks available
putting file /tmp/teacher.txt as \teacher.txt (6.8 kB/s) (average 6.8 kB/s)
  .                                   D        0  Thu Nov 26 14:07:52 2026
  ..                                  D        0  Thu Nov 26 14:07:52 2026
  lesson.txt                          N       19  Thu Nov 26 14:03:55 2026
  teacher.txt                         A       14  Thu Nov 26 14:07:52 2026

                15454048 blocks of size 1024. 13093912 blocks available
[student@student-13 ~]$ echo "=== student1 ==="; smbclient //localhost/school -U student1%Lab13-pass -c 'ls; put /tmp/teacher.txt student.txt'
=== student1 ===
  .                                   D        0  Thu Nov 26 14:07:52 2026
  ..                                  D        0  Thu Nov 26 14:07:52 2026
  lesson.txt                          N       19  Thu Nov 26 14:03:55 2026
  teacher.txt                         A       14  Thu Nov 26 14:07:52 2026

                15454048 blocks of size 1024. 13093896 blocks available
NT_STATUS_ACCESS_DENIED opening remote file \student.txt
[student@student-13 ~]$ echo "=== outsider1 ==="; smbclient //localhost/school -U outsider1%Lab13-pass -c 'ls'
=== outsider1 ===
tree connect failed: NT_STATUS_ACCESS_DENIED
```

- **`teacher1`** (группа `teachers`) видит файлы и успешно загрузил `teacher.txt`: чтение и запись разрешены.
- **`student1`** (группа `students`) видит файлы, но запись закончилась `NT_STATUS_ACCESS_DENIED`: разрешено только чтение.
- **`outsider1`** (нет в группах) получил отказ уже при подключении к ресурсу (`tree connect failed: NT_STATUS_ACCESS_DENIED`): доступа нет.

### Остальные ресурсы: `[public]`, `[private]`, `[team]`

```bash
[student@student-13 ~]$ echo "=== public, guest ==="; smbclient //localhost/public -N -c 'ls; put /tmp/teacher.txt guest.txt'
=== public, guest ===
  .                                   D        0  Thu Nov 26 14:03:48 2026
  ..                                  D        0  Thu Nov 26 14:03:48 2026
  readme.txt                          N       22  Thu Nov 26 14:03:48 2026

                15454048 blocks of size 1024. 13093896 blocks available
NT_STATUS_ACCESS_DENIED opening remote file \guest.txt
[student@student-13 ~]$ echo "=== private, smbuser ==="; smbclient //localhost/private -U smbuser%Lab13-pass -c 'put /tmp/teacher.txt priv.txt; ls'
=== private, smbuser ===
putting file /tmp/teacher.txt as \priv.txt (6.8 kB/s) (average 6.8 kB/s)
  .                                   D        0  Thu Nov 26 14:08:51 2026
  ..                                  D        0  Thu Nov 26 14:08:51 2026
  priv.txt                            A       14  Thu Nov 26 14:08:51 2026

                15454048 blocks of size 1024. 13093892 blocks available
[student@student-13 ~]$ echo "=== private, teacher1 ==="; smbclient //localhost/private -U teacher1%Lab13-pass -c 'ls'
=== private, teacher1 ===
tree connect failed: NT_STATUS_ACCESS_DENIED
[student@student-13 ~]$ echo "=== team, smbuser ==="; smbclient //localhost/team -U smbuser%Lab13-pass -c 'put /tmp/teacher.txt team.txt; ls'
=== team, smbuser ===
putting file /tmp/teacher.txt as \team.txt (6.8 kB/s) (average 6.8 kB/s)
  .                                   D        0  Thu Nov 26 14:09:09 2026
  ..                                  D        0  Thu Nov 26 14:09:09 2026
  team.txt                            A       14  Thu Nov 26 14:09:09 2026

                15454048 blocks of size 1024. 13093888 blocks available
[student@student-13 ~]$ echo "=== team, teacher1 ==="; smbclient //localhost/team -U teacher1%Lab13-pass -c 'ls'
=== team, teacher1 ===
tree connect failed: NT_STATUS_ACCESS_DENIED
```

- **`[public]`**: гость без пароля (`-N`) видит файл `readme.txt`, но запись запрещена (`NT_STATUS_ACCESS_DENIED`): только чтение.
- **`[private]`**: `smbuser` с паролем читает и пишет, `teacher1` не имеет доступа.
- **`[team]`**: `smbuser` (группа `smbteam`) читает и пишет, `teacher1` (не в группе `smbteam`) не имеет доступа.

