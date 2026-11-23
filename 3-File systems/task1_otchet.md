# Григорьев Сергей 241 группа
# Файловые системы

1) Какие файловые системы вы знаете?
2) Как можно классифицировать файловые системы? В чём основные отличия?
3) Какие файловые системы обычно используются в Linux?
4) Как создать файловую систему на блочном устройстве?
5) Что такое монтирование и как подключить файловую систему в систему?
6) Рассмотрите `procfs`, `sysfs`, `tmpfs` и `cifs`.
   6.1) В чём особенности каждой из них?
   6.2) Найдите в своей системе каталоги, куда примонтированы эти файловые системы, если они используются.
7) Как получить информацию о системе, используя содержимое виртуальных файловых систем?
   7.1) выведите информацию о процессоре;
   7.2) выведите информацию о состоянии памяти.
8) Выведите список всех текущих точек монтирования минимум двумя разными способами.

## Критерий приёмки

В отчёте должны быть:
- перечень рассмотренных типов файловых систем;
- вывод, показывающий точки монтирования `procfs`, `sysfs`, `tmpfs` и `cifs`, если она используется;
- вывод информации о CPU через `/proc`;
- вывод информации о памяти через `/proc`;
- два разных способа вывести текущие точки монтирования;
- пример определения типа файловой системы для выбранной точки монтирования;
- краткое пояснение различий между обычной дисковой файловой системой и виртуальными файловыми системами `/proc` и `/sys`.

# Отчёт по практической работе: Файловые системы


## 1. Известные файловые системы

**Дисковые (локальные):**
- `ext2`, `ext3`, `ext4` — основное семейство Linux;
- `XFS`, `JFS`, `ReiserFS` — журналируемые файловые системы;
- `Btrfs`, `ZFS` — файловые системы с copy-on-write, снимками и контрольными суммами;
- `F2FS` — для флеш-накопителей;
- `FAT16/FAT32`, `exFAT`, `NTFS` — Windows и сменные носители;
- `APFS`, `HFS+` — macOS;
- `ISO 9660`, `UDF` — оптические диски.

**Сетевые:**
- `NFS`, `CIFS/SMB` — доступ к удалённым каталогам;
- `SSHFS` (на базе FUSE), `Ceph`, `GlusterFS` — распределённые и сетевые хранилища.

**Виртуальные (псевдо-):**
- `procfs` (`/proc`), `sysfs` (`/sys`), `tmpfs`, `devtmpfs` (`/dev`), `devpts`, `cgroup2`, `debugfs`, `securityfs`.

**Наслаиваемые и сжатые:**
- `overlayfs` (контейнеры), `squashfs` (образы Live-систем, snap).

## 2. Классификация файловых систем

| Признак | Классы | Отличия |
|---|---|---|
| Место хранения данных | дисковые, сетевые, виртуальные | дисковые хранят данные на блочном устройстве, сетевые обращаются к удалённому серверу, виртуальные существуют в ОЗУ или отражают структуры ядра |
| Отказоустойчивость | без журнала (`ext2`, `FAT`) и журналируемые (`ext4`, `XFS`, `NTFS`) | журнал позволяет быстро восстановить согласованность после сбоя |
| Способ записи | обычные и copy-on-write (`Btrfs`, `ZFS`) | CoW не перезаписывает блоки на месте, что даёт снимки и защиту от повреждений |
| Назначение | универсальные, для флеш-памяти, для носителей только для чтения | `F2FS` учитывает износ ячеек, `squashfs`/`ISO 9660` доступны только на чтение |
| Размещение | локальные и кластерные | кластерные и сетевые обслуживают одновременный доступ с нескольких узлов |
| Поддержка прав доступа | с правами POSIX (`ext4`, `XFS`) и без них (`FAT32`, `exFAT`) | на `FAT` нет владельцев и режимов, права задаются опциями монтирования |

## 3. Файловые системы, используемые в Linux

В Linux обычно используются:
- `ext4` — самая распространённая дисковая файловая система;
- `XFS`, `Btrfs` — для больших томов и расширенных возможностей (снимки, CoW);
- `vfat` — системный раздел EFI;
- `tmpfs`, `proc`, `sysfs`, `devtmpfs`, `cgroup2` — служебные виртуальные системы;
- `squashfs` — сжатые образы только для чтения;
- `nfs`, `cifs` — сетевые ресурсы.

Список типов, которые поддерживает ядро сервера:

```bash
[student@student-13 ~]$ cat /proc/filesystems
nodev   sysfs
nodev   tmpfs
nodev   bdev
nodev   proc
nodev   cgroup
nodev   cgroup2
nodev   devtmpfs
nodev   configfs
nodev   debugfs
nodev   tracefs
nodev   securityfs
nodev   sockfs
nodev   bpf
nodev   pipefs
nodev   ramfs
nodev   hugetlbfs
nodev   devpts
        ext3
        ext4
        squashfs
nodev   mqueue
nodev   pstore
nodev   autofs
        fuseblk
nodev   fuse
nodev   fusectl
```

Пометка `nodev` означает, что файловой системе не нужно блочное устройство. Из дисковых типов ядро сейчас поддерживает `ext3`, `ext4`, `squashfs` и `fuseblk` (FUSE поверх блочного устройства); остальные записи — виртуальные файловые системы.

## 4. Создание файловой системы на блочном устройстве

Общий вид: `mkfs.<тип> <устройство>` или `mkfs -t <тип> <устройство>`, например `mkfs.ext4 /dev/sdb1`. Команда уничтожает данные на устройстве, поэтому перед запуском нужно проверить его имя.

Блочные устройства на сервере:

```bash
[student@student-13 ~]$ lsblk -f
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda
└─sda1 ext4   1.0         93c203a1-7a96-4cd1-a5f8-ed900f7b23ab   13.2G     6% /
sdb
sdc
sr0
```

Чтобы не трогать диски `sdb` и `sdc`, файловая система создаётся на файле-образе, который ядро подключает как loop-устройство. Утилиты `mkfs.ext4`, `losetup` и `blkid` лежат в `/sbin`, которого нет в `PATH` обычного пользователя, поэтому они вызываются по полному пути через `sudo`.

Создание образа и файловой системы ext4:

```bash
[student@student-13 ~]$ truncate -s 64M /tmp/disk.img
[student@student-13 ~]$ ls -lh /tmp/disk.img
-rw-r--r-- 1 student student 64M Nov 23 17:39 /tmp/disk.img
[student@student-13 ~]$ sudo /sbin/mkfs.ext4 -F /tmp/disk.img
mke2fs 1.47.1 (20-May-2024)
Discarding device blocks: done
Creating filesystem with 65536 1k blocks and 16384 inodes
Filesystem UUID: e54116c2-6fad-4651-bc63-07b1228530fe
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```

Ключ `-F` нужен, потому что `mkfs` работает с обычным файлом, а не с блочным устройством.

Проверка результата:

```bash
[student@student-13 ~]$ sudo /sbin/blkid /tmp/disk.img
/tmp/disk.img: UUID="e54116c2-6fad-4651-bc63-07b1228530fe" BLOCK_SIZE="1024" TYPE="ext4"
```

Файловая система ext4 создана, её UUID: `e54116c2-6fad-4651-bc63-07b1228530fe`.

## 5. Монтирование

**Монтирование** — это подключение файловой системы к дереву каталогов. Каталог, в который она подключается, называется **точкой монтирования**. Содержимое файловой системы становится доступным по этому пути, а прежнее содержимое каталога на время монтирования скрывается. Общий вид команды: `mount -t <тип> -o <опции> <устройство> <каталог>`; отключение выполняет `umount`.

Создание точки монтирования и подключение образа:

```bash
[student@student-13 ~]$ sudo mkdir -p /mnt/test
[student@student-13 ~]$ sudo mount -o loop /tmp/disk.img /mnt/test
[student@student-13 ~]$ findmnt /mnt/test
TARGET    SOURCE     FSTYPE OPTIONS
/mnt/test /dev/loop0 ext4   rw,relatime
[student@student-13 ~]$ df -hT /mnt/test
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/loop0     ext4   55M   46K   51M   1% /mnt/test
```

Файловая система на `/dev/loop0` примонтирована в `/mnt/test`.

Запись тестового файла:

```bash
[student@student-13 ~]$ echo "hello" | sudo tee /mnt/test/file.txt
hello
[student@student-13 ~]$ ls -l /mnt/test
total 13
-rw-r--r-- 1 root root     6 Nov 23 17:41 file.txt
drwx------ 2 root root 12288 Nov 23 17:40 lost+found
```

Каталог `lost+found` ext4 создаёт автоматически при создании файловой системы.

Отключение и проверка:

```bash
[student@student-13 ~]$ sudo umount /mnt/test
[student@student-13 ~]$ findmnt /mnt/test; echo "код возврата findmnt: $?"
код возврата findmnt: 1
[student@student-13 ~]$ ls -l /mnt/test
total 0
[student@student-13 ~]$ sudo mount -o loop /tmp/disk.img /mnt/test
[student@student-13 ~]$ ls -l /mnt/test
total 13
-rw-r--r-- 1 root root     6 Nov 23 17:41 file.txt
drwx------ 2 root root 12288 Nov 23 17:40 lost+found
[student@student-13 ~]$ sudo umount /mnt/test
```

После `umount` каталог `/mnt/test` пуст: виден сам каталог на корневой файловой системе, а не содержимое образа. После повторного монтирования `file.txt` возвращается: файл всё это время хранился внутри `/tmp/disk.img`, а каталог служит только точкой подключения.

## 6. Файловые системы `procfs`, `sysfs`, `tmpfs`, `cifs`

### 6.1 Особенности

| ФС | Особенности |
|---|---|
| `procfs` | Монтируется в `/proc`. Интерфейс к информации о процессах (`/proc/<PID>/`) и о состоянии ядра (`cpuinfo`, `meminfo`, `mounts`). Файлы создаются ядром «на лету», на диске их нет. Часть параметров можно менять через `/proc/sys` (`sysctl`). |
| `sysfs` | Монтируется в `/sys`. Структурированное представление устройств, шин, драйверов и модулей ядра (`/sys/class`, `/sys/block`, `/sys/devices`). Один файл содержит одно значение. |
| `tmpfs` | Файловая система в оперативной памяти (при нехватке — в swap). Очень быстрая, размер ограничивается опцией `size=`, содержимое пропадает при размонтировании и перезагрузке. Используется для `/run`, `/dev/shm`, `/tmp`. |
| `cifs` | Клиент сетевого протокола SMB/CIFS для доступа к общим папкам Windows и Samba. Требует пакет `cifs-utils`, адрес сервера, имя пользователя и пароль. Права определяются сервером и опциями монтирования (`uid=`, `gid=`, `file_mode=`). |

### 6.2 Точки монтирования в системе

```bash
[student@student-13 ~]$ findmnt -t proc,sysfs,tmpfs,cifs
TARGET                            SOURCE FSTYPE OPTIONS
/proc                             proc   proc   rw,nosuid,noexec,relatime,gid=19
/dev/shm                          tmpfs  tmpfs  rw,nosuid,nodev,inode64
/run                              runfs  tmpfs  rw,relatime,mode=755,inode64
├─/run/credentials/systemd-journald.service
│                                 tmpfs  tmpfs  ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=1024,mo
├─/run/user/1000                  tmpfs  tmpfs  rw,nosuid,nodev,relatime,size=150256k,nr_inodes=37564,mode=700,uid=1000,
├─/run/credentials/systemd-resolved.service
│                                 tmpfs  tmpfs  ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=1024,mo
├─/run/credentials/systemd-networkd.service
│                                 tmpfs  tmpfs  ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=1024,mo
└─/run/credentials/getty@tty1.service
                                  tmpfs  tmpfs  ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=1024,mo
/sys                              sysfs  sysfs  rw,nosuid,nodev,noexec,relatime
/tmp                              tmpfs  tmpfs  rw,nosuid,relatime,inode64
```

- `procfs` смонтирована в `/proc`;
- `sysfs` смонтирована в `/sys`;
- `tmpfs` смонтирована в `/dev/shm`, `/run` (источник называется `runfs`, тип `tmpfs`), `/run/user/1000`, `/tmp` и в каталогах `/run/credentials/...`.

Проверка `cifs`:

```bash
[student@student-13 ~]$ findmnt -t cifs; echo "код возврата findmnt -t cifs: $?"
код возврата findmnt -t cifs: 0
[student@student-13 ~]$ grep cifs /proc/mounts; echo "код возврата grep: $?"
код возврата grep: 1
```

Вывод `findmnt -t cifs` пуст, а `grep` не нашёл `cifs` в `/proc/mounts` и вернул код 1. Значит, `cifs` в системе не используется: сетевые ресурсы SMB не подключены.

## 7. Информация о системе из виртуальных файловых систем

### 7.1 Информация о процессоре

```bash
[student@student-13 ~]$ head -n 15 /proc/cpuinfo
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 44
model name      : Intel(R) Xeon(R) CPU           X5675  @ 3.07GHz
stepping        : 2
microcode       : 0x1f
cpu MHz         : 3066.774
cache size      : 16384 KB
physical id     : 0
siblings        : 1
core id         : 0
cpu cores       : 1
apicid          : 0
initial apicid  : 0
[student@student-13 ~]$ grep -c '^processor' /proc/cpuinfo
1
```

Процессор: Intel Xeon X5675, 3.07 GHz, кэш 16384 КБ. Логический процессор один, ядро одно.

### 7.2 Состояние памяти

```bash
[student@student-13 ~]$ head -n 10 /proc/meminfo
MemTotal:        1502576 kB
MemFree:         1299148 kB
MemAvailable:    1286816 kB
Buffers:           19952 kB
Cached:            92168 kB
SwapCached:            0 kB
Active:            58904 kB
Inactive:          76196 kB
Active(anon):        792 kB
Inactive(anon):    23120 kB
[student@student-13 ~]$ grep -E 'MemTotal|MemFree|MemAvailable|SwapTotal|SwapFree' /proc/meminfo
MemTotal:        1502576 kB
MemFree:         1299148 kB
MemAvailable:    1286828 kB
SwapTotal:             0 kB
SwapFree:              0 kB
```

Основные поля: `MemTotal` — всего ОЗУ (около 1,4 ГиБ), `MemFree` — полностью свободная память, `MemAvailable` — память, реально доступная приложениям с учётом кэша, `SwapTotal` и `SwapFree` — подкачка. Раздела или файла подкачки на сервере нет (0 кБ). Значения `MemAvailable` в двух запусках немного отличаются, потому что `/proc/meminfo` показывает состояние памяти на момент чтения.

## 8. Список текущих точек монтирования

**Способ 1. `findmnt`** (дерево монтирования):

```bash
[student@student-13 ~]$ findmnt
TARGET                           SOURCE  FSTYPE     OPTIONS
/                                /dev/sda1
                                         ext4       rw,relatime
├─/proc                          proc    proc       rw,nosuid,noexec,relatime,gid=19
│ └─/proc/sys/fs/binfmt_misc     systemd-1
│                                        autofs     rw,relatime,fd=36,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe
├─/dev                           udevfs  devtmpfs   rw,relatime,size=5120k,nr_inodes=184459,mode=755,inode64
│ ├─/dev/shm                     tmpfs   tmpfs      rw,nosuid,nodev,inode64
│ ├─/dev/pts                     devpts  devpts     rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666
│ ├─/dev/mqueue                  mqueue  mqueue     rw,nosuid,nodev,noexec,relatime
│ └─/dev/hugepages               hugetlbfs
│                                        hugetlbfs  rw,nosuid,nodev,relatime,pagesize=2M
├─/run                           runfs   tmpfs      rw,relatime,mode=755,inode64
│ ├─/run/credentials/systemd-journald.service
│ │                              tmpfs   tmpfs      ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=102
│ ├─/run/user/1000               tmpfs   tmpfs      rw,nosuid,nodev,relatime,size=150256k,nr_inodes=37564,mode=700,uid=1
│ ├─/run/credentials/systemd-resolved.service
│ │                              tmpfs   tmpfs      ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=102
│ ├─/run/credentials/systemd-networkd.service
│ │                              tmpfs   tmpfs      ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=102
│ └─/run/credentials/getty@tty1.service
│                                tmpfs   tmpfs      ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=102
├─/sys                           sysfs   sysfs      rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/security         securityfs
│ │                                      securityfs rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/cgroup               cgroup2 cgroup2    rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot
│ ├─/sys/fs/pstore               pstore  pstore     rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/bpf                  bpf     bpf        rw,nosuid,nodev,noexec,relatime,mode=700
│ ├─/sys/kernel/tracing          tracefs tracefs    rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/config           configfs
│ │                                      configfs   rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/debug            debugfs debugfs    rw,nosuid,nodev,noexec,relatime
│ └─/sys/fs/fuse/connections     fusectl fusectl    rw,nosuid,nodev,noexec,relatime
└─/tmp                           tmpfs   tmpfs      rw,nosuid,relatime,inode64
```

**Способ 2. Чтение `/proc/mounts`** (таблица монтирования, которую ядро отдаёт напрямую):

```bash
[student@student-13 ~]$ cat /proc/mounts
udevfs /dev devtmpfs rw,relatime,size=5120k,nr_inodes=184459,mode=755,inode64 0 0
runfs /run tmpfs rw,relatime,mode=755,inode64 0 0
/dev/sda1 / ext4 rw,relatime 0 0
proc /proc proc rw,nosuid,noexec,relatime,gid=19 0 0
sysfs /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
securityfs /sys/kernel/security securityfs rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /dev/shm tmpfs rw,nosuid,nodev,inode64 0 0
devpts /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666 0 0
cgroup2 /sys/fs/cgroup cgroup2 rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot 0 0
pstore /sys/fs/pstore pstore rw,nosuid,nodev,noexec,relatime 0 0
bpf /sys/fs/bpf bpf rw,nosuid,nodev,noexec,relatime,mode=700 0 0
systemd-1 /proc/sys/fs/binfmt_misc autofs rw,relatime,fd=36,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=2620 0 0
tmpfs /run/credentials/systemd-journald.service tmpfs ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=1024,mode=700,inode64,noswap 0 0
tracefs /sys/kernel/tracing tracefs rw,nosuid,nodev,noexec,relatime 0 0
mqueue /dev/mqueue mqueue rw,nosuid,nodev,noexec,relatime 0 0
hugetlbfs /dev/hugepages hugetlbfs rw,nosuid,nodev,relatime,pagesize=2M 0 0
configfs /sys/kernel/config configfs rw,nosuid,nodev,noexec,relatime 0 0
debugfs /sys/kernel/debug debugfs rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /run/credentials/systemd-resolved.service tmpfs ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=1024,mode=700,inode64,noswap 0 0
tmpfs /tmp tmpfs rw,nosuid,relatime,inode64 0 0
fusectl /sys/fs/fuse/connections fusectl rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /run/credentials/systemd-networkd.service tmpfs ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=1024,mode=700,inode64,noswap 0 0
tmpfs /run/credentials/getty@tty1.service tmpfs ro,nosuid,nodev,noexec,relatime,nosymfollow,size=1024k,nr_inodes=1024,mode=700,inode64,noswap 0 0
tmpfs /run/user/1000 tmpfs rw,nosuid,nodev,relatime,size=150256k,nr_inodes=37564,mode=700,uid=1000,gid=1001,inode64 0 0
```

Оба способа показывают один и тот же набор точек монтирования: `findmnt` — в виде дерева, `/proc/mounts` — плоским списком. Тестовый каталог `/mnt/test` в списках отсутствует, так как к этому моменту он уже отмонтирован.

## 9. Определение типа файловой системы для точки монтирования

Для корневого каталога `/`:

```bash
[student@student-13 ~]$ findmnt -n -o SOURCE,FSTYPE /
/dev/sda1 ext4
```

Корень находится на устройстве `/dev/sda1`, тип файловой системы — `ext4`. Для виртуальных файловых систем:

```bash
[student@student-13 ~]$ findmnt -n -o SOURCE,FSTYPE /proc
proc proc
[student@student-13 ~]$ findmnt -n -o SOURCE,FSTYPE /sys
sysfs sysfs
```

## 10. Различия между дисковой и виртуальными файловыми системами

| Признак | Дисковая (`ext4`) | Виртуальные (`/proc`, `/sys`) |
|---|---|---|
| Где хранятся данные | на блочном устройстве (`/dev/sda1`) | в памяти ядра, на диске нет |
| Источник содержимого | записывают пользователи и программы | генерирует ядро в момент чтения |
| Блочное устройство | нужно (`/dev/sda1`) | не нужно: источник `proc`/`sysfs`, в `/proc/filesystems` помечены `nodev` |
| Сохранение после перезагрузки | да | нет, содержимое создаётся заново при загрузке |
| Размер файлов | реальный | обычно показывается `0`, но при чтении выдают данные |
| Назначение | долговременное хранение файлов | интерфейс между пользователем и ядром: состояние системы, процессов, устройств |
| Создание | `mkfs` | не создаются, ядро монтирует их само |

Пример: файл `/proc/cpuinfo` имеет нулевой размер, но при чтении выдаёт данные о процессоре.

```bash
[student@student-13 ~]$ ls -l /proc/cpuinfo
-r--r--r-- 1 root root 0 Nov 15 23:01 /proc/cpuinfo
[student@student-13 ~]$ wc -c /proc/cpuinfo
1157 /proc/cpuinfo
```

`/proc` ориентирован на процессы и общую информацию о системе, `/sys` — на устройства, драйверы и их настройки.

## 11. Возврат системы в исходное состояние

```bash
[student@student-13 ~]$ findmnt /mnt/test; echo "код возврата findmnt: $?"
код возврата findmnt: 1
[student@student-13 ~]$ sudo rmdir /mnt/test
[student@student-13 ~]$ command rm /tmp/disk.img
[student@student-13 ~]$ ls -ld /mnt/test; ls -l /tmp/disk.img
ls: cannot access '/mnt/test': No such file or directory
ls: cannot access '/tmp/disk.img': No such file or directory
```

Файловая система отмонтирована, каталог `/mnt/test` и образ `/tmp/disk.img` удалены. Диски `sdb` и `sdc` в работе не использовались.
