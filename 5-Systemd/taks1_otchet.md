# Юниты

1) Что такое systemd unit?
2) Проверьте статус любого systemd unit. Какую информацию выводит эта команда?
3) Остановите выбранный сервис.
4) Запустите его снова.
5) Уберите сервис из автозагрузки.
6) Верните его в автозагрузку.
7) Чем отличаются состояния `active`, `enabled`, `disabled`, `failed`?
8) Что такое systemd timers?
9) Найдите хотя бы один существующий timer в системе и определите, какой unit он запускает.

## Критерий приёмки

В отчёте должны быть:
- имя выбранного unit;
- вывод его состояния до изменений;
- команды остановки и запуска;
- вывод, подтверждающий отключение и повторное включение автозапуска;
- примеры состояний `active`, `enabled`, `disabled`, `failed`;
- список доступных timers;
- имя одного timer и unit, который он запускает;
- краткое пояснение разницы между текущим состоянием сервиса и настройкой автозапуска.

# Отчёт по практической работе: Юниты

## 1. Что такое systemd unit

**systemd unit** — это объект, которым управляет systemd, описанный в конфигурационном файле (unit-файле). Unit описывает ресурс или задачу: службу, точку монтирования, сокет, таймер, группу других unit-ов и т. д. Тип unit-а определяется расширением файла:

| Тип | Назначение |
|-----|-----------|
| `.service` | служба (демон или одноразовая задача) |
| `.timer` | запуск другого unit-а по расписанию |
| `.socket` | сокет, при обращении к которому запускается служба |
| `.mount` | точка монтирования |
| `.target` | группа unit-ов (аналог уровня запуска) |

Unit-файлы пакетов лежат в `/usr/lib/systemd/system/`, а настройки администратора и ссылки автозапуска — в `/etc/systemd/system/`.

Версия systemd в системе:

```bash
[student@student-13 ~]$ systemctl --version | head -n 1
systemd 257 (257.9-alt1)
```

## 2. Статус unit и выбор сервиса

Список запущенных служб:

```bash
[student@student-13 ~]$ systemctl list-units --type=service --state=running --no-pager
  UNIT                     LOAD   ACTIVE SUB     DESCRIPTION
  crond.service            loaded active running Vixie Cron Daemon
  dbus.service             loaded active running D-Bus System Message Bus
  getty@tty1.service       loaded active running Getty on tty1
  qemu-guest-agent.service loaded active running QEMU Guest Agent
  sshd.service             loaded active running OpenSSH server daemon
  systemd-journald.service loaded active running Journal Service
  systemd-logind.service   loaded active running User Login Management
  systemd-networkd.service loaded active running Network Configuration
  systemd-resolved.service loaded active running Network Name Resolution
  systemd-udevd.service    loaded active running Rule-based Manager for Device Events and Files
  systemd-userdbd.service  loaded active running User Database Manager
  user@1000.service        loaded active running User Manager for UID 1000

12 loaded units listed.
```

Для экспериментов выбран unit **`crond.service`** (планировщик cron). Его кратковременная остановка безопасна; `sshd` и другие системные службы не трогали, чтобы не потерять доступ к серверу.

Состояние unit до изменений:

```bash
[student@student-13 ~]$ systemctl status crond.service --no-pager
● crond.service - Vixie Cron Daemon
     Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-11-24 11:16:54 UTC; 8h ago
 Invocation: c610aa39e5e942f2acf5804fdebddd5e
   Main PID: 661 (crond)
      Tasks: 1 (limit: 1728)
     Memory: 432K (peak: 1.3M)
        CPU: 150ms
     CGroup: /system.slice/crond.service
             └─661 /usr/sbin/crond -n

Nov 24 15:01:01 student-13 crond[1627]: pam_tcb(crond:session): Session opened for root by (uid=0)
Nov 24 15:01:01 student-13 crond[1627]: pam_tcb(crond:session): Session closed for root
Nov 24 16:01:01 student-13 crond[1667]: pam_tcb(crond:session): Session opened for root by (uid=0)
Nov 24 16:01:01 student-13 crond[1667]: pam_tcb(crond:session): Session closed for root
Nov 24 17:01:01 student-13 crond[1707]: pam_tcb(crond:session): Session opened for root by (uid=0)
Nov 24 17:01:01 student-13 crond[1707]: pam_tcb(crond:session): Session closed for root
Nov 24 18:01:01 student-13 crond[1748]: pam_tcb(crond:session): Session opened for root by (uid=0)
Nov 24 18:01:01 student-13 crond[1748]: pam_tcb(crond:session): Session closed for root
Nov 24 19:01:01 student-13 crond[1789]: pam_tcb(crond:session): Session opened for root by (uid=0)
Nov 24 19:01:01 student-13 crond[1789]: pam_tcb(crond:session): Session closed for root

[student@student-13 ~]$ systemctl is-active crond.service
active
[student@student-13 ~]$ systemctl is-enabled crond.service
enabled
```

**Какую информацию выводит `systemctl status`:**

- **Описание unit** (`Vixie Cron Daemon`) и значок состояния (`●` — активен, `○` — неактивен, `×` — ошибка).
- **Loaded** — загружен ли unit-файл, путь к нему, включён ли автозапуск (`enabled`/`disabled`) и предустановка дистрибутива (`preset`).
- **Active** — текущее состояние (`active (running)`) и время, с которого оно действует.
- **Invocation** — идентификатор текущего запуска unit-а.
- **Main PID** — PID главного процесса и команда.
- **Tasks, Memory, CPU** — количество задач, потребление памяти и процессорное время.
- **CGroup** — контрольная группа и дерево процессов сервиса.
- **Последние строки журнала** (journal) этого unit-а.

Исходное состояние `crond.service`: `active` и `enabled`.

## 3. Остановка сервиса

```bash
[student@student-13 ~]$ sudo systemctl stop crond.service
[student@student-13 ~]$ systemctl is-active crond.service
inactive
[student@student-13 ~]$ systemctl status crond.service --no-pager
○ crond.service - Vixie Cron Daemon
     Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; preset: enabled)
     Active: inactive (dead) since Tue 2026-11-24 19:59:20 UTC; 16s ago
   Duration: 8h 42min 25.187s
 Invocation: c610aa39e5e942f2acf5804fdebddd5e
    Process: 661 ExecStart=/usr/sbin/crond -n (code=exited, status=0/SUCCESS)
   Main PID: 661 (code=exited, status=0/SUCCESS)
   Mem peak: 1.3M
        CPU: 150ms

Nov 24 16:01:01 student-13 crond[1667]: pam_tcb(crond:session): Session closed for root
Nov 24 17:01:01 student-13 crond[1707]: pam_tcb(crond:session): Session opened for root by (uid=0)
Nov 24 17:01:01 student-13 crond[1707]: pam_tcb(crond:session): Session closed for root
Nov 24 18:01:01 student-13 crond[1748]: pam_tcb(crond:session): Session opened for root by (uid=0)
Nov 24 18:01:01 student-13 crond[1748]: pam_tcb(crond:session): Session closed for root
Nov 24 19:01:01 student-13 crond[1789]: pam_tcb(crond:session): Session opened for root by (uid=0)
Nov 24 19:01:01 student-13 crond[1789]: pam_tcb(crond:session): Session closed for root
Nov 24 19:59:20 student-13 systemd[1]: Stopping crond.service - Vixie Cron Daemon...
Nov 24 19:59:20 student-13 systemd[1]: crond.service: Deactivated successfully.
Nov 24 19:59:20 student-13 systemd[1]: Stopped crond.service - Vixie Cron Daemon.
```

Сервис остановлен: `inactive (dead)`. При этом в строке `Loaded` по-прежнему указано `enabled` — остановка не влияет на автозапуск.

## 4. Запуск сервиса

```bash
[student@student-13 ~]$ sudo systemctl start crond.service
[student@student-13 ~]$ systemctl is-active crond.service
active
[student@student-13 ~]$ systemctl status crond.service --no-pager
● crond.service - Vixie Cron Daemon
     Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-11-24 20:00:13 UTC; 14s ago
 Invocation: b85fc65a19134b4f93dbf4ec3790ed8e
   Main PID: 2116 (crond)
      Tasks: 1 (limit: 1728)
     Memory: 300K (peak: 1M)
        CPU: 8ms
     CGroup: /system.slice/crond.service
             └─2116 /usr/sbin/crond -n

Nov 24 20:00:13 student-13 systemd[1]: Started crond.service - Vixie Cron Daemon.
```

Сервис снова `active (running)`. У него новый PID (2116 вместо 661) и новый Invocation ID — запущен новый процесс.

## 5. Отключение автозапуска

```bash
[student@student-13 ~]$ sudo systemctl disable crond.service
Synchronizing state of crond.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install disable crond
Removed '/etc/systemd/system/multi-user.target.wants/crond.service'.
[student@student-13 ~]$ systemctl is-enabled crond.service
disabled
[student@student-13 ~]$ systemctl is-active crond.service
active
```

Команда `disable` удалила ссылку `/etc/systemd/system/multi-user.target.wants/crond.service`, поэтому при следующей загрузке системы сервис автоматически не запустится. При этом **сейчас сервис продолжает работать** (`active`): `disable` не останавливает уже запущенную службу.

## 6. Возврат в автозагрузку

```bash
[student@student-13 ~]$ sudo systemctl enable crond.service
Synchronizing state of crond.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable crond
Created symlink '/etc/systemd/system/multi-user.target.wants/crond.service' → '/usr/lib/systemd/system/crond.service'.
[student@student-13 ~]$ systemctl is-enabled crond.service
enabled
[student@student-13 ~]$ systemctl is-active crond.service
active
```

Символическая ссылка создана заново, автозапуск включён, сервис работает. Исходное состояние `crond.service` (`active` + `enabled`) восстановлено.

## 7. Состояния `active`, `enabled`, `disabled`, `failed`

Эти слова описывают два разных свойства unit-а:

- **Текущее состояние** (команда `systemctl is-active`): `active`, `inactive`, `failed`.
- **Настройка автозапуска** (команда `systemctl is-enabled`): `enabled`, `disabled`, `static` и др.

| Состояние | Значение | Пример из этой системы |
|-----------|----------|------------------------|
| `active` | unit сейчас запущен и работает | `crond.service` — `active (running)` |
| `enabled` | автозапуск включён: unit будет запущен при загрузке (есть ссылка в `*.wants`) | `crond.service` — `enabled` |
| `disabled` | автозапуск выключен: при загрузке unit сам не запустится, но его можно запустить вручную | `crond.service` после `systemctl disable`; `console-getty.service` — `disabled` |
| `failed` | unit попытался запуститься или работал, но завершился с ошибкой | `mdmonitor.service` — `failed` |

Пример состояния `failed`:

```bash
[student@student-13 ~]$ systemctl --failed --no-pager
  UNIT              LOAD   ACTIVE SUB    DESCRIPTION
● mdmonitor.service loaded failed failed MD array monitor

1 loaded units listed.
[student@student-13 ~]$ systemctl status mdmonitor.service --no-pager
× mdmonitor.service - MD array monitor
     Loaded: loaded (/usr/lib/systemd/system/mdmonitor.service; static)
     Active: failed (Result: exit-code) since Tue 2026-11-24 13:05:37 UTC; 6h ago
   Duration: 12ms
 Invocation: 45c946fca9834b48ab2246b1eca46245
       Docs: man:mdadm(8)
   Main PID: 1431 (code=exited, status=1/FAILURE)
   Mem peak: 1M
        CPU: 5ms

Nov 24 13:05:37 student-13 systemd[1]: Started mdmonitor.service - MD array mo…tor.
Nov 24 13:05:37 student-13 mdadm[1431]: mdadm: No mail address or alert comman…ing.
Nov 24 13:05:37 student-13 systemd[1]: mdmonitor.service: Main process exited,…LURE
Nov 24 13:05:37 student-13 systemd[1]: mdmonitor.service: Failed with result '…de'.
Hint: Some lines were ellipsized, use -l to show in full.
[student@student-13 ~]$ systemctl is-active mdmonitor.service
failed
[student@student-13 ~]$ systemctl is-enabled mdmonitor.service
static
```

`mdmonitor.service` завершился с кодом 1 (`status=1/FAILURE`). Из журнала видно, что для монитора MD-массивов не задан почтовый адрес или команда оповещения (`mdadm: No mail address or alert comman…`). Для лабораторной работы сервис не исправлялся.

Пример `disabled` + `inactive` (автозапуск отключён, сервис не работает):

```bash
[student@student-13 ~]$ systemctl is-enabled console-getty.service
disabled
[student@student-13 ~]$ systemctl is-active console-getty.service
inactive
```

Дополнительно: у `mdmonitor.service` состояние автозапуска `static` — в unit-файле нет секции `[Install]`, поэтому командой `enable` его включить нельзя; такие unit-ы запускаются по зависимости от других unit-ов или вручную.

### Разница между текущим состоянием сервиса и настройкой автозапуска

- **Текущее состояние** (`active` / `inactive` / `failed`) отвечает на вопрос «работает ли сервис прямо сейчас». Его меняют команды `start` и `stop`.
- **Автозапуск** (`enabled` / `disabled`) отвечает на вопрос «запустится ли сервис при загрузке системы». Его меняют команды `enable` и `disable`.

Эти свойства независимы. В ходе работы это подтверждено на практике: после `stop` сервис был `inactive`, но оставался `enabled` (п. 3); после `disable` сервис оставался `active` (п. 5). Возможны все четыре сочетания, например `console-getty.service` — `disabled` и `inactive`, а `crond.service` — `enabled` и `active`.

## 8. Что такое systemd timers

**systemd timer** — это unit типа `.timer`, который запускает по расписанию другой unit (обычно `.service`). Это аналог cron, встроенный в systemd. По умолчанию timer `имя.timer` запускает unit с тем же именем `имя.service`; другой unit можно указать параметром `Unit=`.

Расписание задаётся двумя способами:

- **по календарю** — параметр `OnCalendar=` (например, ежедневно в заданное время);
- **монотонные таймеры** — относительно событий: `OnBootSec=` (через время после загрузки), `OnUnitActiveSec=` (через время после последней активации).

Преимущества по сравнению с cron: timer — обычный unit (управляется через `systemctl`, пишет журнал, имеет зависимости), а запускаемая служба имеет все возможности systemd.

## 9. Timers в системе

Список timers:

```bash
[student@student-13 ~]$ systemctl list-timers --all --no-pager --full
NEXT                        LEFT LAST                        PASSED UNIT                         ACTIVATES
Wed 2026-11-25 11:31:58 UTC  15h Tue 2026-11-24 11:31:58 UTC 8h ago systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service

1 timers listed.
```

Все установленные unit-файлы timers:

```bash
[student@student-13 ~]$ systemctl list-unit-files --type=timer --no-pager
UNIT FILE                    STATE    PRESET
fstrim.timer                 disabled disabled
mdadm-last-resort@.timer     static   -
systemd-tmpfiles-clean.timer static   -

3 unit files listed.
```

Активен один timer — `systemd-tmpfiles-clean.timer`. Остальные (`fstrim.timer`, `mdadm-last-resort@.timer`) не включены.

Содержимое unit-файла timer:

```bash
[student@student-13 ~]$ systemctl cat systemd-tmpfiles-clean.timer --no-pager
# /usr/lib/systemd/system/systemd-tmpfiles-clean.timer
#  SPDX-License-Identifier: LGPL-2.1-or-later
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Daily Cleanup of Temporary Directories
Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)
ConditionPathExists=!/etc/initrd-release

[Timer]
OnBootSec=15min
OnUnitActiveSec=1d
```

Какой unit запускает timer:

```bash
[student@student-13 ~]$ systemctl show systemd-tmpfiles-clean.timer -p Unit
Unit=systemd-tmpfiles-clean.service
```

**Результат:** timer `systemd-tmpfiles-clean.timer` запускает unit `systemd-tmpfiles-clean.service` (колонка `ACTIVATES` в `list-timers` и параметр `Unit` совпадают). Timer срабатывает через 15 минут после загрузки (`OnBootSec=15min`) и далее раз в сутки после предыдущей активации (`OnUnitActiveSec=1d`); служба очищает временные каталоги.
