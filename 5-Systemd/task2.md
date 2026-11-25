# Пишем юниты

1) Создайте скрипт, который создаёт каталог и четыре файла с именами `1`, `2`, `3`, `4`.
   В файлы запишите соответственно:
   - текущую дату;
   - версию ядра;
   - имя компьютера;
   - список файлов в домашнем каталоге пользователя, от которого выполняется скрипт.
   Не забудьте проверять существование каталогов и файлов.

2) Создайте systemd service, который запускает этот скрипт. Проверьте его работу.
3) Создайте timer, который запускает этот service каждые 5 минут.
4) От какого пользователя по умолчанию выполняются системные units?
5) Создайте отдельного пользователя для выполнения вашего скрипта.
6) Дополните unit так, чтобы скрипт выполнялся от созданного пользователя.
7) Дополните скрипт так, чтобы он независимо от текущего каталога выполнял работу в домашнем каталоге пользователя, от которого запущен.
8) Определите PID процесса во время выполнения, если он успевает существовать достаточно долго.
9) Найдите в journal записи о запуске service.
10) Намеренно внесите ошибку в скрипт или unit, добейтесь состояния `failed`, определите причину через `systemctl` и `journalctl`, затем исправьте.

## Критерий приёмки

В отчёте должны быть:
- полный текст скрипта;
- содержимое созданного service unit;
- содержимое созданного timer unit;
- команды `daemon-reload`, запуска и включения timer;
- вывод, подтверждающий работу service;
- вывод, подтверждающий расписание timer;
- информация о пользователе, от которого запускается скрипт;
- пример записей из journal о запуске;
- пример намеренно созданной ошибки;
- вывод, по которому была определена причина состояния `failed`;
- подтверждение успешной работы после исправления.


# Отчёт по практической работе: Пишем юниты
 
## Предварительная проверка системы
 
Использованные команды:
 
```bash
whoami; hostname; echo "$HOME"; uname -r; date
command -v bash; systemctl --version | head -1
ls -A ~
id
getent passwd | tail -5
```
 
Полученный результат:
 
```bash
[student@student-13 ~]$ whoami; hostname; echo "$HOME"; uname -r; date
student
student-13
/home/student
6.12.51-6.12-alt1
Tue Nov 24 08:09:54 PM UTC 2026
[student@student-13 ~]$ command -v bash; systemctl --version | head -1
/usr/bin/bash
systemd 257 (257.9-alt1)
[student@student-13 ~]$ id
uid=1000(student) gid=1001(student) groups=1001(student),4(adm),10(wheel),14(uucp),19(proc),100(users),987(systemd-journal),1000(lxd)
[student@student-13 ~]$ getent passwd | tail -5
systemd-network:x:996:984:systemd Network Management:/var/empty:/dev/null
systemd-resolve:x:995:983:systemd Resolver:/var/empty:/dev/null
iputils:x:994:982::/dev/null:/dev/null
student:x:1000:1001:ALT Linux Cloud User:/home/student:/bin/bash
user1:x:1001:1002::/home/user1:/bin/bash
```
 
Пояснение: определили реальные пути (`/usr/bin/bash`), версию systemd и текущего пользователя. Скрипт будет лежать в `/usr/local/bin`, чтобы его мог запускать и другой пользователь.
 
---
 
## 1. Скрипт, создающий каталог и четыре файла
 
Использованные команды (первая версия скрипта, каталог указан относительным путём):
 
```bash
sudo tee /usr/local/bin/lab_units.sh >/dev/null <<'EOF'
#!/usr/bin/bash
# Создаёт каталог units_lab и файлы 1-4 с системной информацией
 
DIR="units_lab"
 
# Создаём каталог, только если его ещё нет
[ -d "$DIR" ] || mkdir -p "$DIR"
 
# Создаём файлы, только если их ещё нет
for f in 1 2 3 4; do
    [ -e "$DIR/$f" ] || touch "$DIR/$f"
done
 
date > "$DIR/1"
uname -r > "$DIR/2"
hostname > "$DIR/3"
ls -A ~ > "$DIR/4"
EOF
sudo chmod 755 /usr/local/bin/lab_units.sh
ls -l /usr/local/bin/lab_units.sh
cd ~ && /usr/local/bin/lab_units.sh; echo "код возврата: $?"
ls -l ~/units_lab
head -n 20 ~/units_lab/1 ~/units_lab/2 ~/units_lab/3 ~/units_lab/4
```
 
Полученный результат:
 
```bash
[student@student-13 ~]$ ls -l /usr/local/bin/lab_units.sh
-rwxr-xr-x 1 root root 461 Nov 24 20:11 /usr/local/bin/lab_units.sh
[student@student-13 ~]$ cd ~ && /usr/local/bin/lab_units.sh; echo "код возврата: $?"
код возврата: 0
[student@student-13 ~]$ ls -l ~/units_lab
total 16
-rw-r--r-- 1 student student  32 Nov 24 20:11 1
-rw-r--r-- 1 student student  18 Nov 24 20:11 2
-rw-r--r-- 1 student student  11 Nov 24 20:11 3
-rw-r--r-- 1 student student 153 Nov 24 20:11 4
[student@student-13 ~]$ head -n 20 ~/units_lab/1 ~/units_lab/2 ~/units_lab/3 ~/units_lab/4
==> /home/student/units_lab/1 <==
Tue Nov 24 08:11:39 PM UTC 2026
 
==> /home/student/units_lab/2 <==
6.12.51-6.12-alt1
 
==> /home/student/units_lab/3 <==
student-13
 
==> /home/student/units_lab/4 <==
.bash_history
.bash_logout
.bash_profile
.bashrc
.cache
.config
.lesshst
.local
.lpoptions
.mutt
.rpmmacros
scripts
.ssh
units_lab
.xprofile
.xsession.d
```
 
Пояснение: скрипт создал каталог `units_lab` и файлы `1`–`4` (дата, версия ядра, имя компьютера, список файлов домашнего каталога). Существование каталога и файлов проверяется через `[ -d ... ] ||` и `[ -e ... ] ||`. Итоговая версия скрипта (с работой в домашнем каталоге) приведена в пункте 7.
 
---
 
## 2. Systemd service, запускающий скрипт
 
Использованные команды:
 
```bash
sudo tee /etc/systemd/system/lab-units.service >/dev/null <<'EOF'
[Unit]
Description=Lab script: create units_lab directory with files 1-4
 
[Service]
Type=oneshot
ExecStart=/usr/local/bin/lab_units.sh
EOF
sudo systemctl daemon-reload
sudo systemctl start lab-units.service
systemctl status lab-units.service --no-pager
ls -l /units_lab
sudo head -n 20 /units_lab/1 /units_lab/2 /units_lab/3 /units_lab/4
```
 
Содержимое service unit (первая версия):
 
```ini
[Unit]
Description=Lab script: create units_lab directory with files 1-4
 
[Service]
Type=oneshot
ExecStart=/usr/local/bin/lab_units.sh
```
 
Полученный результат:
 
```bash
[student@student-13 ~]$ systemctl status lab-units.service --no-pager
○ lab-units.service - Lab script: create units_lab directory with files 1-4
     Loaded: loaded (/etc/systemd/system/lab-units.service; static)
     Active: inactive (dead)
 
Nov 24 20:12:57 student-13 systemd[1]: Starting lab-units.service - Lab script…4...
Nov 24 20:12:57 student-13 systemd[1]: lab-units.service: Deactivated successfully.
Nov 24 20:12:57 student-13 systemd[1]: Finished lab-units.service - Lab script…1-4.
Hint: Some lines were ellipsized, use -l to show in full.
[student@student-13 ~]$ ls -l /units_lab
total 16
-rw-r--r-- 1 root root  32 Nov 24 20:12 1
-rw-r--r-- 1 root root  18 Nov 24 20:12 2
-rw-r--r-- 1 root root  11 Nov 24 20:12 3
-rw-r--r-- 1 root root 283 Nov 24 20:12 4
[student@student-13 ~]$ sudo head -n 20 /units_lab/1 /units_lab/2 /units_lab/3 /units_lab/4
==> /units_lab/1 <==
Tue Nov 24 08:12:57 PM UTC 2026
 
==> /units_lab/2 <==
6.12.51-6.12-alt1
 
==> /units_lab/3 <==
student-13
 
==> /units_lab/4 <==
.bash_logout
.bash_profile
.bashrc
.i18n
.install-log
mac-change-backup-student-mac-change-20261116-024745
.rpmmacros
.ssh
ssh-password-backup-student-password-ssh-20261116-173614
student-boot-fix-backup-student-boot-fix-20261114-223635
.tcshrc
tmp
.zlogout
.zprofile
.zshenv
.zshrc
```
 
Пояснение: служба отработала успешно (`Finished`, `Deactivated successfully`). Статус `inactive (dead)` для `Type=oneshot` нормален: скрипт выполнился и завершился. Служба запустилась от root в каталоге `/`, поэтому файлы появились в `/units_lab` и принадлежат root, а в файле `4` список файлов из домашнего каталога root. Это исправлено в пунктах 6–7.
 
---
 
## 3. Timer, запускающий service каждые 5 минут
 
Использованные команды:
 
```bash
sudo tee /etc/systemd/system/lab-units.timer >/dev/null <<'EOF'
[Unit]
Description=Run lab-units.service every 5 minutes
 
[Timer]
OnCalendar=*:0/5
Unit=lab-units.service
 
[Install]
WantedBy=timers.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable --now lab-units.timer
systemctl status lab-units.timer --no-pager
systemctl list-timers lab-units.timer --no-pager
systemctl is-enabled lab-units.timer
```
 
Содержимое timer unit:
 
```ini
[Unit]
Description=Run lab-units.service every 5 minutes
 
[Timer]
OnCalendar=*:0/5
Unit=lab-units.service
 
[Install]
WantedBy=timers.target
```
 
Полученный результат (расписание):
 
```bash
[student@student-13 ~]$ sudo systemctl daemon-reload
[student@student-13 ~]$ sudo systemctl enable --now lab-units.timer
Created symlink '/etc/systemd/system/timers.target.wants/lab-units.timer' → '/etc/systemd/system/lab-units.timer'.
[student@student-13 ~]$ systemctl status lab-units.timer --no-pager
● lab-units.timer - Run lab-units.service every 5 minutes
     Loaded: loaded (/etc/systemd/system/lab-units.timer; enabled; preset: disabled)
     Active: active (waiting) since Tue 2026-11-24 20:14:23 UTC; 6s ago
 Invocation: bb9f8a779e2349d58afea3f98d254dc1
    Trigger: Tue 2026-11-24 20:15:00 UTC; 30s left
   Triggers: ● lab-units.service
 
Nov 24 20:14:23 student-13 systemd[1]: Started lab-units.timer - Run lab-units…tes.
Hint: Some lines were ellipsized, use -l to show in full.
[student@student-13 ~]$ systemctl list-timers lab-units.timer --no-pager
NEXT                        LEFT LAST PASSED UNIT            ACTIVATES
Tue 2026-11-24 20:15:00 UTC  25s -         - lab-units.timer lab-units.service
 
1 timers listed.
Pass --all to see loaded but inactive timers, too.
[student@student-13 ~]$ systemctl is-enabled lab-units.timer
enabled
```
 
Проверка срабатывания таймера:
 
```bash
[student@student-13 ~]$ date
Tue Nov 24 08:15:32 PM UTC 2026
[student@student-13 ~]$ systemctl list-timers lab-units.timer --no-pager
NEXT                 LEFT LAST             PASSED UNIT            ACTIVATES
Tue 2026-11-24 … 4min 17s Tue 2026-11-24 … 4s ago lab-units.timer lab-units.service
 
1 timers listed.
Pass --all to see loaded but inactive timers, too.
[student@student-13 ~]$ sudo cat /units_lab/1
Tue Nov 24 08:15:38 PM UTC 2026
[student@student-13 ~]$ journalctl -u lab-units.service --no-pager -n 10
Nov 24 20:12:57 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 24 20:12:57 student-13 systemd[1]: lab-units.service: Deactivated successfully.
Nov 24 20:12:57 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
Nov 24 20:15:38 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 24 20:15:38 student-13 systemd[1]: lab-units.service: Deactivated successfully.
Nov 24 20:15:38 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
```
 
Пояснение: `OnCalendar=*:0/5` задаёт запуск в 00, 05, 10… минут каждого часа. Секция `[Install]` нужна, чтобы `enable` включил таймер в автозагрузку. Таймер сработал (запись в `LAST`, обновился файл `/units_lab/1`, в journal новая тройка записей). Запуск произошёл в 20:15:38, а не в 20:15:00, потому что у таймеров по умолчанию допуск `AccuracySec=1min`.
 
---
 
## 4. От какого пользователя по умолчанию выполняются системные units
 
Использованные команды:
 
```bash
systemctl show lab-units.service -p User -p Group -p ExecMainStatus
ls -ld /units_lab
```
 
Полученный результат:
 
```bash
[student@student-13 ~]$ systemctl show lab-units.service -p User -p Group -p ExecMainStatus
ExecMainStatus=0
User=
Group=
[student@student-13 ~]$ ls -ld /units_lab
drwxr-xr-x 2 root root 4096 Nov 24 20:12 /units_lab
```
 
Пояснение: в unit пользователь не задан (`User=` пуст), а каталог `/units_lab`, созданный службой, принадлежит root. Значит, системные units по умолчанию выполняются от **root**.
 
---
 
## 5. Отдельный пользователь для выполнения скрипта
 
Использованные команды:
 
```bash
sudo useradd -m -s /bin/false labrunner
id labrunner
getent passwd labrunner
ls -ld /home/labrunner
```
 
Полученный результат:
 
```bash
[student@student-13 ~]$ id labrunner
uid=1002(labrunner) gid=1005(labrunner) groups=1005(labrunner)
[student@student-13 ~]$ getent passwd labrunner
labrunner:x:1002:1005::/home/labrunner:/bin/false
[student@student-13 ~]$ ls -ld /home/labrunner
drwx------ 8 labrunner labrunner 4096 Nov 24 20:16 /home/labrunner
```
 
Пояснение: создан пользователь `labrunner` (UID 1002) с домашним каталогом `/home/labrunner` (`-m`) и без интерактивного входа (`-s /bin/false`). Для запуска службы оболочка не нужна.
 
---
 
## 6. Запуск service от созданного пользователя
 
Использованные команды:
 
```bash
sudo tee /etc/systemd/system/lab-units.service >/dev/null <<'EOF'
[Unit]
Description=Lab script: create units_lab directory with files 1-4
 
[Service]
Type=oneshot
User=labrunner
ExecStart=/usr/local/bin/lab_units.sh
EOF
sudo systemctl daemon-reload
sudo systemctl start lab-units.service; echo "код возврата start: $?"
systemctl show lab-units.service -p User -p Result -p ExecMainStatus
systemctl status lab-units.service --no-pager -l
journalctl -u lab-units.service --no-pager -n 8
```
 
Полученный результат:
 
```bash
[student@student-13 ~]$ sudo systemctl start lab-units.service; echo "код возврата start: $?"
Job for lab-units.service failed because the control process exited with error code.
See "systemctl status lab-units.service" and "journalctl -xeu lab-units.service" for details.
код возврата start: 1
[student@student-13 ~]$ systemctl show lab-units.service -p User -p Result -p ExecMainStatus
Result=exit-code
ExecMainStatus=1
User=labrunner
[student@student-13 ~]$ systemctl status lab-units.service --no-pager -l
× lab-units.service - Lab script: create units_lab directory with files 1-4
     Loaded: loaded (/etc/systemd/system/lab-units.service; static)
     Active: failed (Result: exit-code) since Tue 2026-11-24 20:18:36 UTC; 11s ago
 Invocation: d6526ebbc875462fbddf3704d6c18567
TriggeredBy: ● lab-units.timer
    Process: 2652 ExecStart=/usr/local/bin/lab_units.sh (code=exited, status=1/FAILURE)
   Main PID: 2652 (code=exited, status=1/FAILURE)
   Mem peak: 1.1M
        CPU: 13ms
 
Nov 24 20:18:36 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 24 20:18:36 student-13 lab_units.sh[2653]: /usr/local/bin/lab_units.sh: line 14: units_lab/1: Permission denied
Nov 24 20:18:36 student-13 lab_units.sh[2654]: /usr/local/bin/lab_units.sh: line 15: units_lab/2: Permission denied
Nov 24 20:18:36 student-13 systemd[1]: lab-units.service: Main process exited, code=exited, status=1/FAILURE
Nov 24 20:18:36 student-13 lab_units.sh[2655]: /usr/local/bin/lab_units.sh: line 16: units_lab/3: Permission denied
Nov 24 20:18:36 student-13 systemd[1]: lab-units.service: Failed with result 'exit-code'.
Nov 24 20:18:36 student-13 lab_units.sh[2656]: /usr/local/bin/lab_units.sh: line 17: units_lab/4: Permission denied
Nov 24 20:18:36 student-13 systemd[1]: Failed to start lab-units.service - Lab script: create units_lab directory with files 1-4.
```
 
Пояснение: unit запускается от `labrunner` (`User=labrunner`), но скрипт падает: текущий каталог службы `/`, там уже есть `/units_lab` от root, и `labrunner` не может в него писать (`Permission denied`). Это исправляется в пункте 7.
 
---
 
## 7. Работа в домашнем каталоге пользователя, от которого запущен скрипт
 
Использованные команды:
 
```bash
sudo tee /usr/local/bin/lab_units.sh >/dev/null <<'EOF'
#!/usr/bin/bash
# Создаёт каталог units_lab с файлами 1-4 в домашнем каталоге
# пользователя, от которого запущен скрипт
 
# Домашний каталог текущего пользователя (не зависит от текущего каталога)
USER_HOME=$(getent passwd "$(id -un)" | cut -d: -f6)
cd "$USER_HOME" || exit 1
 
DIR="units_lab"
 
# Создаём каталог, только если его ещё нет
[ -d "$DIR" ] || mkdir -p "$DIR"
 
# Создаём файлы, только если их ещё нет
for f in 1 2 3 4; do
    [ -e "$DIR/$f" ] || touch "$DIR/$f"
done
 
date > "$DIR/1"
uname -r > "$DIR/2"
hostname > "$DIR/3"
ls -A "$USER_HOME" > "$DIR/4"
EOF
cat /usr/local/bin/lab_units.sh
cd /tmp && /usr/local/bin/lab_units.sh; echo "код возврата: $?"; cd ~
ls -l ~/units_lab
sudo systemctl start lab-units.service; echo "код возврата start: $?"
systemctl status lab-units.service --no-pager -l
sudo ls -l /home/labrunner/units_lab
sudo head -n 20 /home/labrunner/units_lab/1 /home/labrunner/units_lab/2 /home/labrunner/units_lab/3 /home/labrunner/units_lab/4
```
 
Полный текст скрипта `/usr/local/bin/lab_units.sh` (итоговая версия):
 
```bash
#!/usr/bin/bash
# Создаёт каталог units_lab с файлами 1-4 в домашнем каталоге
# пользователя, от которого запущен скрипт
 
# Домашний каталог текущего пользователя (не зависит от текущего каталога)
USER_HOME=$(getent passwd "$(id -un)" | cut -d: -f6)
cd "$USER_HOME" || exit 1
 
DIR="units_lab"
 
# Создаём каталог, только если его ещё нет
[ -d "$DIR" ] || mkdir -p "$DIR"
 
# Создаём файлы, только если их ещё нет
for f in 1 2 3 4; do
    [ -e "$DIR/$f" ] || touch "$DIR/$f"
done
 
date > "$DIR/1"
uname -r > "$DIR/2"
hostname > "$DIR/3"
ls -A "$USER_HOME" > "$DIR/4"
```
 
Содержимое итогового service unit `/etc/systemd/system/lab-units.service`:
 
```ini
[Unit]
Description=Lab script: create units_lab directory with files 1-4
 
[Service]
Type=oneshot
User=labrunner
ExecStart=/usr/local/bin/lab_units.sh
```
 
Полученный результат:
 
```bash
[student@student-13 ~]$ cd /tmp && /usr/local/bin/lab_units.sh; echo "код возврата: $?"; cd ~
код возврата: 0
[student@student-13 ~]$ ls -l ~/units_lab
total 16
-rw-r--r-- 1 student student  32 Nov 24 20:20 1
-rw-r--r-- 1 student student  18 Nov 24 20:20 2
-rw-r--r-- 1 student student  11 Nov 24 20:20 3
-rw-r--r-- 1 student student 153 Nov 24 20:20 4
[student@student-13 ~]$ sudo systemctl start lab-units.service; echo "код возврата start: $?"
код возврата start: 0
[student@student-13 ~]$ systemctl status lab-units.service --no-pager -l
○ lab-units.service - Lab script: create units_lab directory with files 1-4
     Loaded: loaded (/etc/systemd/system/lab-units.service; static)
     Active: inactive (dead) since Tue 2026-11-24 20:20:24 UTC; 5s ago
 Invocation: 23237867b6764ccc9f558d7bb59e9de0
TriggeredBy: ● lab-units.timer
    Process: 2683 ExecStart=/usr/local/bin/lab_units.sh (code=exited, status=0/SUCCESS)
   Main PID: 2683 (code=exited, status=0/SUCCESS)
   Mem peak: 1.8M
        CPU: 53ms
 
Nov 24 20:20:24 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 24 20:20:24 student-13 systemd[1]: lab-units.service: Deactivated successfully.
Nov 24 20:20:24 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
[student@student-13 ~]$ sudo ls -l /home/labrunner/units_lab
total 16
-rw-r--r-- 1 labrunner labrunner  32 Nov 24 20:20 1
-rw-r--r-- 1 labrunner labrunner  18 Nov 24 20:20 2
-rw-r--r-- 1 labrunner labrunner  11 Nov 24 20:20 3
-rw-r--r-- 1 labrunner labrunner 122 Nov 24 20:20 4
[student@student-13 ~]$ sudo head -n 20 /home/labrunner/units_lab/1 /home/labrunner/units_lab/2 /home/labrunner/units_lab/3 /home/labrunner/units_lab/4
==> /home/labrunner/units_lab/1 <==
Tue Nov 24 08:20:24 PM UTC 2026
 
==> /home/labrunner/units_lab/2 <==
6.12.51-6.12-alt1
 
==> /home/labrunner/units_lab/3 <==
student-13
 
==> /home/labrunner/units_lab/4 <==
.bash_logout
.bash_profile
.bashrc
.cache
.config
.local
.lpoptions
.mutt
.rpmmacros
.ssh
units_lab
.xprofile
.xsession.d
```
 
Пояснение: скрипт определяет домашний каталог по имени пользователя (`getent passwd "$(id -un)"`) и переходит в него. Запуск вручную из `/tmp` от `student` создал `units_lab` в `/home/student`, а служба от `labrunner` создала его в `/home/labrunner`. В файле `4` у `labrunner` список файлов его собственного домашнего каталога. Служба отрабатывает успешно (`status=0/SUCCESS`).
 
---
 
## 8. PID процесса во время выполнения
 
Скрипт выполняется доли секунды, поэтому на время проверки в него временно вставили `sleep 30`, а затем вернули исходную версию из резервной копии.
 
Использованные команды:
 
```bash
sudo cp /usr/local/bin/lab_units.sh /tmp/lab_units.sh.bak
sudo sed -i '/^DIR=/i sleep 30' /usr/local/bin/lab_units.sh
grep -n sleep /usr/local/bin/lab_units.sh
sudo systemctl start --no-block lab-units.service
sleep 2
systemctl show lab-units.service -p MainPID -p ActiveState -p SubState
systemctl status lab-units.service --no-pager | head -n 12
ps -u labrunner -o pid,ppid,user,cmd
sleep 30
sudo cp /tmp/lab_units.sh.bak /usr/local/bin/lab_units.sh
grep -c sleep /usr/local/bin/lab_units.sh
systemctl show lab-units.service -p ActiveState -p Result
```
 
Полученный результат:
 
```bash
[student@student-13 ~]$ grep -n sleep /usr/local/bin/lab_units.sh
9:sleep 30
[student@student-13 ~]$ sudo systemctl start --no-block lab-units.service
[student@student-13 ~]$ sleep 2
[student@student-13 ~]$ systemctl show lab-units.service -p MainPID -p ActiveState -p SubState
MainPID=6106
ActiveState=activating
SubState=start
[student@student-13 ~]$ systemctl status lab-units.service --no-pager | head -n 12
● lab-units.service - Lab script: create units_lab directory with files 1-4
     Loaded: loaded (/etc/systemd/system/lab-units.service; static)
     Active: activating (start) since Wed 2026-11-25 12:57:10 UTC; 14s ago
        Job: 28908
 Invocation: 205563dabbfa4a5b9422f7ff165e41f5
TriggeredBy: ● lab-units.timer
   Main PID: 6106 (lab_units.sh)
      Tasks: 2 (limit: 1728)
     Memory: 892K (peak: 1.7M)
        CPU: 25ms
     CGroup: /system.slice/lab-units.service
             ├─6106 /usr/bin/bash /usr/local/bin/lab_units.sh
[student@student-13 ~]$ ps -u labrunner -o pid,ppid,user,cmd
    PID    PPID USER     CMD
   6106       1 labrunn+ /usr/bin/bash /usr/local/bin/lab_units.sh
   6111    6106 labrunn+ sleep 30
[student@student-13 ~]$ sudo cp /tmp/lab_units.sh.bak /usr/local/bin/lab_units.sh
[student@student-13 ~]$ grep -c sleep /usr/local/bin/lab_units.sh
0
[student@student-13 ~]$ systemctl show lab-units.service -p ActiveState -p Result
Result=success
ActiveState=inactive
```
 
Пояснение: во время выполнения служба в состоянии `activating (start)`, её главный процесс (`Main PID`) имеет PID **6106**, дочерний процесс `sleep 30` имеет PID 6111. Процесс запущен от пользователя `labrunner` (в выводе `ps` имя обрезано до `labrunn+`), родитель — PID 1 (systemd). После проверки скрипт возвращён в исходное состояние (`grep -c sleep` дал 0).
 
---
 
## 9. Записи в journal о запуске service
 
Использованные команды:
 
```bash
journalctl -u lab-units.service --since today --no-pager
journalctl -u lab-units.timer --since today --no-pager
journalctl -u lab-units.service --since today -o short-precise --no-pager | grep -E "Starting|Finished"
journalctl -u lab-units.timer --no-pager
```
 
Полученный результат (фрагмент: вывод `--since today` за 25 ноября содержит запуски каждые 5 минут с 00:00, здесь приведены первые записи и запись ручного запуска из пункта 8):
 
```bash
[student@student-13 ~]$ journalctl -u lab-units.service --since today --no-pager
Nov 25 00:00:08 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 25 00:00:08 student-13 systemd[1]: lab-units.service: Deactivated successfully.
Nov 25 00:00:08 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
Nov 25 00:05:08 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 25 00:05:08 student-13 systemd[1]: lab-units.service: Deactivated successfully.
Nov 25 00:05:08 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
...
Nov 25 12:55:38 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 25 12:55:38 student-13 systemd[1]: lab-units.service: Deactivated successfully.
Nov 25 12:55:38 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
Nov 25 12:57:10 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 25 12:57:40 student-13 systemd[1]: lab-units.service: Deactivated successfully.
Nov 25 12:57:40 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
[student@student-13 ~]$ journalctl -u lab-units.service --since today -o short-precise --no-pager | grep -E "Starting|Finished"
Nov 25 00:00:08.559560 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 25 00:00:08.607515 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
...
Nov 25 12:57:10.211652 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 25 12:57:40.266796 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
[student@student-13 ~]$ journalctl -u lab-units.timer --since today --no-pager
-- No entries --
[student@student-13 ~]$ journalctl -u lab-units.timer --no-pager
Nov 24 20:14:23 student-13 systemd[1]: Started lab-units.timer - Run lab-units.service every 5 minutes.
```
 
Пояснение: в journal видны запуски service каждые 5 минут (`Starting` → `Deactivated successfully` → `Finished`). Секунды в записях плавают из-за допуска таймера (`AccuracySec=1min`). Запись с интервалом 30 секунд между `Starting` (12:57:10) и `Finished` (12:57:40) соответствует ручному запуску с `sleep 30` из пункта 8. Журнал таймера за сегодня пуст, потому что запись о включении таймера сделана вчера (24 ноября 20:14:23), её показывает команда без `--since today`.
 
---
 
## 10. Намеренная ошибка и состояние `failed`
 
Использованные команды (внесение ошибки: в unit указан несуществующий файл `lab_units_typo.sh`):
 
```bash
sudo cp /etc/systemd/system/lab-units.service /tmp/lab-units.service.bak
sudo sed -i 's|^ExecStart=.*|ExecStart=/usr/local/bin/lab_units_typo.sh|' /etc/systemd/system/lab-units.service
grep ExecStart /etc/systemd/system/lab-units.service
sudo systemctl daemon-reload
sudo systemctl start lab-units.service; echo "код возврата start: $?"
```
 
Полученный результат:
 
```bash
[student@student-13 ~]$ grep ExecStart /etc/systemd/system/lab-units.service
ExecStart=/usr/local/bin/lab_units_typo.sh
[student@student-13 ~]$ sudo systemctl start lab-units.service; echo "код возврата start: $?"
Job for lab-units.service failed because the control process exited with error code.
See "systemctl status lab-units.service" and "journalctl -xeu lab-units.service" for details.
код возврата start: 1
```
 
Диагностика причины:
 
```bash
systemctl status lab-units.service --no-pager -l
systemctl --failed --no-pager
systemctl show lab-units.service -p Result -p ExecMainStatus
journalctl -u lab-units.service -n 6 --no-pager
```
 
```bash
[student@student-13 ~]$ systemctl status lab-units.service --no-pager -l
× lab-units.service - Lab script: create units_lab directory with files 1-4
     Loaded: loaded (/etc/systemd/system/lab-units.service; static)
     Active: failed (Result: exit-code) since Wed 2026-11-25 13:00:15 UTC; 23s ago
 Invocation: b38aa75dd66c4565a1ea8ee4b8e7abda
TriggeredBy: ● lab-units.timer
    Process: 6199 ExecStart=/usr/local/bin/lab_units_typo.sh (code=exited, status=203/EXEC)
   Main PID: 6199 (code=exited, status=203/EXEC)
   Mem peak: 1.1M
        CPU: 6ms
 
Nov 25 13:00:15 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 25 13:00:15 student-13 (_typo.sh)[6199]: lab-units.service: Unable to locate executable '/usr/local/bin/lab_units_typo.sh': No such file or directory
Nov 25 13:00:15 student-13 (_typo.sh)[6199]: lab-units.service: Failed at step EXEC spawning /usr/local/bin/lab_units_typo.sh: No such file or directory
Nov 25 13:00:15 student-13 systemd[1]: lab-units.service: Main process exited, code=exited, status=203/EXEC
Nov 25 13:00:15 student-13 systemd[1]: lab-units.service: Failed with result 'exit-code'.
Nov 25 13:00:15 student-13 systemd[1]: Failed to start lab-units.service - Lab script: create units_lab directory with files 1-4.
[student@student-13 ~]$ systemctl --failed --no-pager
  UNIT              LOAD   ACTIVE SUB    DESCRIPTION
● lab-units.service loaded failed failed Lab script: create units_lab directory wit…
● mdmonitor.service loaded failed failed MD array monitor
 
Legend: LOAD   → Reflects whether the unit definition was properly loaded.
        ACTIVE → The high-level unit activation state, i.e. generalization of SUB.
        SUB    → The low-level unit activation state, values depend on unit type.
 
2 loaded units listed.
[student@student-13 ~]$ systemctl show lab-units.service -p Result -p ExecMainStatus
Result=exit-code
ExecMainStatus=203
[student@student-13 ~]$ journalctl -u lab-units.service -n 6 --no-pager
Nov 25 13:00:15 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 25 13:00:15 student-13 (_typo.sh)[6199]: lab-units.service: Unable to locate executable '/usr/local/bin/lab_units_typo.sh': No such file or directory
Nov 25 13:00:15 student-13 (_typo.sh)[6199]: lab-units.service: Failed at step EXEC spawning /usr/local/bin/lab_units_typo.sh: No such file or directory
Nov 25 13:00:15 student-13 systemd[1]: lab-units.service: Main process exited, code=exited, status=203/EXEC
Nov 25 13:00:15 student-13 systemd[1]: lab-units.service: Failed with result 'exit-code'.
Nov 25 13:00:15 student-13 systemd[1]: Failed to start lab-units.service - Lab script: create units_lab directory with files 1-4.
```
 
Причина `failed`: код `status=203/EXEC` означает, что systemd не смог запустить исполняемый файл, а сообщение `Unable to locate executable '/usr/local/bin/lab_units_typo.sh': No such file or directory` показывает, что файла по пути из `ExecStart` не существует. (В списке `--failed` также есть `mdmonitor.service`, он к этой работе не относится.)
 
Исправление (возврат исходного unit из копии):
 
```bash
sudo cp /tmp/lab-units.service.bak /etc/systemd/system/lab-units.service
cat /etc/systemd/system/lab-units.service
sudo systemctl daemon-reload
sudo systemctl reset-failed lab-units.service
sudo systemctl start lab-units.service; echo "код возврата start: $?"
systemctl show lab-units.service -p User -p Result -p ExecMainStatus
systemctl status lab-units.service --no-pager -l
systemctl --failed --no-pager
sudo cat /home/labrunner/units_lab/1
date
```
 
```bash
[student@student-13 ~]$ cat /etc/systemd/system/lab-units.service
[Unit]
Description=Lab script: create units_lab directory with files 1-4
 
[Service]
Type=oneshot
User=labrunner
ExecStart=/usr/local/bin/lab_units.sh
[student@student-13 ~]$ sudo systemctl start lab-units.service; echo "код возврата start: $?"
код возврата start: 0
[student@student-13 ~]$ systemctl show lab-units.service -p User -p Result -p ExecMainStatus
Result=success
ExecMainStatus=0
User=labrunner
[student@student-13 ~]$ systemctl status lab-units.service --no-pager -l
○ lab-units.service - Lab script: create units_lab directory with files 1-4
     Loaded: loaded (/etc/systemd/system/lab-units.service; static)
     Active: inactive (dead) since Wed 2026-11-25 13:01:55 UTC; 10s ago
 Invocation: 3948cff33ffa4e85bbfd798dc89396f8
TriggeredBy: ● lab-units.timer
    Process: 6287 ExecStart=/usr/local/bin/lab_units.sh (code=exited, status=0/SUCCESS)
   Main PID: 6287 (code=exited, status=0/SUCCESS)
   Mem peak: 1.6M
        CPU: 37ms
 
Nov 25 13:01:50 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 25 13:01:55 student-13 systemd[1]: lab-units.service: Deactivated successfully.
Nov 25 13:01:55 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
[student@student-13 ~]$ systemctl --failed --no-pager
  UNIT              LOAD   ACTIVE SUB    DESCRIPTION
● mdmonitor.service loaded failed failed MD array monitor
 
Legend: LOAD   → Reflects whether the unit definition was properly loaded.
        ACTIVE → The high-level unit activation state, i.e. generalization of SUB.
        SUB    → The low-level unit activation state, values depend on unit type.
 
1 loaded units listed.
[student@student-13 ~]$ sudo cat /home/labrunner/units_lab/1
Wed Nov 25 01:01:50 PM UTC 2026
```
 
Проверка, что таймер после исправления снова запускает service от `labrunner`:
 
```bash
date
systemctl list-timers lab-units.timer --no-pager
journalctl -u lab-units.service --since "13:02" --no-pager
sudo ls -l --time-style=full-iso /home/labrunner/units_lab
```
 
```bash
[student@student-13 ~]$ date
Wed Nov 25 01:10:14 PM UTC 2026
[student@student-13 ~]$ systemctl list-timers lab-units.timer --no-pager
NEXT LEFT LAST                         PASSED UNIT            ACTIVATES
-       - Wed 2026-11-25 13:10:19 UTC 7ms ago lab-units.timer lab-units.service
 
1 timers listed.
Pass --all to see loaded but inactive timers, too.
[student@student-13 ~]$ journalctl -u lab-units.service --since "13:02" --no-pager
Nov 25 13:05:28 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 25 13:05:28 student-13 systemd[1]: lab-units.service: Deactivated successfully.
Nov 25 13:05:28 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
Nov 25 13:10:19 student-13 systemd[1]: Starting lab-units.service - Lab script: create units_lab directory with files 1-4...
Nov 25 13:10:19 student-13 systemd[1]: lab-units.service: Deactivated successfully.
Nov 25 13:10:19 student-13 systemd[1]: Finished lab-units.service - Lab script: create units_lab directory with files 1-4.
[student@student-13 ~]$ sudo ls -l --time-style=full-iso /home/labrunner/units_lab
total 16
-rw-r--r-- 1 labrunner labrunner  32 2026-11-25 13:10:19.415000000 +0000 1
-rw-r--r-- 1 labrunner labrunner  18 2026-11-25 13:10:19.419000000 +0000 2
-rw-r--r-- 1 labrunner labrunner  11 2026-11-25 13:10:19.422000000 +0000 3
-rw-r--r-- 1 labrunner labrunner 122 2026-11-25 13:10:19.426000000 +0000 4
```
 
Пояснение: после исправления `Result=success`, `ExecMainStatus=0`, `lab-units.service` пропал из списка `--failed`, а файлы в `/home/labrunner/units_lab` обновились. Таймер продолжает работать: после исправления он запустил service в 13:05:28 и 13:10:19, и файлы принадлежат `labrunner`.
 
---
 
## Критерий приёмки
 
| Требование | Где подтверждено |
|---|---|
| Полный текст скрипта | пункт 7 (итоговая версия), пункт 1 (первая версия) |
| Содержимое service unit | пункты 2 и 7 |
| Содержимое timer unit | пункт 3 |
| Команды `daemon-reload`, запуска и включения timer | пункт 3 (`daemon-reload`, `enable --now lab-units.timer`) |
| Вывод, подтверждающий работу service | пункты 2, 7 и 10 (`status`, `Finished`, `status=0/SUCCESS`) |
| Вывод, подтверждающий расписание timer | пункты 3 и 10 (`status`, `list-timers`, `is-enabled`) |
| Пользователь, от которого запускается скрипт | пункты 4–7 (`User=` пуст → root; затем `User=labrunner`, `id labrunner`) |
| Записи из journal о запуске | пункт 9 |
| Пример намеренно созданной ошибки | пункт 10 (несуществующий путь в `ExecStart`) |
| Вывод, по которому определена причина `failed` | пункт 10 (`status=203/EXEC`, `Unable to locate executable ... No such file or directory`) |
| Подтверждение успешной работы после исправления | пункт 10 (`Result=success`, запуски таймера в 13:05:28 и 13:10:19) |
 
