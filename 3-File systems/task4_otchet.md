# Григорьев Сергей 241 группа
# RAID-массивы

1) Что такое RAID и какие основные уровни RAID вы знаете?
2) Добавьте в виртуальную машину два дополнительных диска.
3) На двух дисках создайте RAID0-массив средствами Linux software RAID.
4) Проверьте состояние массива.
5) Создайте файловую систему ext4 уже на RAID-устройстве и примонтируйте её.
6) Запишите на массив тестовые файлы.
7) Удалите RAID0 и создайте RAID1.
8) Создайте файловую систему на RAID1 и снова проверьте чтение/запись.
9) В чём практическая разница между RAID0 и RAID1?
10) Имитируйте отказ одного диска RAID1:
    10.1) пометьте диск как неисправный;
    10.2) удалите его из массива;
    10.3) убедитесь, что данные остаются доступны.
11) Верните диск в массив или добавьте новый и наблюдайте процесс восстановления.
12) Какие файловые системы могут самостоятельно обеспечивать функции, похожие на RAID?
13) Можно ли создать RAID во время установки Linux?

## Критерий приёмки

В отчёте должны быть:
- вывод списка дисков, используемых для RAID;
- команды создания RAID0;
- вывод состояния RAID0;
- команда создания ext4 на RAID-устройстве;
- подтверждение монтирования и записи тестовых файлов;
- команды удаления RAID0 и создания RAID1;
- вывод состояния RAID1;
- команды имитации отказа одного диска;
- подтверждение доступности данных после отказа;
- вывод процесса восстановления массива после возврата или замены диска;
- краткое сравнение RAID0 и RAID1 по производительности, ёмкости и отказоустойчивости.

# Отчёт по практической работе: RAID-массивы


## 1. Что такое RAID и основные уровни
 
**RAID** (Redundant Array of Independent Disks — избыточный массив независимых дисков) — технология, объединяющая несколько физических дисков в один логический том. Цели: повысить скорость работы, увеличить доступную ёмкость и/или обеспечить отказоустойчивость (сохранность данных при выходе из строя диска).
 
RAID бывает **аппаратный** (отдельный контроллер), **программный** (средствами ОС; в Linux это подсистема `md` и утилита `mdadm`) и «фейковый» (поддержка в BIOS/чипсете + драйвер). В работе используется программный RAID Linux.
 
Основные уровни:
 
| Уровень | Принцип | Мин. дисков | Полезная ёмкость | Отказоустойчивость |
|---------|---------|-------------|------------------|--------------------|
| RAID0 | чередование (striping) данных по дискам | 2 | сумма всех дисков | нет: отказ любого диска = потеря всех данных |
| RAID1 | зеркалирование (mirroring), на всех дисках одинаковые данные | 2 | размер одного (наименьшего) диска | массив работает, пока жив хотя бы один диск |
| RAID5 | чередование + распределённая чётность (parity) | 3 | (N − 1) дисков | 1 диск |
| RAID6 | чередование + двойная чётность | 4 | (N − 2) дисков | 2 диска |
| RAID10 (1+0) | чередование поверх зеркал | 4 | N / 2 | минимум 1 диск (до одного в каждом зеркале) |
 
Также встречаются комбинации (RAID50, RAID60) и режим linear/JBOD (последовательное объединение дисков без избыточности).
 
> RAID защищает от отказа диска, но **не заменяет резервное копирование**: случайное удаление файла, вирус или ошибка файловой системы одинаково отразятся на всех дисках массива.
 
## 2. Диски для RAID
 
### Подготовка: установка `mdadm`
 
Исходное состояние диска и RAID в системе:
 
```bash
[student@student-13 ~]$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   15G  0 disk
└─sda1   8:1    0   15G  0 part /
sdb      8:16   0    1G  0 disk
└─sdb1   8:17   0 1022M  0 part /mnt/newdisk
sdc      8:32   0    1G  0 disk
sr0     11:0    1 1024M  0 rom
[student@student-13 ~]$ cat /proc/mdstat
Personalities :
unused devices: <none>
[student@student-13 ~]$ ls -l /sbin/mdadm /usr/sbin/mdadm
ls: cannot access '/sbin/mdadm': No such file or directory
ls: cannot access '/usr/sbin/mdadm': No such file or directory
```
 
Массивов нет, `mdadm` не установлен. Диск `sdb` занят предыдущей практической работой (смонтирован в `/mnt/newdisk` и прописан в `/etc/fstab`), а свободен только `sdc`, то есть двух свободных дисков для RAID на сервере нет. Добавить новые виртуальные диски в настройках гипервизора на этом сервере возможности нет, поэтому в качестве «дисков» для RAID используются два **loop-устройства**, созданные из файлов-образов. Для `mdadm` и ядра это обычные блочные устройства, а `sdb` и `sdc` остаются нетронутыми.
 
Проверка поддержки RAID в ядре и наличия пакета в репозитории:
 
```bash
[student@student-13 ~]$ /sbin/modinfo raid0 raid1 | grep -E 'filename|name:'
modinfo: ERROR: could not get modinfo from 'raid0': Permission denied
modinfo: ERROR: could not get modinfo from 'raid1': Permission denied
filename:       /lib/modules/6.12.51-6.12-alt1/kernel/drivers/md/raid0.ko.zst
filename:       /lib/modules/6.12.51-6.12-alt1/kernel/drivers/md/raid1.ko.zst
[student@student-13 ~]$ sudo apt-get update
[...]
Fetched 60.0MB in 11m51s (84.3kB/s)
Reading Package Lists... Done
Building Dependency Tree... Done
[student@student-13 ~]$ apt-cache policy mdadm
mdadm:
  Installed: (none)
  Candidate: 4.6-alt1:sisyphus+417031.100.1.1@1777751135
  Version Table:
     4.6-alt1:sisyphus+417031.100.1.1@1777751135 0
        500 http://ftp.altlinux.org Sisyphus/x86_64/classic pkglist [...]
```
 
Модули ядра `raid0` и `raid1` присутствуют (сообщения `Permission denied` появились из-за запуска `modinfo` без `sudo` и на результат не влияют). Перед установкой выполнена симуляция (`-s`), показывающая, что будет изменено:
 
```bash
[student@student-13 ~]$ sudo apt-get install -s mdadm
[...]
The following packages will be upgraded
  glibc-core           glibc-locales  glibc-preinstall  glibc-utils
  glibc-gconv-modules  glibc-nss      glibc-pthread
The following NEW packages will be installed:
  mdadm  mdadm-tool  schedutils
7 upgraded, 3 newly installed, 0 removed and 178 not upgraded.
```
 
Вместе с `mdadm` обновляются 7 пакетов `glibc`, так как `mdadm` из репозитория собран под более новую версию библиотеки. Из-за периодических обрывов SSH-сессии установка запущена в фоне, чтобы обрыв не прервал её:
 
```bash
[student@student-13 ~]$ nohup sudo apt-get install -y mdadm > ~/apt-mdadm.log 2>&1 &
[1] 950
[student@student-13 ~]$ [1]+  Done                    nohup sudo apt-get install -y mdadm > ~/apt-mdadm.log 2>&1
[student@student-13 ~]$ tail -n 12 ~/apt-mdadm.log
[...]
15: glibc-pthread-6:2.40.0.142.2eb180-alt1 ################################# [ 88%]
16: glibc-core-6:2.40.0.142.2eb180-alt1 ################################# [ 94%]
17: glibc-preinstall-6:2.40.0.142.2eb180-alt1 ################################# [100%]
Done.
[student@student-13 ~]$ rpm -q mdadm mdadm-tool glibc-core
mdadm-4.6-alt1.x86_64
mdadm-tool-4.6-alt1.x86_64
glibc-core-2.43.0.35.fe0ccc-alt1.x86_64
[student@student-13 ~]$ sudo /sbin/mdadm --version
mdadm - v4.6 - 2026-03-16
```
 
Установка завершилась без ошибок, `mdadm` версии 4.6 работает.
 
### Создание дисков для RAID
 
Два файла-образа по 256 МБ в `/tmp` подключаются как loop-устройства:
 
```bash
[student@student-13 ~]$ df -h /tmp
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           734M     0  734M   0% /tmp
[student@student-13 ~]$ truncate -s 256M /tmp/raid_disk1.img /tmp/raid_disk2.img
[student@student-13 ~]$ ls -l /tmp/raid_disk*.img
-rw-r--r-- 1 student student 268435456 Nov 24 12:59 /tmp/raid_disk1.img
-rw-r--r-- 1 student student 268435456 Nov 24 12:59 /tmp/raid_disk2.img
[student@student-13 ~]$ sudo /sbin/losetup -f --show /tmp/raid_disk1.img
/dev/loop0
[student@student-13 ~]$ sudo /sbin/losetup -f --show /tmp/raid_disk2.img
/dev/loop1
[student@student-13 ~]$ lsblk | grep -E 'NAME|loop'
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0  256M  0 loop
loop1    7:1    0  256M  0 loop
```
 
Для RAID используются `/dev/loop0` и `/dev/loop1` (по 256 МБ), разделов и файловых систем на них нет. Каталог `/tmp` расположен в оперативной памяти (`tmpfs`), поэтому после перезагрузки образы исчезнут, для лабораторной это допустимо.
 
## 3. Создание RAID0
 
```bash
[student@student-13 ~]$ sudo /sbin/mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/loop0 /dev/loop1
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```
 
Параметры: `--create /dev/md0` — создать устройство массива, `--level=0` — уровень RAID0, `--raid-devices=2` — число дисков, далее перечислены сами диски.
 
## 4. Состояние RAID0
 
```bash
[student@student-13 ~]$ cat /proc/mdstat
Personalities : [raid0]
md0 : active raid0 loop1[1] loop0[0]
      520192 blocks super 1.2 512k chunks
 
unused devices: <none>
[student@student-13 ~]$ sudo /sbin/mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Tue Nov 24 13:00:36 2026
        Raid Level : raid0
        Array Size : 520192 (508.00 MiB 532.68 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent
 
       Update Time : Tue Nov 24 13:00:36 2026
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0
 
            Layout : original
        Chunk Size : 512K
 
Consistency Policy : none
 
              Name : student-13:0  (local to host student-13)
              UUID : 260ca52b:cdab5ffa:b12e992e:095bc329
            Events : 0
 
    Number   Major   Minor   RaidDevice State
       0       7        0        0      active sync   /dev/loop0
       1       7        1        1      active sync   /dev/loop1
```
 
Массив `md0` активен, уровень `raid0`, состояние `clean`, оба диска в статусе `active sync`. Размер 508 МиБ — сумма двух дисков по 256 МБ, ёмкость не теряется. Данные чередуются блоками по 512 КБ (`Chunk Size`).
 
## 5. Файловая система ext4 на RAID-устройстве и монтирование
 
Файловая система создаётся на устройстве массива `/dev/md0`, а не на отдельных loop-устройствах:
 
```bash
[student@student-13 ~]$ sudo /sbin/mkfs.ext4 /dev/md0
mke2fs 1.47.1 (20-May-2024)
Discarding device blocks: done
Creating filesystem with 520192 1k blocks and 130048 inodes
Filesystem UUID: a08f78d2-b8ea-490f-85a8-6b4960b379cf
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729, 204801, 221185, 401409
 
Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
 
[student@student-13 ~]$ sudo mkdir /mnt/raid
[student@student-13 ~]$ sudo mount /dev/md0 /mnt/raid
[student@student-13 ~]$ findmnt /mnt/raid
TARGET    SOURCE   FSTYPE OPTIONS
/mnt/raid /dev/md0 ext4   rw,relatime,stripe=1024
[student@student-13 ~]$ df -h /mnt/raid
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0        466M  141K  437M   1% /mnt/raid
```
 
Файловая система смонтирована. Параметр `stripe=1024` ext4 подобрала автоматически под геометрию массива: чередование 512 КБ на каждом из двух дисков при размере блока 1 КБ даёт 1024 блока.
 
## 6. Тестовые файлы на RAID0
 
```bash
[student@student-13 ~]$ echo "RAID0 test file 1" | sudo tee /mnt/raid/raid0_file1.txt
RAID0 test file 1
[student@student-13 ~]$ echo "RAID0 test file 2" | sudo tee /mnt/raid/raid0_file2.txt
RAID0 test file 2
[student@student-13 ~]$ sudo dd if=/dev/urandom of=/mnt/raid/raid0_data.bin bs=1M count=20
20+0 records in
20+0 records out
20971520 bytes (21 MB, 20 MiB) copied, 0.1363 s, 154 MB/s
[student@student-13 ~]$ ls -l /mnt/raid
total 20494
drwx------ 2 root root    12288 Nov 24 13:01 lost+found
-rw-r--r-- 1 root root 20971520 Nov 24 13:02 raid0_data.bin
-rw-r--r-- 1 root root       18 Nov 24 13:02 raid0_file1.txt
-rw-r--r-- 1 root root       18 Nov 24 13:02 raid0_file2.txt
[student@student-13 ~]$ cat /mnt/raid/raid0_file1.txt
RAID0 test file 1
[student@student-13 ~]$ df -h /mnt/raid
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0        466M   21M  417M   5% /mnt/raid
```
 
Запись и чтение на RAID0 работают. Каталог `lost+found` создаётся автоматически при создании ext4.
 
## 7. Удаление RAID0 и создание RAID1
 
Для удаления RAID0 файловая система размонтируется, массив останавливается, а с дисков стираются служебные метаданные `md` (суперблоки):
 
```bash
[student@student-13 ~]$ sudo umount /mnt/raid
[student@student-13 ~]$ sudo /sbin/mdadm --stop /dev/md0
mdadm: stopped /dev/md0
[student@student-13 ~]$ sudo /sbin/mdadm --zero-superblock /dev/loop0 /dev/loop1
[student@student-13 ~]$ cat /proc/mdstat
Personalities : [raid0]
unused devices: <none>
[student@student-13 ~]$ lsblk | grep -E 'NAME|loop|md'
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0  256M  0 loop
loop1    7:1    0  256M  0 loop
```
 
Массивов в системе нет, данные RAID0 уничтожены. Создание RAID1 на тех же двух устройствах:
 
```bash
[student@student-13 ~]$ sudo /sbin/mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/loop0 /dev/loop1
To optimize recovery speed, it is recommended to enable write-intent bitmap, do you want to enable it now? [y/N]? y
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```
 
На запросы `mdadm` дан ответ `y`: включён write-intent bitmap (ускоряет восстановление, так как после сбоя пересинхронизируются только изменённые участки) и подтверждено создание массива (предупреждение касается только использования массива как загрузочного устройства, что здесь не нужно).
 
### Состояние RAID1
 
```bash
[student@student-13 ~]$ cat /proc/mdstat
Personalities : [raid0] [raid1]
md0 : active raid1 loop1[1] loop0[0]
      261120 blocks super 1.2 [2/2] [UU]
      bitmap: 1/1 pages [4KB], 65536KB chunk
 
unused devices: <none>
[student@student-13 ~]$ sleep 10
[student@student-13 ~]$ cat /proc/mdstat
Personalities : [raid0] [raid1]
md0 : active raid1 loop1[1] loop0[0]
      261120 blocks super 1.2 [2/2] [UU]
      bitmap: 0/1 pages [0KB], 65536KB chunk
 
unused devices: <none>
[student@student-13 ~]$ sudo /sbin/mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Tue Nov 24 13:04:25 2026
        Raid Level : raid1
        Array Size : 261120 (255.00 MiB 267.39 MB)
     Used Dev Size : 261120 (255.00 MiB 267.39 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent
 
     Intent Bitmap : Internal
 
       Update Time : Tue Nov 24 13:04:27 2026
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0
 
Consistency Policy : bitmap
 
              Name : student-13:0  (local to host student-13)
              UUID : 38c04c11:f4d46108:f8dfdb0f:e9ef0f54
            Events : 17
 
    Number   Major   Minor   RaidDevice State
       0       7        0        0      active sync   /dev/loop0
       1       7        1        1      active sync   /dev/loop1
```
 
Массив `raid1` в состоянии `clean`, `[UU]` — оба диска работают. Полезный размер 255 МиБ, то есть равен одному диску (у RAID0 из тех же дисков было 508 МиБ). Полосы начальной синхронизации (`resync`) в выводе нет: массив из пустых устройств по 256 МБ синхронизировался мгновенно.
 
## 8. Файловая система на RAID1 и проверка чтения/записи
 
```bash
[student@student-13 ~]$ sudo /sbin/mkfs.ext4 /dev/md0
mke2fs 1.47.1 (20-May-2024)
Discarding device blocks: done
Creating filesystem with 261120 1k blocks and 65280 inodes
Filesystem UUID: 8e2af17f-3a6c-4d9c-a0a0-bc63c6a2ffff
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729, 204801, 221185
 
Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
 
[student@student-13 ~]$ sudo mount /dev/md0 /mnt/raid
[student@student-13 ~]$ findmnt /mnt/raid
TARGET    SOURCE   FSTYPE OPTIONS
/mnt/raid /dev/md0 ext4   rw,relatime
```
 
Запись и чтение тестовых файлов. Контрольная сумма большого файла сохраняется в домашний каталог, то есть вне массива, чтобы потом проверить целостность данных после «отказа» диска:
 
```bash
[student@student-13 ~]$ echo "RAID1 test file 1" | sudo tee /mnt/raid/raid1_file1.txt
RAID1 test file 1
[student@student-13 ~]$ echo "RAID1 test file 2" | sudo tee /mnt/raid/raid1_file2.txt
RAID1 test file 2
[student@student-13 ~]$ sudo dd if=/dev/urandom of=/mnt/raid/raid1_data.bin bs=1M count=20
20+0 records in
20+0 records out
20971520 bytes (21 MB, 20 MiB) copied, 0.100251 s, 209 MB/s
[student@student-13 ~]$ sha256sum /mnt/raid/raid1_data.bin | tee ~/raid1_data.sha256
f2b72839d866a71bda6d805c0822ee73a8d0b1cd63b22ef1412b79aff74c5078  /mnt/raid/raid1_data.bin
[student@student-13 ~]$ ls -l /mnt/raid
total 20494
drwx------ 2 root root    12288 Nov 24 13:05 lost+found
-rw-r--r-- 1 root root 20971520 Nov 24 13:06 raid1_data.bin
-rw-r--r-- 1 root root       18 Nov 24 13:05 raid1_file1.txt
-rw-r--r-- 1 root root       18 Nov 24 13:06 raid1_file2.txt
[student@student-13 ~]$ cat /mnt/raid/raid1_file2.txt
RAID1 test file 2
```
 
Чтение и запись на RAID1 работают.
 
## 9. Практическая разница между RAID0 и RAID1
 
| Критерий | RAID0 (striping) | RAID1 (mirroring) |
|----------|------------------|-------------------|
| Производительность | Максимальная: данные чередуются по дискам, чтение и запись идут параллельно на всех дисках. | Чтение может ускоряться (запросы распределяются между зеркалами), запись примерно как на одном диске, так как данные пишутся на все диски сразу. |
| Ёмкость | Сумма дисков. В работе: 2 × 256 МБ → 508 МиБ. | Размер одного диска. В работе: 2 × 256 МБ → 255 МиБ (эффективность 50 % для двух дисков). |
| Отказоустойчивость | Отсутствует: отказ любого диска приводит к потере всех данных массива, риск выше, чем у одиночного диска. | Высокая: данные сохраняются, пока жив хотя бы один диск (проверено в п. 10). |
 
Практически: RAID0 подходит для временных данных, кэшей и задач, где важна скорость, а потеря данных не критична. RAID1 — для данных, которые нельзя потерять.
 
Замечание: скорости `dd` в п. 6 и 8 (154 и 209 МБ/с) для сравнения производительности не годятся. Диски в этой работе — файлы в оперативной памяти, а скорость записи ограничена генератором `/dev/urandom` и разбросом от запуска к запуску. Поэтому сравнение по производительности в таблице теоретическое, а по ёмкости и отказоустойчивости подтверждено выводами выше.
 
## 10. Имитация отказа одного диска RAID1
 
Перед отказом массив в состоянии `clean`, `[UU]` (см. п. 8).
 
### 10.1. Пометка диска как неисправного
 
```bash
[student@student-13 ~]$ sudo /sbin/mdadm /dev/md0 --fail /dev/loop1
[student@student-13 ~]$ cat /proc/mdstat
Personalities : [raid0] [raid1]
md0 : active raid1 loop1[1](F) loop0[0]
      261120 blocks super 1.2 [2/1] [U_]
      bitmap: 0/1 pages [0KB], 65536KB chunk
 
unused devices: <none>
[student@student-13 ~]$ sudo /sbin/mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Tue Nov 24 13:04:25 2026
        Raid Level : raid1
        Array Size : 261120 (255.00 MiB 267.39 MB)
     Used Dev Size : 261120 (255.00 MiB 267.39 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent
 
     Intent Bitmap : Internal
 
       Update Time : Tue Nov 24 13:07:09 2026
             State : clean, degraded
    Active Devices : 1
   Working Devices : 1
    Failed Devices : 1
     Spare Devices : 0
 
Consistency Policy : bitmap
 
              Name : student-13:0  (local to host student-13)
              UUID : 38c04c11:f4d46108:f8dfdb0f:e9ef0f54
            Events : 19
 
    Number   Major   Minor   RaidDevice State
       0       7        0        0      active sync   /dev/loop0
       -       0        0        1      removed
 
       1       7        1        -      faulty   /dev/loop1
```
 
Диск `/dev/loop1` помечен `(F)` (faulty), массив перешёл в состояние `clean, degraded`, `[U_]` — работает один диск из двух.
 
### 10.2. Удаление диска из массива
 
```bash
[student@student-13 ~]$ sudo /sbin/mdadm /dev/md0 --remove /dev/loop1
mdadm: hot removed /dev/loop1 from /dev/md0
[student@student-13 ~]$ cat /proc/mdstat
Personalities : [raid0] [raid1]
md0 : active raid1 loop0[0]
      261120 blocks super 1.2 [2/1] [U_]
      bitmap: 0/1 pages [0KB], 65536KB chunk
 
unused devices: <none>
[student@student-13 ~]$ sudo /sbin/mdadm --detail /dev/md0 | tail -n 5
            Events : 20
 
    Number   Major   Minor   RaidDevice State
       0       7        0        0      active sync   /dev/loop0
       -       0        0        1      removed
```
 
### 10.3. Проверка доступности данных
 
Файловая система остаётся смонтированной, файлы читаются, целостность большого файла подтверждается контрольной суммой, запись новых данных тоже работает:
 
```bash
[student@student-13 ~]$ ls -l /mnt/raid
total 20494
drwx------ 2 root root    12288 Nov 24 13:05 lost+found
-rw-r--r-- 1 root root 20971520 Nov 24 13:06 raid1_data.bin
-rw-r--r-- 1 root root       18 Nov 24 13:05 raid1_file1.txt
-rw-r--r-- 1 root root       18 Nov 24 13:06 raid1_file2.txt
[student@student-13 ~]$ cat /mnt/raid/raid1_file1.txt
RAID1 test file 1
[student@student-13 ~]$ sha256sum -c ~/raid1_data.sha256
/mnt/raid/raid1_data.bin: OK
[student@student-13 ~]$ echo "written in degraded mode" | sudo tee /mnt/raid/raid1_degraded.txt
written in degraded mode
[student@student-13 ~]$ cat /mnt/raid/raid1_degraded.txt
written in degraded mode
```
 
Данные полностью доступны, несмотря на «отказ» одного диска: в этом и состоит смысл RAID1.
 
## 11. Возврат диска в массив и восстановление
 
Диск `/dev/loop1` возвращается в массив, и сразу смотрим состояние:
 
```bash
[student@student-13 ~]$ sudo /sbin/mdadm /dev/md0 --add /dev/loop1; cat /proc/mdstat; sudo /sbin/mdadm --detail /dev/md0 | tail -n 22
mdadm: re-added /dev/loop1
Personalities : [raid0] [raid1]
md0 : active raid1 loop1[1] loop0[0]
      261120 blocks super 1.2 [2/1] [U_]
      [>....................]  recovery =  0.9% (3008/261120) finish=1.4min speed=3008K/sec
      bitmap: 1/1 pages [4KB], 65536KB chunk
 
unused devices: <none>
       Persistence : Superblock is persistent
 
     Intent Bitmap : Internal
 
       Update Time : Tue Nov 24 13:09:28 2026
             State : active, degraded, recovering
    Active Devices : 1
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 1
 
Consistency Policy : bitmap
 
    Rebuild Status : 11% complete
 
              Name : student-13:0  (local to host student-13)
              UUID : 38c04c11:f4d46108:f8dfdb0f:e9ef0f54
            Events : 32
 
    Number   Major   Minor   RaidDevice State
       0       7        0        0      active sync   /dev/loop0
       1       7        1        1      spare rebuilding   /dev/loop1
```
 
`mdadm` сообщил `re-added`: диск вернулся со своими прежними метаданными, поэтому благодаря bitmap копируются только блоки, изменённые за время отсутствия диска. В выводе виден процесс восстановления: `recovery` в `/proc/mdstat`, состояние `active, degraded, recovering`, статус диска `spare rebuilding`.
 
После завершения восстановления (через 5 секунд):
 
```bash
[student@student-13 ~]$ sleep 5
[student@student-13 ~]$ cat /proc/mdstat
Personalities : [raid0] [raid1]
md0 : active raid1 loop1[1] loop0[0]
      261120 blocks super 1.2 [2/2] [UU]
      bitmap: 0/1 pages [0KB], 65536KB chunk
 
unused devices: <none>
[student@student-13 ~]$ sudo /sbin/mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Tue Nov 24 13:04:25 2026
        Raid Level : raid1
        Array Size : 261120 (255.00 MiB 267.39 MB)
     Used Dev Size : 261120 (255.00 MiB 267.39 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent
 
     Intent Bitmap : Internal
 
       Update Time : Tue Nov 24 13:09:58 2026
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0
 
Consistency Policy : bitmap
 
              Name : student-13:0  (local to host student-13)
              UUID : 38c04c11:f4d46108:f8dfdb0f:e9ef0f54
            Events : 41
 
    Number   Major   Minor   RaidDevice State
       0       7        0        0      active sync   /dev/loop0
       1       7        1        1      active sync   /dev/loop1
[student@student-13 ~]$ ls /mnt/raid
lost+found  raid1_data.bin  raid1_degraded.txt  raid1_file1.txt  raid1_file2.txt
[student@student-13 ~]$ sha256sum -c ~/raid1_data.sha256
/mnt/raid/raid1_data.bin: OK
```
 
Массив снова в состоянии `clean`, `[UU]`, `Failed Devices : 0` — оба диска синхронизированы. Файл `raid1_degraded.txt`, записанный в деградированном режиме, сохранился, контрольная сумма `raid1_data.bin` совпадает.
 
## Завершение работы: удаление созданных объектов
 
Массив размонтирован и остановлен, метаданные стёрты, loop-устройства отключены, образы дисков, тестовый каталог и вспомогательные файлы удалены. Диски `sdb` и `sdc` в ходе работы не использовались:
 
```bash
[student@student-13 ~]$ sudo umount /mnt/raid
[student@student-13 ~]$ sudo /sbin/mdadm --stop /dev/md0
mdadm: stopped /dev/md0
[student@student-13 ~]$ sudo /sbin/mdadm --zero-superblock /dev/loop0 /dev/loop1
[student@student-13 ~]$ sudo /sbin/losetup -d /dev/loop0 /dev/loop1
[student@student-13 ~]$ command rm -f /tmp/raid_disk1.img /tmp/raid_disk2.img ~/raid1_data.sha256 ~/apt-mdadm.log
[student@student-13 ~]$ sudo rmdir /mnt/raid
[student@student-13 ~]$ cat /proc/mdstat
Personalities : [raid0] [raid1]
unused devices: <none>
[student@student-13 ~]$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   15G  0 disk
└─sda1   8:1    0   15G  0 part /
sdb      8:16   0    1G  0 disk
└─sdb1   8:17   0 1022M  0 part /mnt/newdisk
sdc      8:32   0    1G  0 disk
sr0     11:0    1 1024M  0 rom
[student@student-13 ~]$ ls /tmp/raid_disk*.img ~/raid1_data.sha256 /mnt/raid
ls: cannot access '/tmp/raid_disk*.img': No such file or directory
ls: cannot access '/home/student/raid1_data.sha256': No such file or directory
ls: cannot access '/mnt/raid': No such file or directory
```
 
Пакет `mdadm` остаётся установленным (он нужен для работы с RAID).
 
## 12. Файловые системы с RAID-подобными функциями
 
Классические файловые системы (ext4, XFS, NTFS, FAT) работают на одном блочном устройстве и RAID самостоятельно не обеспечивают: для этого нужен отдельный слой (`md`, LVM, аппаратный контроллер). Но есть файловые системы, которые сами управляют несколькими дисками:
 
- **Btrfs** — встроенные профили для данных и метаданных: `single`, `dup`, `raid0`, `raid1`, `raid1c3`, `raid1c4`, `raid10`, а также `raid5/raid6` (последние считаются проблемными из-за уязвимости write hole и не рекомендуются для важных данных). Есть контрольные суммы данных, снимки и самовосстановление (при наличии избыточности).
- **ZFS** — объединяет файловую систему и менеджер томов: пулы из `mirror` (аналог RAID1), `raidz1`, `raidz2`, `raidz3` (чётность, аналоги RAID5/6 и тройной чётности), чередование нескольких vdev, контрольные суммы всех блоков и автоматическое лечение повреждённых данных (self-healing).
- **Распределённые файловые системы** (Ceph, GlusterFS и др.) — хранят копии данных (репликацию) или кодирование со стиранием (erasure coding) на разных серверах.
- **ReFS** со Storage Spaces в Windows — зеркалирование и чётность на уровне системы хранения.
Отдельно стоит упомянуть **LVM**: это не файловая система, а менеджер логических томов, но он тоже умеет зеркалирование и чередование (например, `lvcreate --type raid1`).
 
## 13. Можно ли создать RAID во время установки Linux?
 
Да. Большинство установщиков Linux позволяют создать программный RAID на этапе разметки диска: в ручном (Custom/Manual) режиме разметки выбираются диски или разделы, из них собирается md-устройство нужного уровня, а затем на нём создаются файловые системы или LVM. Например, такая возможность есть в установщике Debian (пункт «Настроить программный RAID») и в Anaconda (RHEL, Fedora, где при добавлении точки монтирования можно выбрать тип устройства RAID и его уровень). Автоматическая разметка «весь диск» RAID, как правило, не создаёт.
 
Нюансы: чтобы система могла загружаться с RAID, `/boot` (и раздел EFI) обычно размещают на RAID1 с метаданными старого формата (`0.90` или `1.0`), которые лежат в конце диска и позволяют загрузчику видеть раздел как обычный (на это же указывает предупреждение `mdadm` из п. 7). Раздел EFI часто дублируют вручную на каждый диск. Данный пункт теоретический: установка Linux в рамках работы не выполнялась.



















