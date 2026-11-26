# Журналы

1) Посмотрите журнал службы SSH.
2) Выведите журнал системы в реальном времени.
3) Выведите в реальном времени журнал только службы SSH.
4) Отфильтруйте журнал службы SSH за текущую загрузку системы.
5) Покажите сообщения не ниже уровня `warning`.
6) Можно ли получить доступ к журналам systemd без команды `journalctl`? Где journald хранит данные и почему это не обычный текстовый лог?
7) Найдите в журнале событие собственного подключения по SSH.
8) Сколько будет 2 - 2?

## Критерий приёмки

В отчёте должны быть:
- команда и вывод журнала службы SSH;
- команда просмотра журнала в реальном времени;
- команда фильтрации журнала только по SSH;
- пример вывода журнала за текущую загрузку;
- пример фильтрации по уровню `warning` или выше;
- найденная запись о собственном SSH-подключении;
- краткое пояснение, чем журнал journald отличается от обычного текстового лог-файла.


## Отчет по практической работе: Журналы

## 1. Журнал службы SSH

Использованные команды:

```bash
systemctl list-units --type=service --all | grep -i ssh
journalctl -u sshd.service --no-pager -n 20
```

Полученный результат:

```bash
[student@student-13 ~]$ systemctl list-units --type=service --all | grep -i ssh
  sshd.service     loaded    active   running OpenSSH server daemon
[student@student-13 ~]$ journalctl -u sshd.service --no-pager -n 20
Nov 24 19:40:46 student-13 sshd[1818]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 24 19:40:52 student-13 sshd[1818]: pam_tcb(sshd:auth): Authentication passed for student from (uid=0)
Nov 24 19:40:52 student-13 sshd[1818]: Accepted password for student from 192.168.6.83 port 34718 ssh2
Nov 24 19:40:52 student-13 sshd[1818]: pam_tcb(sshd:session): Session opened for student by (uid=0)
Nov 24 19:57:32 student-13 sshd[2049]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 24 19:57:39 student-13 sshd[2049]: pam_tcb(sshd:auth): Authentication passed for student from (uid=0)
Nov 24 19:57:39 student-13 sshd[2049]: Accepted password for student from 192.168.6.83 port 49430 ssh2
Nov 24 19:57:39 student-13 sshd[2049]: pam_tcb(sshd:session): Session opened for student by (uid=0)
Nov 24 20:09:39 student-13 sshd[2335]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 24 20:09:45 student-13 sshd[2335]: pam_tcb(sshd:auth): Authentication passed for student from (uid=0)
Nov 24 20:09:45 student-13 sshd[2335]: Accepted password for student from 192.168.6.83 port 42438 ssh2
Nov 24 20:09:45 student-13 sshd[2335]: pam_tcb(sshd:session): Session opened for student by (uid=0)
Nov 25 12:56:28 student-13 sshd[6046]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 25 12:56:36 student-13 sshd[6046]: pam_tcb(sshd:auth): Authentication passed for student from (uid=0)
Nov 25 12:56:36 student-13 sshd[6046]: Accepted password for student from 192.168.6.83 port 34554 ssh2
Nov 25 12:56:36 student-13 sshd[6046]: pam_tcb(sshd:session): Session opened for student by (uid=0)
Nov 26 09:04:58 student-13 sshd[10351]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 26 09:05:03 student-13 sshd[10351]: pam_tcb(sshd:auth): Authentication passed for student from (uid=0)
Nov 26 09:05:03 student-13 sshd[10351]: Accepted password for student from 192.168.6.83 port 43994 ssh2
Nov 26 09:05:03 student-13 sshd[10351]: pam_tcb(sshd:session): Session opened for student by (uid=0)
```

**Пояснение:** служба SSH называется `sshd.service`. `journalctl -u` показывает записи только этого юнита. Перед каждым успешным входом sshd пишет `error: PAM: Application needs to call libpam again`. Это штатное сообщение при входе по паролю через PAM, после него вход проходит успешно (`Accepted password`).

## 2. Журнал системы в реальном времени

Использованная команда:

```bash
(sleep 3; logger "lab-journal-test") & timeout 8 journalctl -f -n 5 --no-pager
```

Полученный результат:

```bash
[student@student-13 ~]$ (sleep 3; logger "lab-journal-test") & timeout 8 journalctl -f -n 5 --no-pager
[1] 10409
Nov 26 09:05:04 student-13 systemd[10368]: Reached target basic.target - Basic System.
Nov 26 09:05:04 student-13 systemd[10368]: Reached target default.target - Main User Target.
Nov 26 09:05:04 student-13 systemd[10368]: Startup finished in 182ms.
Nov 26 09:05:04 student-13 systemd[1]: Started user@1000.service - User Manager for UID 1000.
Nov 26 09:05:04 student-13 systemd[1]: Started session-103.scope - Session 103 of User student.
[1]+  Done                    ( sleep 3; logger "lab-journal-test" )
Nov 26 09:05:45 student-13 student[10409]: lab-journal-test
```

**Пояснение:** ключ `-f` (follow) не завершается после вывода последних записей, а ждёт новых. Для демонстрации через 3 секунды командой `logger` отправлено тестовое сообщение `lab-journal-test`, и оно появилось в потоке. `-n 5` задаёт число последних записей, `timeout 8` завершает просмотр через 8 секунд.

## 3. Журнал только службы SSH в реальном времени

Использованная команда:

```bash
timeout 40 journalctl -f -u sshd.service -n 3 --no-pager
```

Во время работы команды было открыто второе SSH-подключение к серверу.

Полученный результат:

```bash
[student@student-13 ~]$ timeout 40 journalctl -f -u sshd.service -n 3 --no-pager
Nov 26 09:05:03 student-13 sshd[10351]: pam_tcb(sshd:auth): Authentication passed for student from (uid=0)
Nov 26 09:05:03 student-13 sshd[10351]: Accepted password for student from 192.168.6.83 port 43994 ssh2
Nov 26 09:05:03 student-13 sshd[10351]: pam_tcb(sshd:session): Session opened for student by (uid=0)
Nov 26 09:06:45 student-13 sshd[10418]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 26 09:06:50 student-13 sshd[10418]: pam_tcb(sshd:auth): Authentication passed for student from (uid=0)
Nov 26 09:06:50 student-13 sshd[10418]: Accepted password for student from 192.168.6.83 port 35762 ssh2
Nov 26 09:06:50 student-13 sshd[10418]: pam_tcb(sshd:session): Session opened for student by (uid=0)
```

**Пояснение:** комбинация `-f -u sshd.service` показывает в реальном времени только записи SSH. Первые три строки это последние записи на момент запуска, остальные появились в потоке при новом подключении (`sshd[10418]`, порт 35762).

## 4. Журнал службы SSH за текущую загрузку

Использованные команды:

```bash
journalctl --list-boots --no-pager | tail -n 3
journalctl -b -u sshd.service --no-pager -n 15
```

Полученный результат:

```bash
[student@student-13 ~]$ journalctl --list-boots --no-pager | tail -n 3
 -2 9895dd0c86664ddab089c16702839450 Sun 2026-11-15 12:25:53 UTC Sun 2026-11-15 22:56:06 UTC
 -1 89d603aa6ed2459dacc7b0ee3d17a123 Sun 2026-11-15 22:56:40 UTC Tue 2026-11-24 11:16:30 UTC
  0 2ead8eb678d64a6b817f764850a1ea20 Tue 2026-11-24 11:16:45 UTC Thu 2026-11-26 09:07:36 UTC
[student@student-13 ~]$ journalctl -b -u sshd.service --no-pager -n 15
Nov 24 20:09:45 student-13 sshd[2335]: pam_tcb(sshd:auth): Authentication passed for student from (uid=0)
Nov 24 20:09:45 student-13 sshd[2335]: Accepted password for student from 192.168.6.83 port 42438 ssh2
Nov 24 20:09:45 student-13 sshd[2335]: pam_tcb(sshd:session): Session opened for student by (uid=0)
Nov 25 12:56:28 student-13 sshd[6046]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 25 12:56:36 student-13 sshd[6046]: pam_tcb(sshd:auth): Authentication passed for student from (uid=0)
Nov 25 12:56:36 student-13 sshd[6046]: Accepted password for student from 192.168.6.83 port 34554 ssh2
Nov 25 12:56:36 student-13 sshd[6046]: pam_tcb(sshd:session): Session opened for student by (uid=0)
Nov 26 09:04:58 student-13 sshd[10351]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 26 09:05:03 student-13 sshd[10351]: pam_tcb(sshd:auth): Authentication passed for student from (uid=0)
Nov 26 09:05:03 student-13 sshd[10351]: Accepted password for student from 192.168.6.83 port 43994 ssh2
Nov 26 09:05:03 student-13 sshd[10351]: pam_tcb(sshd:session): Session opened for student by (uid=0)
Nov 26 09:06:45 student-13 sshd[10418]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 26 09:06:50 student-13 sshd[10418]: pam_tcb(sshd:auth): Authentication passed for student from (uid=0)
Nov 26 09:06:50 student-13 sshd[10418]: Accepted password for student from 192.168.6.83 port 35762 ssh2
Nov 26 09:06:50 student-13 sshd[10418]: pam_tcb(sshd:session): Session opened for student by (uid=0)
```

**Пояснение:** ключ `-b` ограничивает вывод текущей загрузкой (в списке она имеет номер `0` и началась `Tue 2026-11-24 11:16:45 UTC`). Список загрузок также показывает, что журнал сохраняется между перезагрузками.

## 5. Сообщения не ниже уровня `warning`

Использованная команда:

```bash
journalctl -b -p warning --no-pager -n 15
```

Полученный результат:

```bash
[student@student-13 ~]$ journalctl -b -p warning --no-pager -n 15
Nov 24 13:04:25 student-13 systemd[1]: mdmonitor.service: Failed with result 'exit-code'.
Nov 24 13:04:25 student-13 systemd[1]: mdmonitor.service: Failed with result 'exit-code'.
Nov 24 13:05:37 student-13 systemd[1]: mdmonitor.service: Failed with result 'exit-code'.
Nov 24 13:07:09 student-13 kernel: md/raid1:md0: Disk failure on loop1, disabling device.
                                   md/raid1:md0: Operation continuing on 1 devices.
Nov 24 19:40:46 student-13 sshd[1818]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 24 19:57:32 student-13 sshd[2049]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 24 20:09:39 student-13 sshd[2335]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 24 20:18:36 student-13 systemd[1]: lab-units.service: Failed with result 'exit-code'.
Nov 24 20:18:36 student-13 systemd[1]: Failed to start lab-units.service - Lab script: create units_lab directory with files 1-4.
Nov 25 12:56:28 student-13 sshd[6046]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 25 13:00:15 student-13 (_typo.sh)[6199]: lab-units.service: Failed at step EXEC spawning /usr/local/bin/lab_units_typo.sh: No such file or directory
Nov 25 13:00:15 student-13 systemd[1]: lab-units.service: Failed with result 'exit-code'.
Nov 25 13:00:15 student-13 systemd[1]: Failed to start lab-units.service - Lab script: create units_lab directory with files 1-4.
Nov 26 09:04:58 student-13 sshd[10351]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
Nov 26 09:06:45 student-13 sshd[10418]: error: PAM: Application needs to call libpam again for student from 192.168.6.83
```

**Пояснение:** ключ `-p warning` выводит записи уровня `warning` и выше (`warning`, `err`, `crit`, `alert`, `emerg`), информационные сообщения отфильтрованы. В выводе присутствуют сбои служб (`mdmonitor.service`, `lab-units.service`), сообщение ядра об отключении диска `loop1` в RAID и сообщения SSH про PAM, которые sshd пишет с уровнем `error`.

## 6. Доступ к журналам без `journalctl`. Где journald хранит данные

Использованные команды:

```bash
sudo ls -laR /var/log/journal | head -n 15
ls -la /run/log/journal
journalctl --disk-usage; grep -v '^#' /etc/systemd/journald.conf | grep -v '^$'
sudo head -c 8 /var/log/journal/*/system.journal | od -c
sudo grep -a -m 1 -o 'MESSAGE=lab-journal-test' /var/log/journal/*/system.journal
sudo ls /var/log/journal/*/ | grep -v '^system@'
sudo grep -a -l 'MESSAGE=lab-journal-test' /var/log/journal/*/*.journal
sudo grep -a -m 1 -o 'MESSAGE=Accepted password for student' /var/log/journal/*/system.journal
```

Полученный результат:

```bash
[student@student-13 ~]$ sudo ls -laR /var/log/journal | head -n 15
/var/log/journal:
total 20
drwxr-sr-x+ 3 root systemd-journal 4096 Sep 14 22:38 .
drwxr-xr-x  4 root root            4096 Sep 14 22:38 ..
drwxr-sr-x+ 2 root systemd-journal 4096 Nov 24 20:18 b4117562a5d847bb8715163d2eebbfc2

/var/log/journal/b4117562a5d847bb8715163d2eebbfc2:
total 79988
drwxr-sr-x+ 2 root systemd-journal     4096 Nov 24 20:18 .
drwxr-sr-x+ 3 root systemd-journal     4096 Sep 14 22:38 ..
-rw-r-----+ 1 root systemd-journal 16777216 Sep 25 04:01 system@55f1fe03bef8432dbeabf4dbc386bf4c-0000000000000001-00065b7916ded418.journal
-rw-r-----+ 1 root systemd-journal 16777216 Oct  7 03:01 system@55f1fe03bef8432dbeabf4dbc386bf4c-0000000000003555-00065c46c452c8f7.journal
-rw-r-----+ 1 root systemd-journal 16777216 Oct 19 01:01 system@55f1fe03bef8432dbeabf4dbc386bf4c-0000000000006d44-00065d3753e31a6f.journal
-rw-r-----+ 1 root systemd-journal 16777216 Oct 29 06:01 system@55f1fe03bef8432dbeabf4dbc386bf4c-000000000000a4d9-00065e270c2ecb4c.journal
-rw-r-----+ 1 root systemd-journal 25165824 Nov 26 09:07 system.journal
[student@student-13 ~]$ ls -la /run/log/journal
total 0
drwxr-sr-x+ 2 root systemd-journal 40 Nov 24 11:16 .
drwxr-xr-x  3 root root            60 Nov 24 11:16 ..
[student@student-13 ~]$ journalctl --disk-usage; grep -v '^#' /etc/systemd/journald.conf | grep -v '^$'
Archived and active journals take up 78M in the file system.
grep: /etc/systemd/journald.conf: No such file or directory
[student@student-13 ~]$ sudo head -c 8 /var/log/journal/*/system.journal | od -c
0000000   L   P   K   S   H   H   R   H
0000010
[student@student-13 ~]$ sudo grep -a -m 1 -o 'MESSAGE=lab-journal-test' /var/log/journal/*/system.journal
[student@student-13 ~]$ sudo ls /var/log/journal/*/ | grep -v '^system@'
system.journal
user-1000@55f1fe03bef8432dbeabf4dbc386bf4c-00000000000006f5-00065b7e33c96fa6.journal
user-1000.journal
user-1002.journal
[student@student-13 ~]$ sudo grep -a -l 'MESSAGE=lab-journal-test' /var/log/journal/*/*.journal
/var/log/journal/b4117562a5d847bb8715163d2eebbfc2/user-1000.journal
[student@student-13 ~]$ sudo grep -a -m 1 -o 'MESSAGE=Accepted password for student' /var/log/journal/*/system.journal
MESSAGE=Accepted password for student
```

**Пояснение:**

- **Можно ли без `journalctl`?** Технически да: файлы журнала лежат на диске, и `grep -a` смог найти в них строки (`MESSAGE=Accepted password for student`). Но это «сырые» бинарные данные, а удобного разбора записей без `journalctl` нет. Доступ ограничен правами (`-rw-r-----`, группа `systemd-journal`), поэтому понадобился `sudo`.
- **Где хранятся данные?** В каталоге `/var/log/journal/<machine-id>/` (здесь `b4117562a5d847bb8715163d2eebbfc2`): активный `system.journal` и архивные `system@....journal` (ротация, по 16 МБ). Записи пользователей лежат в отдельных файлах `user-<UID>.journal`: тестовая запись `lab-journal-test` найдена в `user-1000.journal`, а не в `system.journal`. Общий размер журнала 78 МБ. Каталог `/run/log/journal` (временное хранилище в памяти) пуст, так как журнал хранится постоянно на диске. Файла `/etc/systemd/journald.conf` в системе нет, то есть используются настройки по умолчанию.
- **Почему не обычный текстовый лог?** Файл начинается с сигнатуры `LPKSHHRH`, то есть это бинарный формат journald, а не текст. Запись в нём структурирована: набор полей `ПОЛЕ=значение` (например `MESSAGE=...`), поэтому `journalctl` умеет быстро фильтровать по службе (`-u`), загрузке (`-b`), уровню (`-p`) и времени, чего у обычного текстового файла `.log` нет. Кроме того, журнал собирает сообщения ядра, служб и пользовательских сессий в одном месте.

## 7. Запись о собственном SSH-подключении

Использованные команды:

```bash
echo $SSH_CONNECTION
PORT=$(echo $SSH_CONNECTION | awk '{print $2}'); journalctl -u sshd.service --no-pager | grep "port $PORT "
```

Полученный результат:

```bash
[student@student-13 ~]$ echo $SSH_CONNECTION
192.168.6.83 43994 192.168.6.113 22
[student@student-13 ~]$ PORT=$(echo $SSH_CONNECTION | awk '{print $2}'); journalctl -u sshd.service --no-pager | grep "port $PORT "
Nov 26 09:05:03 student-13 sshd[10351]: Accepted password for student from 192.168.6.83 port 43994 ssh2
```

**Пояснение:** переменная `SSH_CONNECTION` текущей сессии содержит порт клиента `43994`. По этому порту в журнале найдена запись о собственном подключении: `Accepted password for student from 192.168.6.83 port 43994` (`Nov 26 09:05:03`, процесс `sshd[10351]`).

## 8. Сколько будет 2 − 2?

Использованная команда:

```bash
echo $((2 - 2))
```

Полученный результат:

```bash
[student@student-13 ~]$ echo $((2 - 2))
0
```

**Ответ:** 0.
