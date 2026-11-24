# Григорьев Сергей 241 группа
# Продолжаем

1) Выведите содержимое `/etc/fstab`. Что хранится в этом файле?
2) Добавьте в виртуальную машину ещё один виртуальный диск.
3) Определите, как система видит новый диск, и выведите информацию о блочных устройствах.
4) Создайте на новом диске таблицу разделов и один раздел.
5) Создайте на разделе файловую систему ext4.
6) Создайте отдельный каталог в `/mnt` и примонтируйте туда файловую систему.
7) Создайте на примонтированном диске несколько файлов.
8) Отмонтируйте диск и проверьте, видны ли созданные файлы через каталог монтирования. Объясните результат.
9) Настройте автоматическое монтирование файловой системы при загрузке через `/etc/fstab`.
10) Для записи в `/etc/fstab` используйте UUID файловой системы, а не имя вида `/dev/sdX`.
11) Проверьте корректность `/etc/fstab` до перезагрузки.
12) Перезагрузите систему и убедитесь, что файловая система примонтирована автоматически.

## Критерий приёмки

В отчёте должны быть:
- вывод `/etc/fstab` до изменений;
- вывод списка блочных устройств после добавления диска;
- информация о созданной таблице разделов и разделе;
- команда создания ext4;
- команда ручного монтирования и вывод, подтверждающий успешное монтирование;
- список тестовых файлов на примонтированном диске;
- результат проверки после размонтирования;
- UUID созданной файловой системы;
- добавленная строка из `/etc/fstab`;
- команда проверки корректности `/etc/fstab` до перезагрузки;
- вывод после перезагрузки, подтверждающий автоматическое монтирование.

# Отчёт по практической работе: Продолжаем

## 1. Содержимое `/etc/fstab` (до изменений)

Использованная команда:

```bash
cat /etc/fstab
```

Полученный результат:

```text
proc            /proc                   proc    nosuid,noexec,gid=proc                          0 0
devpts          /dev/pts                devpts  nosuid,noexec,gid=tty,mode=620,ptmxmode=0666    0 0
tmpfs           /tmp                    tmpfs   nosuid                                          0 0
UUID=93c203a1-7a96-4cd1-a5f8-ed900f7b23ab       /       ext4    defaults        1 1
```

**Что хранится в этом файле:** `/etc/fstab` — таблица статической информации о файловых системах. В ней описано, какие файловые системы монтировать при загрузке, в какой каталог, с каким типом ФС и какими параметрами. Здесь описаны виртуальные ФС `proc`, `devpts`, `tmpfs` (для `/tmp`) и корневая ФС, подключённая по UUID.

Формат строки (6 полей):

| Поле | Значение |
|------|----------|
| файловая система | устройство (UUID, метка, `/dev/...`) или тип виртуальной ФС |
| точка монтирования | каталог, куда монтируется ФС |
| тип | тип файловой системы (`ext4`, `tmpfs` и т. д.) |
| параметры | параметры монтирования (`defaults`, `nosuid`, ...) |
| dump | нужна ли резервная копия утилитой `dump` (0 — нет) |
| pass | порядок проверки `fsck` при загрузке (0 — не проверять, 1 — корень, 2 — остальные) |

## 2. Виртуальный диск

Добавить виртуальный диск на учебном сервере самостоятельно нельзя (нет доступа к гипервизору). Дополнительные диски `sdb` и `sdc` (по 1 ГБ) уже предоставлены средой; на них не было ни разделов, ни файловых систем (см. `lsblk -f` ниже). Для работы использован диск `/dev/sdb`, диск `/dev/sdc` не использовался.

Использованная команда:

```bash
lsblk -f
```

Полученный результат:

```text
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda
└─sda1 ext4   1.0         93c203a1-7a96-4cd1-a5f8-ed900f7b23ab   13.2G     6% /
sdb
sdc
sr0
```

## 3. Как система видит новый диск

Использованные команды:

```bash
lsblk
sudo /sbin/fdisk -l /dev/sdb
sudo dmesg | grep -i sdb
```

Полученный результат:

```text
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   15G  0 disk
└─sda1   8:1    0   15G  0 part /
sdb      8:16   0    1G  0 disk
sdc      8:32   0    1G  0 disk
sr0     11:0    1 1024M  0 rom
```

```text
Disk /dev/sdb: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

```text
[    2.122173] sd 9:0:0:1: [sdb] 2097152 512-byte logical blocks: (1.07 GB/1.00 GiB)
[    2.122216] sd 9:0:0:1: [sdb] Write Protect is off
[    2.122219] sd 9:0:0:1: [sdb] Mode Sense: 63 00 10 08
[    2.122355] sd 9:0:0:1: [sdb] Write cache: enabled, read cache: enabled, supports DPO and FUA
[    2.126630] sd 9:0:0:1: [sdb] Attached SCSI disk
```

Новый диск определился как `/dev/sdb` размером 1 GiB (модель QEMU HARDDISK). Таблицы разделов на нём нет: в `fdisk -l` нет строки `Disklabel type`, в `lsblk` нет разделов. Сообщения `dmesg` подтверждают, что ядро определило диск при загрузке.

## 4. Таблица разделов и один раздел

Использованная команда (таблица GPT, один раздел на весь диск):

```bash
sudo /sbin/fdisk /dev/sdb <<'EOF'
g
n
1


w
EOF
```

Проверка:

```bash
sudo /sbin/fdisk -l /dev/sdb
lsblk /dev/sdb
```

Полученный результат:

```text
Disk /dev/sdb: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 81BA283A-2F86-4B19-A69F-CCF00F1811AF

Device     Start     End Sectors  Size Type
/dev/sdb1   2048 2095103 2093056 1022M Linux filesystem
```

```text
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdb      8:16   0    1G  0 disk
└─sdb1   8:17   0 1022M  0 part
```

На диске создана таблица разделов GPT и один раздел `/dev/sdb1` на весь диск (1022M, тип Linux filesystem).

## 5. Файловая система ext4

Использованные команды:

```bash
sudo /sbin/mkfs.ext4 /dev/sdb1
sudo /sbin/blkid /dev/sdb1
lsblk -f /dev/sdb
```

Полученный результат:

```text
mke2fs 1.47.1 (20-May-2024)
Discarding device blocks: done
Creating filesystem with 261632 4k blocks and 65408 inodes
Filesystem UUID: c3fd1860-3f06-4c32-9790-dff4116f4a2a
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```

```text
/dev/sdb1: UUID="c3fd1860-3f06-4c32-9790-dff4116f4a2a" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="b34c262d-14f2-4620-9a12-f0ce307305c9"
```

```text
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sdb
└─sdb1 ext4   1.0         c3fd1860-3f06-4c32-9790-dff4116f4a2a
```

На разделе `/dev/sdb1` создана файловая система ext4. **UUID файловой системы: `c3fd1860-3f06-4c32-9790-dff4116f4a2a`.**

## 6. Каталог в `/mnt` и ручное монтирование

Использованные команды:

```bash
sudo mkdir /mnt/newdisk
sudo mount /dev/sdb1 /mnt/newdisk
findmnt /mnt/newdisk
df -h /mnt/newdisk
```

Полученный результат:

```text
TARGET       SOURCE    FSTYPE OPTIONS
/mnt/newdisk /dev/sdb1 ext4   rw,relatime
```

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       988M  276K  921M   1% /mnt/newdisk
```

Раздел `/dev/sdb1` примонтирован в `/mnt/newdisk`.

## 7. Тестовые файлы на примонтированном диске

Использованные команды:

```bash
sudo touch /mnt/newdisk/file1.txt /mnt/newdisk/file2.txt
echo "test data" | sudo tee /mnt/newdisk/file3.txt
ls -l /mnt/newdisk
```

Полученный результат:

```text
test data
total 20
-rw-r--r-- 1 root root     0 Nov 24 11:14 file1.txt
-rw-r--r-- 1 root root     0 Nov 24 11:14 file2.txt
-rw-r--r-- 1 root root    10 Nov 24 11:14 file3.txt
drwx------ 2 root root 16384 Nov 24 11:12 lost+found
```

Создано три файла. Каталог `lost+found` создаётся автоматически при создании ext4.

## 8. Размонтирование и проверка

Использованные команды:

```bash
sudo umount /mnt/newdisk
ls -l /mnt/newdisk
findmnt /mnt/newdisk; echo "код возврата findmnt: $?"
```

Полученный результат:

```text
total 0
код возврата findmnt: 1
```

**Объяснение результата:** после размонтирования каталог `/mnt/newdisk` пуст, `findmnt` ничего не выводит и возвращает код 1 (точка не примонтирована). Файлы не удалены: они хранятся в файловой системе на `/dev/sdb1`. Сам каталог `/mnt/newdisk` — лишь пустая точка монтирования на корневой ФС (`/dev/sda1`). Пока диск смонтирован, каталог показывает содержимое `sdb1`, после размонтирования — собственное (пустое) содержимое. После повторного монтирования файлы снова становятся видны.

## 9–10. Автомонтирование через `/etc/fstab` по UUID

В записи использован UUID файловой системы, а не имя `/dev/sdb1`, так как имя устройства может измениться при смене порядка дисков, а UUID остаётся постоянным. Перед правкой сделана резервная копия.

Использованные команды:

```bash
sudo cp /etc/fstab /etc/fstab.bak
echo 'UUID=c3fd1860-3f06-4c32-9790-dff4116f4a2a  /mnt/newdisk  ext4  defaults  0  2' | sudo tee -a /etc/fstab
cat /etc/fstab
```

Полученный результат:

```text
UUID=c3fd1860-3f06-4c32-9790-dff4116f4a2a  /mnt/newdisk  ext4  defaults  0  2
```

```text
proc            /proc                   proc    nosuid,noexec,gid=proc            0 0
devpts          /dev/pts                devpts  nosuid,noexec,gid=tty,mode=620,ptmxmode=0666       0 0
tmpfs           /tmp                    tmpfs   nosuid                            0 0
UUID=93c203a1-7a96-4cd1-a5f8-ed900f7b23ab       /       ext4    defaults        1 1
UUID=c3fd1860-3f06-4c32-9790-dff4116f4a2a  /mnt/newdisk  ext4  defaults  0  2
```

**Добавленная строка:**

```fstab
UUID=c3fd1860-3f06-4c32-9790-dff4116f4a2a  /mnt/newdisk  ext4  defaults  0  2
```

## 11. Проверка корректности `/etc/fstab` до перезагрузки

Использованные команды:

```bash
sudo findmnt --verify
sudo mount -a
findmnt /mnt/newdisk
ls /mnt/newdisk
```

Полученный результат:

```text
Success, no errors or warnings detected
```

```text
TARGET       SOURCE    FSTYPE OPTIONS
/mnt/newdisk /dev/sdb1 ext4   rw,relatime
```

```text
file1.txt  file2.txt  file3.txt  lost+found
```

`findmnt --verify` не нашла ошибок. `mount -a` примонтировала файловую систему по записи из `/etc/fstab`, файлы на месте.

## 12. Перезагрузка и проверка автомонтирования

Использованная команда:

```bash
sudo reboot
```

Полученный результат:

```text
Broadcast message from root@student-13 on pts/1 (Tue 2026-11-24 11:16:27 UTC):

The system will reboot now!
```

После повторного подключения:

```bash
uptime -p
findmnt /mnt/newdisk
lsblk -f /dev/sdb
ls /mnt/newdisk
```

```text
up 21 minutes
```

```text
TARGET       SOURCE    FSTYPE OPTIONS
/mnt/newdisk /dev/sdb1 ext4   rw,relatime
```

```text
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sdb
└─sdb1 ext4   1.0         c3fd1860-3f06-4c32-9790-dff4116f4a2a  920.1M     0% /mnt/newdisk
```

```text
file1.txt  file2.txt  file3.txt  lost+found
```

После перезагрузки (`uptime -p` показывает время работы в пределах минут после `reboot`) файловая система на `/dev/sdb1` примонтирована автоматически в `/mnt/newdisk`, тестовые файлы сохранились.

## Критерий приёмки

- вывод `/etc/fstab` до изменений — пункт 1;
- вывод списка блочных устройств после добавления диска — пункт 3;
- информация о созданной таблице разделов и разделе — пункт 4;
- команда создания ext4 — пункт 5 (`mkfs.ext4 /dev/sdb1`);
- команда ручного монтирования и вывод, подтверждающий успех, — пункт 6;
- список тестовых файлов на примонтированном диске — пункт 7;
- результат проверки после размонтирования — пункт 8;
- UUID созданной файловой системы — пункт 5 (`c3fd1860-3f06-4c32-9790-dff4116f4a2a`);
- добавленная строка из `/etc/fstab` — пункты 9–10;
- команда проверки `/etc/fstab` до перезагрузки — пункт 11 (`findmnt --verify`, `mount -a`);
- вывод после перезагрузки, подтверждающий автомонтирование, — пункт 12.
