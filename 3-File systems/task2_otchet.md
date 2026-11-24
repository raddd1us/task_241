# Григорьев Сергей 241 группа
# Структура каталогов

1) Какая структура каталогов используется в Linux? Выведите список объектов в корне системы.
2) Где обычно хранятся домашние каталоги обычных пользователей?
3) Где находится домашний каталог суперпользователя?
4) Где обычно хранятся основные системные конфигурационные файлы?
5) Объясните назначение каталогов:
   - `/bin`
   - `/sbin`
   - `/usr/bin`
   - `/usr/sbin`
   - `/etc`
   - `/var`
   - `/tmp`
   - `/home`
   - `/root`
   - `/proc`
   - `/sys`
6) Проверьте, являются ли `/bin` и `/sbin` реальными каталогами или символическими ссылками в вашей системе.
7) Найдите, где физически расположен исполняемый файл одной из команд, которой вы пользовались ранее.

## Критерий приёмки

В отчёте должны быть:
- список объектов в корне `/`;
- краткое назначение каталогов `/bin`, `/sbin`, `/usr/bin`, `/usr/sbin`, `/etc`, `/var`, `/tmp`, `/home`, `/root`, `/proc`, `/sys`;
- вывод, показывающий, являются ли `/bin` и `/sbin` каталогами или символическими ссылками;
- команда и результат поиска исполняемого файла выбранной команды;
- краткое пояснение различий между обычным каталогом, символической ссылкой и виртуальной файловой системой.

# Отчёт по практической работе: Структура каталогов


## 1. Структура каталогов в Linux и список объектов в корне

В Linux используется **единое иерархическое дерево каталогов**. У него один корень `/`, а все остальные файлы, каталоги, устройства и подключённые диски располагаются внутри него. Отдельных «дисков» вроде `C:` и `D:` нет: дополнительные разделы и носители монтируются в каталоги дерева (например, в `/mnt` или `/media`). Расположение основных каталогов описывает стандарт **FHS** (Filesystem Hierarchy Standard).

Список объектов в корне системы:

```bash
[student@student-13 ~]$ ls /
bin   dev  home  lib64   lost+found  mnt  proc  run   selinux  sys  usr
boot  etc  lib   libx32  media       opt  root  sbin  srv      tmp  var
```

Подробный вывод с типами и правами:

```bash
[student@student-13 ~]$ ls -l /
total 60
lrwxrwxrwx   1 root root     7 Apr  6  2024 bin -> usr/bin
drwx------   3 root root  4096 Oct  7  2025 boot
drwxr-xr-x  18 root root  3820 Nov 15 22:56 dev
drwxr-xr-x  70 root root  4096 Nov 23 11:55 etc
drwxr-xr-x   5 root root  4096 Nov 23 11:55 home
lrwxrwxrwx   1 root root     7 Apr  6  2024 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Apr  6  2024 lib64 -> usr/lib64
lrwxrwxrwx   1 root root    10 Apr  6  2024 libx32 -> usr/libx32
drwx------   2 root root 16384 Oct  7  2025 lost+found
drwxr-xr-x   2 root root  4096 Apr  6  2024 media
drwxr-xr-x   2 root root  4096 Nov 23 17:45 mnt
drwxr-xr-x   2 root root  4096 Apr  6  2024 opt
dr-xr-xr-x 167 root root     0 Nov 15 22:56 proc
drwxr-xr-x   8 root root  4096 Nov 16 13:37 root
drwxr-xr-x  19 root root   500 Nov 23 17:40 run
lrwxrwxrwx   1 root root     8 Apr  6  2024 sbin -> usr/sbin
dr-xr-xr-x   2 root root  4096 Apr  6  2024 selinux
drwxr-xr-x   2 root root  4096 Apr  6  2024 srv
dr-xr-xr-x  13 root root     0 Nov 22 16:23 sys
drwxrwxrwt   8 root root   160 Nov 23 17:45 tmp
drwxr-xr-x  14 root root  4096 Oct  7  2025 usr
drwxr-xr-x  16 root root  4096 Oct  7  2025 var
```

Из вывода видно, что `/bin`, `/sbin`, `/lib`, `/lib64` и `/libx32` в этой системе являются символическими ссылками на каталоги внутри `/usr` (первый символ в правах — `l`, после имени стоит `->`). У `/proc` и `/sys` размер `0` — это виртуальные файловые системы.

## 2. Где хранятся домашние каталоги обычных пользователей

Домашние каталоги обычных пользователей хранятся в **`/home`**, по одному каталогу на пользователя: `/home/<имя_пользователя>`.

```bash
[student@student-13 ~]$ ls /home
student  user1  user2
[student@student-13 ~]$ echo $HOME
/home/student
```

## 3. Домашний каталог суперпользователя

Домашний каталог `root` находится в **`/root`**, а не в `/home`. Это сделано для того, чтобы администратор мог работать, даже если раздел `/home` не смонтирован.

```bash
[student@student-13 ~]$ getent passwd root
root:x:0:0:System Administrator:/root:/bin/bash
[student@student-13 ~]$ ls -ld /root
drwxr-xr-x 8 root root 4096 Nov 16 13:37 /root
```

## 4. Где хранятся основные системные конфигурационные файлы

Основные системные конфигурационные файлы хранятся в каталоге **`/etc`**. Это в основном текстовые файлы, которые можно редактировать вручную с правами администратора.

```bash
[student@student-13 ~]$ ls -d /etc/passwd /etc/group /etc/fstab /etc/hostname /etc/ssh
ls: cannot access '/etc/ssh': No such file or directory
/etc/fstab  /etc/group  /etc/hostname  /etc/passwd
```

Каталога `/etc/ssh` в этой системе нет: конфигурация SSH находится в `/etc/openssh`.

```bash
[student@student-13 ~]$ ls -d /etc/openssh
/etc/openssh
[student@student-13 ~]$ ls -l /etc/shadow
-r-------- 1 root root 535 Oct  7  2025 /etc/shadow
```

Примеры: `/etc/passwd` и `/etc/group` — учётные записи и группы, `/etc/shadow` — хеши паролей (читать может только root), `/etc/fstab` — таблица монтирования, `/etc/openssh/` — настройки SSH.

## 5. Назначение каталогов

| Каталог | Назначение |
|---|---|
| `/bin` | Основные исполняемые файлы команд, нужные всем пользователям и для работы системы (`ls`, `cp`, `cat`). В этой системе — символическая ссылка на `/usr/bin` |
| `/sbin` | Системные программы для администрирования, обычно запускаются от root. В этой системе — символическая ссылка на `/usr/sbin` |
| `/usr/bin` | Основная масса исполняемых файлов пользовательских программ и команд, установленных из пакетов |
| `/usr/sbin` | Административные программы, не нужные при обычной работе пользователя |
| `/etc` | Системные конфигурационные файлы (учётные записи, сеть, службы, монтирование) |
| `/var` | Изменяемые данные: журналы (`/var/log`), кэш, очереди, базы данных, данные пакетных менеджеров |
| `/tmp` | Временные файлы всех пользователей и программ; доступен на запись всем (права `drwxrwxrwt`) |
| `/home` | Домашние каталоги обычных пользователей с их личными файлами и настройками |
| `/root` | Домашний каталог суперпользователя `root` |
| `/proc` | Виртуальная файловая система (procfs): информация о процессах и состоянии ядра (`/proc/cpuinfo`, `/proc/<PID>`) |
| `/sys` | Виртуальная файловая система (sysfs): информация об устройствах, драйверах и подсистемах ядра |

## 6. `/bin` и `/sbin`: каталоги или символические ссылки

```bash
[student@student-13 ~]$ ls -ld /bin /sbin /usr/bin /usr/sbin
lrwxrwxrwx 1 root root     7 Apr  6  2024 /bin -> usr/bin
lrwxrwxrwx 1 root root     8 Apr  6  2024 /sbin -> usr/sbin
drwxr-xr-x 2 root root 20480 Oct  7  2025 /usr/bin
drwxr-xr-x 2 root root 12288 Oct  7  2025 /usr/sbin
[student@student-13 ~]$ readlink -f /bin
/usr/bin
[student@student-13 ~]$ readlink -f /sbin
/usr/sbin
```

**Вывод:** в моей системе `/bin` и `/sbin` — это **символические ссылки** на `/usr/bin` и `/usr/sbin` соответственно (первый символ в правах — `l`, после имени стоит `->`). Реальными каталогами являются `/usr/bin` и `/usr/sbin` (первый символ — `d`). Все исполняемые файлы физически лежат в `/usr`, а старые пути `/bin` и `/sbin` сохранены для совместимости.

## 7. Поиск исполняемого файла команды

Выбрана команда `cat`.

```bash
[student@student-13 ~]$ which cat
/usr/bin/cat
[student@student-13 ~]$ type -a cat
cat is /usr/bin/cat
cat is /bin/cat
[student@student-13 ~]$ whereis cat
cat: /usr/bin/cat /usr/share/man/man1/cat.1.xz
[student@student-13 ~]$ readlink -f /bin/cat
/usr/bin/cat
[student@student-13 ~]$ rpm -qf $(which cat)
coreutils-9.4.0.75.7f2c97-alt1.x86_64
coreutils-9.4.0.75.7f2c97-alt1.x86_64
```

**Вывод:** физически исполняемый файл `cat` расположен в **`/usr/bin/cat`** и принадлежит пакету `coreutils`. Путь `/bin/cat` работает только потому, что `/bin` — символическая ссылка на `/usr/bin`. Вывод `rpm -qf` содержит одну и ту же строку дважды — это один и тот же пакет.

## 8. Различия между обычным каталогом, символической ссылкой и виртуальной файловой системой

| | Обычный каталог | Символическая ссылка | Виртуальная файловая система |
|---|---|---|---|
| Что это | Контейнер для файлов и подкаталогов, реально хранящийся на диске | Специальный файл, содержащий путь к другому файлу или каталогу | Файловая система, существующая только в оперативной памяти и создаваемая ядром «на лету» |
| Где хранятся данные | На блочном устройстве (диск, раздел) | В самой ссылке хранится только путь-указатель, данных цели в ней нет | Не хранятся на диске: содержимое формирует ядро при обращении |
| Тип в `ls -l` | `d` | `l` (`/bin -> usr/bin`) | Обычно каталог `d` (`/proc`, `/sys`), но смонтирован отдельным типом ФС |
| Размер | Занимает место на диске (минимум блок) | Равен длине пути (`/bin` → 7 байт) | `0` |
| Если цель удалить | Данные теряются | Ссылка становится «битой» | Содержимое отражает текущее состояние ядра и меняется само |
| Примеры | `/usr/bin`, `/etc`, `/home` | `/bin`, `/sbin`, `/lib` | `/proc` (procfs), `/sys` (sysfs) |

Проверка того, что `/proc` и `/sys` являются виртуальными файловыми системами:

```bash
[student@student-13 ~]$ findmnt /proc
TARGET
      SOURCE
           FSTYPE OPTIONS
/proc proc proc   rw,nosuid,noexec,relatime,gid=19
[student@student-13 ~]$ findmnt /sys
TARGET
     SOURCE
           FSTYPE OPTIONS
/sys sysfs sysfs  rw,nosuid,nodev,noexec,relatime
```

Типы файловых систем `proc` и `sysfs` показывают, что это не каталоги на диске, а виртуальные файловые системы ядра.

```bash
[student@student-13 ~]$ ls -l /proc/cpuinfo
-r--r--r-- 1 root root 0 Nov 15 23:01 /proc/cpuinfo
[student@student-13 ~]$ head -n 3 /proc/cpuinfo
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
```

Файл `/proc/cpuinfo` имеет размер `0`, но при чтении выдаёт информацию о процессоре — она формируется ядром в момент обращения.
