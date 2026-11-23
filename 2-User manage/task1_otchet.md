# Григорьев Сергей 241 группа

# Управление пользователями

1) Добавьте пользователей `user1` и `user2`:
   1.1) `user1` — оболочка bash;
   1.2) `user2` — оболочка sh;
   1.3) установите им пароли.

2) Создайте отдельную группу `labusers`.
   2.1) Добавьте `user1` в группу администраторов системы.
   2.2) Добавьте `user1` и `user2` в группу `labusers`.

3) Что такое права доступа? Выведите права доступа на файлы в домашней директории одного из пользователей.

4) Создайте файл, доступный на чтение всем пользователям, а на запись только владельцу.
   4.1) Временно выставьте права `777`.
   4.2) Объясните, почему такая настройка обычно избыточна.
   4.3) Верните минимально необходимые права.

5) Как называется встроенная учётная запись администратора в Linux?

6) Как выполнить одну команду с административными правами?

7) Есть ли у суперпользователя абсолютная возможность выполнить любую операцию? Приведите пример ограничения, существующего не из-за обычных файловых прав.

8) Удалите пользователя `user2`, сохранив его домашний каталог. Затем покажите, что пользователь действительно удалён.

9) Как изменить владельца и группу файла или каталога? Измените владельца созданного в пункте 4 файла.

10) Покажите UID, основной GID и дополнительные группы `user1`.


# Отчёт по практической работе: Управление пользователями

## 1. Добавление пользователей `user1` и `user2`

Перед созданием проверено, что пользователи и группа ещё не существуют, а также определены доступные оболочки и группа администраторов.

Использованные команды:

```bash
whoami
grep -E '^(NAME|VERSION)=' /etc/os-release
getent passwd user1 user2
getent group labusers sudo wheel
cat /etc/shells
```

Полученный результат:

```bash
[student@student-13 ~]$ whoami
student
[student@student-13 ~]$ grep -E '^(NAME|VERSION)=' /etc/os-release
NAME="ALT Linux"
VERSION="20250612"
[student@student-13 ~]$ getent passwd user1 user2
[student@student-13 ~]$ getent group labusers sudo wheel
wheel:x:10:root,student
[student@student-13 ~]$ cat /etc/shells
/bin/sh
/usr/bin/sh
/bin/bash
/usr/bin/bash
/bin/ash
/usr/bin/ash
/bin/bsh
/usr/bin/bsh
/bin/csh
/usr/bin/csh
/bin/tcsh
/usr/bin/tcsh
/bin/ksh
/usr/bin/ksh
/bin/zsh
/usr/bin/zsh
/bin/xonsh
/usr/bin/xonsh
```

Пользователей `user1`, `user2` и группы `labusers` в системе нет. В ALT Linux группа администраторов называется `wheel`.

### 1.1–1.2. Создание пользователей и проверка оболочек

Использованные команды:

```bash
sudo useradd -m -s /bin/bash user1
sudo useradd -m -s /bin/sh user2
grep -E '^(user1|user2):' /etc/passwd
```

Полученный результат:

```bash
[student@student-13 ~]$ sudo useradd -m -s /bin/bash user1
[student@student-13 ~]$ sudo useradd -m -s /bin/sh user2
[student@student-13 ~]$ grep -E '^(user1|user2):' /etc/passwd
user1:x:1001:1002::/home/user1:/bin/bash
user2:x:1002:1003::/home/user2:/bin/sh
```

Ключ `-m` создаёт домашний каталог, `-s` задаёт оболочку. У `user1` оболочка `/bin/bash`, у `user2` — `/bin/sh`.

### 1.3. Установка паролей

Использованные команды:

```bash
sudo passwd user1
sudo passwd user2
```

Полученный результат (сообщения о требованиях к паролю сокращены, предложенный системой пароль скрыт):

```bash
[student@student-13 ~]$ sudo passwd user1
passwd: updating all authentication tokens for user user1.

You can now choose the new password or passphrase.
...
Alternatively, if no one else can see your terminal now, you can pick this as
your password: "<скрыто>".

Enter new password:
Re-type new password:
passwd: all authentication tokens updated successfully.
[student@student-13 ~]$ sudo passwd user2
passwd: updating all authentication tokens for user user2.

You can now choose the new password or passphrase.
...
Alternatively, if no one else can see your terminal now, you can pick this as
your password: "<скрыто>".

Enter new password:
Re-type new password:
passwd: all authentication tokens updated successfully.
```

Пароли обоим пользователям успешно установлены. Система проверяет сложность пароля, поэтому слишком простые пароли отклоняются.

## 2. Группа `labusers` и членство в группах

Использованные команды:

```bash
sudo groupadd labusers
sudo usermod -aG wheel user1
sudo usermod -aG labusers user1
sudo usermod -aG labusers user2
getent group wheel labusers
```

Полученный результат:

```bash
[student@student-13 ~]$ sudo groupadd labusers
[student@student-13 ~]$ sudo usermod -aG wheel user1
[student@student-13 ~]$ sudo usermod -aG labusers user1
[student@student-13 ~]$ sudo usermod -aG labusers user2
[student@student-13 ~]$ getent group wheel labusers
wheel:x:10:root,student,user1
labusers:x:1004:user1,user2
```

Создана группа `labusers` (GID 1004). `user1` добавлен в группу администраторов `wheel` (2.1), а `user1` и `user2` — в `labusers` (2.2). Ключ `-aG` добавляет пользователя в дополнительные группы, не затрагивая уже имеющиеся.

## 3. Права доступа

**Что такое права доступа.** Права доступа определяют, какие действия (чтение `r`, запись `w`, выполнение `x`) могут выполнять с файлом или каталогом три категории субъектов: владелец, группа владельца и все остальные пользователи.

Использованная команда:

```bash
sudo ls -la /home/user1
```

Полученный результат:

```bash
[student@student-13 ~]$ sudo ls -la /home/user1
total 56
drwx------ 8 user1 user1 4096 Nov 23 11:13 .
drwxr-xr-x 5 root  root  4096 Nov 23 11:13 ..
-rw------- 1 user1 user1  217 Jul 26  2021 .bash_logout
-rw------- 1 user1 user1  259 Jul 26  2021 .bash_profile
-rw------- 1 user1 user1  188 Jul 26  2021 .bashrc
drwx------ 2 user1 user1 4096 Jul 26  2021 .cache
drwx------ 2 user1 user1 4096 Jul 26  2021 .config
drwx------ 3 user1 user1 4096 Oct  7  2025 .local
-rw------- 1 user1 user1   17 Jul 26  2021 .lpoptions
drwx------ 3 user1 user1 4096 Oct  7  2025 .mutt
-rw------- 1 user1 user1  121 Jul 26  2021 .rpmmacros
drwx------ 2 user1 user1 4096 Oct  7  2025 .ssh
-rwx------ 1 user1 user1  319 Jul 26  2021 .xprofile
drwx------ 2 user1 user1 4096 Jul 26  2021 .xsession.d
```

Домашний каталог `user1` имеет права `drwx------` (700), файлы — `-rw-------` (600). Владелец и группа у всех объектов — `user1`. Доступ есть только у владельца: группа и остальные пользователи не имеют никаких прав.

## 4. Файл с правами: читать всем, писать только владельцу

Так как домашний каталог `user1` закрыт правами 700, другие пользователи не смогли бы до него добраться. Поэтому файл создан в `/tmp`, доступном всем.

Использованные команды:

```bash
echo "Hello, Linux!" > /tmp/lab_test.txt
ls -l /tmp/lab_test.txt
chmod 644 /tmp/lab_test.txt
ls -l /tmp/lab_test.txt
```

Полученный результат:

```bash
[student@student-13 ~]$ echo "Hello, Linux!" > /tmp/lab_test.txt
[student@student-13 ~]$ ls -l /tmp/lab_test.txt
-rw-r--r-- 1 student student 14 Nov 23 11:18 /tmp/lab_test.txt
[student@student-13 ~]$ chmod 644 /tmp/lab_test.txt
[student@student-13 ~]$ ls -l /tmp/lab_test.txt
-rw-r--r-- 1 student student 14 Nov 23 11:18 /tmp/lab_test.txt
```

Права `644` (`rw-r--r--`): владелец читает и пишет, группа и остальные только читают. Исходные права файла уже были такими (по умолчанию из `umask`), поэтому `chmod 644` их не изменил.

### 4.1. Временная установка прав `777`

Использованные команды:

```bash
chmod 777 /tmp/lab_test.txt
ls -l /tmp/lab_test.txt
```

Полученный результат:

```bash
[student@student-13 ~]$ chmod 777 /tmp/lab_test.txt
[student@student-13 ~]$ ls -l /tmp/lab_test.txt
-rwxrwxrwx 1 student student 14 Nov 23 11:18 /tmp/lab_test.txt
```

### 4.2. Почему `777` обычно избыточна

Права `777` разрешают всем пользователям системы читать, изменять и выполнять файл. Запись для группы и остальных позволяет любому постороннему пользователю изменить или испортить содержимое, а право выполнения для обычного текстового файла не нужно вовсе. Это нарушает принцип наименьших привилегий и создаёт риск для безопасности. Для такого файла достаточно `644`.

### 4.3. Возврат минимально необходимых прав

Использованные команды:

```bash
chmod 644 /tmp/lab_test.txt
ls -l /tmp/lab_test.txt
```

Полученный результат:

```bash
[student@student-13 ~]$ chmod 644 /tmp/lab_test.txt
[student@student-13 ~]$ ls -l /tmp/lab_test.txt
-rw-r--r-- 1 student student 14 Nov 23 11:18 /tmp/lab_test.txt
```

Состояние прав файла:

| Этап | Права |
|------|-------|
| До изменения (644) | `-rw-r--r--` |
| При временной установке 777 | `-rwxrwxrwx` |
| После возврата к 644 | `-rw-r--r--` |

## 5. Встроенная учётная запись администратора

Встроенная учётная запись администратора в Linux называется **root** (суперпользователь), её UID равен 0.

Использованная команда:

```bash
id root
```

Полученный результат:

```bash
[student@student-13 ~]$ id root
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),19(proc)
```

## 6. Выполнение одной команды с административными правами

Для этого используется `sudo`: команда выполняется от имени root, а сессия остаётся у обычного пользователя.

Использованные команды:

```bash
sudo whoami
whoami
```

Полученный результат:

```bash
[student@student-13 ~]$ sudo whoami
root
[student@student-13 ~]$ whoami
student
```

`sudo whoami` вернул `root`, а следующий `whoami` показал `student`: права повышены только на одну команду.

## 7. Возможности суперпользователя

Нет, у root нет абсолютной возможности выполнить любую операцию. Пример ограничения не из-за обычных прав `rwx` — атрибут неизменяемости файла (`chattr +i`): пока он установлен, файл нельзя удалить, переименовать или изменить даже от root. Другие примеры — системы мандатного контроля доступа (SELinux, AppArmor) и файловые системы, смонтированные только для чтения.

Использованные команды:

```bash
touch ~/immut_test.txt
sudo chattr +i ~/immut_test.txt
lsattr ~/immut_test.txt
sudo rm -f ~/immut_test.txt
sudo chattr -i ~/immut_test.txt
sudo rm -f ~/immut_test.txt
ls -l ~/immut_test.txt
```

Полученный результат:

```bash
[student@student-13 ~]$ touch ~/immut_test.txt
[student@student-13 ~]$ sudo chattr +i ~/immut_test.txt
[student@student-13 ~]$ lsattr ~/immut_test.txt
----i---------e------- /home/student/immut_test.txt
[student@student-13 ~]$ sudo rm -f ~/immut_test.txt
rm: cannot remove '/home/student/immut_test.txt': Operation not permitted
[student@student-13 ~]$ sudo chattr -i ~/immut_test.txt
[student@student-13 ~]$ sudo rm -f ~/immut_test.txt
[student@student-13 ~]$ ls -l ~/immut_test.txt
ls: cannot access '/home/student/immut_test.txt': No such file or directory
```

Удаление файла с атрибутом `i` от имени root завершилось ошибкой `Operation not permitted`. После снятия атрибута файл удалился. Тестовый файл удалён, лишнего на сервере не осталось.

## 8. Удаление `user2` с сохранением домашнего каталога

Команда `userdel` без ключа `-r` удаляет учётную запись, но оставляет домашний каталог.

Использованные команды:

```bash
sudo userdel user2
id user2
getent passwd user2
ls -ld /home/user2
getent group labusers
```

Полученный результат:

```bash
[student@student-13 ~]$ sudo userdel user2
[student@student-13 ~]$ id user2
id: ‘user2’: no such user
[student@student-13 ~]$ getent passwd user2
[student@student-13 ~]$ ls -ld /home/user2
drwx------ 8 1002 1003 4096 Nov 23 11:13 /home/user2
[student@student-13 ~]$ getent group labusers
labusers:x:1004:user1
```

Пользователь удалён: `id` сообщает, что такого пользователя нет, а `getent passwd` ничего не выводит. Из группы `labusers` он тоже исчез. Каталог `/home/user2` сохранился, но владелец и группа показаны числами (`1002 1003`), так как соответствующих имён в системе больше нет.

## 9. Изменение владельца и группы файла

Владелец и группа меняются командой `chown владелец:группа объект` (только от root). Для каталогов с содержимым добавляется ключ `-R`. Изменим владельца и группу файла из пункта 4.

Использованные команды:

```bash
ls -l /tmp/lab_test.txt
sudo chown user1:labusers /tmp/lab_test.txt
ls -l /tmp/lab_test.txt
```

Полученный результат:

```bash
[student@student-13 ~]$ ls -l /tmp/lab_test.txt
-rw-r--r-- 1 student student 14 Nov 23 11:18 /tmp/lab_test.txt
[student@student-13 ~]$ sudo chown user1:labusers /tmp/lab_test.txt
[student@student-13 ~]$ ls -l /tmp/lab_test.txt
-rw-r--r-- 1 user1 labusers 14 Nov 23 11:18 /tmp/lab_test.txt
```

Владелец файла изменился с `student` на `user1`, группа — с `student` на `labusers`. Права доступа остались прежними.

## 10. UID, основной GID и дополнительные группы `user1`

Использованные команды:

```bash
id user1
groups user1
getent passwd user1
```

Полученный результат:

```bash
[student@student-13 ~]$ id user1
uid=1001(user1) gid=1002(user1) groups=1002(user1),10(wheel),1004(labusers)
[student@student-13 ~]$ groups user1
user1 : user1 wheel labusers
[student@student-13 ~]$ getent passwd user1
user1:x:1001:1002::/home/user1:/bin/bash
```

У `user1` UID равен 1001, основной GID — 1002 (группа `user1`), дополнительные группы — `wheel` (10) и `labusers` (1004).

## Назначение прав `r`, `w`, `x`

Для файлов:

- `r` (read) — чтение содержимого файла;
- `w` (write) — изменение и перезапись содержимого файла;
- `x` (execute) — запуск файла как программы или скрипта.

Для каталогов:

- `r` (read) — просмотр списка файлов в каталоге;
- `w` (write) — создание, удаление и переименование файлов внутри каталога;
- `x` (execute) — вход в каталог (`cd`) и доступ к файлам внутри по известным именам.

## Очистка

Тестовый файл `/tmp/lab_test.txt` удалён.

```bash
[student@student-13 ~]$ sudo rm -f /tmp/lab_test.txt
[student@student-13 ~]$ ls -l /tmp/lab_test.txt
ls: cannot access '/tmp/lab_test.txt': No such file or directory
```

## Критерий приёмки

- команды создания `user1`, `user2` и группы `labusers` — пункты 1 и 2;
- вывод, подтверждающий оболочки пользователей, — пункт 1.1–1.2 (`/etc/passwd`);
- вывод `id` для подтверждения UID, GID и групп — пункт 10;
- команда и результат настройки прав на созданный файл — пункт 4;
- состояние прав файла до и после возврата от `777` к минимально необходимым — пункт 4 (таблица);
- команда изменения владельца и группы файла — пункт 9;
- подтверждение удаления `user2` — пункт 8;
- краткое пояснение назначения `r`, `w`, `x` для файла и каталога — раздел «Назначение прав `r`, `w`, `x`».











