# Григорьев Сергей 241 группа
# Запрещаем

1) Запретите пользователю `user1` из предыдущего задания выполнять вход в систему.
2) Покажите, как именно вы это сделали, и как проверить состояние учётной записи.
3) Какие ещё способы запретить интерактивный вход вы знаете?
4) Можно ли создать двух пользователей с одинаковым `username`?
5) Можно ли создать двух пользователей с одинаковым UID? Что в таком случае увидит система?
6) Верните пользователю `user1` возможность входа.

## Критерий приёмки

В отчёте должны быть:
- команда, которой был запрещён вход `user1`;
- вывод, подтверждающий текущее состояние учётной записи;
- описание как минимум двух способов запрета интерактивного входа;
- ответ на вопрос о возможности одинаковых `username`;
- ответ на вопрос о возможности одинаковых UID и краткое описание последствий;
- команда восстановления возможности входа;
- подтверждение, что после восстановления вход снова разрешён.

# Отчёт по практической работе: Запрещаем

## 0. Исходное состояние `user1`

```bash
[student@student-13 ~]$ getent passwd user1
user1:x:1001:1002::/home/user1:/bin/bash
[student@student-13 ~]$ sudo passwd -S user1
Password set, yescrypt encryption.
[student@student-13 ~]$ sudo chage -l user1
Last password change                                    : Nov 23, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : -1
Maximum number of days between password change          : -1
Number of days of warning before password expires       : -1
```

Учётная запись активна: пароль задан, срок действия не ограничен, оболочка `/bin/bash`.

## 1. Запрет входа

```bash
[student@student-13 ~]$ sudo usermod -L user1
```

Команда `usermod -L` ставит символ `!` перед хэшем пароля, и вход по паролю становится невозможен. Сам хэш сохраняется, поэтому при разблокировке пароль останется прежним.

## 2. Проверка состояния учётной записи

```bash
[student@student-13 ~]$ sudo passwd -S user1
Locked password, yescrypt encryption.
```

Статус `Locked password` подтверждает блокировку.

Проверка хэша пароля. На этом сервере пароли хранятся в отдельных файлах `/etc/tcb/<пользователь>/shadow`, в общем `/etc/shadow` записи `user1` нет:

```bash
[student@student-13 ~]$ sudo awk -F: '$1=="user1"{print $1":"substr($2,1,2)}' /etc/tcb/user1/shadow
user1:!$
```

Поле пароля начинается с `!` перед `$`, значит, пароль заблокирован. (Выводятся только первые два символа поля, сам хэш не показывается.)

Проверка попыткой входа от обычного пользователя (`root` обходит проверку пароля, поэтому проверяем от `student`):

```bash
[student@student-13 ~]$ su - user1
Password:
su: Authentication failure
```

Вход отклонён.

## 3. Другие способы запретить интерактивный вход

### Способ 1. Оболочка `nologin`

```bash
[student@student-13 ~]$ sudo usermod -s /usr/sbin/nologin user1
[student@student-13 ~]$ getent passwd user1
user1:x:1001:1002::/home/user1:/usr/sbin/nologin
[student@student-13 ~]$ sudo su - user1
This account is currently not available.
```

Вместо оболочки пользователю выдаётся сообщение, и сессия сразу завершается.

### Способ 2. Истечение срока действия учётной записи

Оболочка возвращена в `/bin/bash`, затем срок действия учётки закрыт датой в прошлом:

```bash
[student@student-13 ~]$ sudo usermod -s /bin/bash user1
[student@student-13 ~]$ sudo chage -E 0 user1
[student@student-13 ~]$ sudo chage -l user1 | grep 'Account expires'
Account expires                                         : Jan 01, 1970
```

Проверка через `sudo su - user1` показывает, что `root` обходит это ограничение (вход выполняется, ошибка помечена как `Ignored`):

```bash
[student@student-13 ~]$ sudo su - user1
Your account has expired; please contact your system administrator.
Account management:- User account has expired
(Ignored)
[user1@student-13 ~]$ exit
logout
```

Чтобы проверить именно запрет по сроку, пароль временно разблокирован (иначе вход отклонился бы на проверке пароля), и выполнена попытка входа от обычного пользователя:

```bash
[student@student-13 ~]$ sudo usermod -U user1
[student@student-13 ~]$ sudo passwd -S user1
Password set, yescrypt encryption.
[student@student-13 ~]$ su - user1
Password:
Your account has expired; please contact your system administrator.
su: User account has expired
```

Даже с верным паролем вход запрещён.

### Другие известные способы (описание, на сервере не выполнялись)

- **`DenyUsers user1` в `/etc/ssh/sshd_config`** запрещает вход пользователя только по SSH.
- **Модуль PAM `pam_access`** (правила в `/etc/security/access.conf`, например `- : user1 : ALL`) запрещает вход в зависимости от настроек PAM.
- **Файл `/etc/nologin`** запрещает вход всем, кроме `root`, на время технических работ.

Блокировка пароля (`usermod -L`) не запрещает вход по SSH-ключу, поэтому для полного запрета её сочетают с `nologin` или `chage -E 0`.

## 4. Можно ли создать двух пользователей с одинаковым `username`?

**Нет.**

```bash
[student@student-13 ~]$ sudo useradd user1
useradd: user 'user1' already exists
[student@student-13 ~]$ echo "код возврата: $?"
код возврата: 9
[student@student-13 ~]$ getent passwd | grep -c '^user1:'
1
```

Повторное создание отклонено (код 9), запись `user1` остаётся одна.

## 5. Можно ли создать двух пользователей с одинаковым UID?

**Да, но только с ключом `-o` (non-unique).** Без него система отказывает:

```bash
[student@student-13 ~]$ sudo useradd -m -u 1001 user3; echo "код возврата: $?"
useradd: UID 1001 is not unique
код возврата: 4
```

С ключом `-o` пользователь создаётся:

```bash
[student@student-13 ~]$ sudo useradd -m -o -u 1001 -s /bin/bash user3; echo "код возврата: $?"
код возврата: 0
[student@student-13 ~]$ getent passwd | awk -F: '$3==1001'
user1:x:1001:1002::/home/user1:/bin/bash
user3:x:1001:1005::/home/user3:/bin/bash
```

Что видит система:

```bash
[student@student-13 ~]$ id user1; id user3
uid=1001(user1) gid=1002(user1) groups=1002(user1),10(wheel),1004(labusers)
uid=1001(user1) gid=1005(user3) groups=1002(user1)
[student@student-13 ~]$ sudo -u user3 touch /home/user3/test_uid.txt
[student@student-13 ~]$ sudo ls -l /home/user3/test_uid.txt
-rw-r--r-- 1 user1 user3 0 Nov 23 11:53 /home/user3/test_uid.txt
[student@student-13 ~]$ sudo ls -ln /home/user3/test_uid.txt
-rw-r--r-- 1 1001 1005 0 Nov 23 11:53 /home/user3/test_uid.txt
[student@student-13 ~]$ sudo stat -c 'owner=%U uid=%u group=%G gid=%g' /home/user3/test_uid.txt
owner=user1 uid=1001 group=user3 gid=1005
[student@student-13 ~]$ ls -ld /home/user3
drwx------ 8 user1 user3 4096 Nov 23 11:53 /home/user3
```

**Последствия.** Ядро хранит только числовой UID, а имя подставляется по первой найденной записи в `/etc/passwd`:

- `id user3` показывает `uid=1001(user1)`;
- файл, созданный от `user3`, отображается как принадлежащий `user1` (по числам: UID 1001, GID 1005);
- домашний каталог `user3` тоже числится за `user1`;
- у обоих пользователей одинаковые права на файлы, различить их по владельцу и в журналах нельзя.

Это ошибка конфигурации или дыра в безопасности. Тестовый пользователь удалён:

```bash
[student@student-13 ~]$ sudo userdel -r user3; echo "код возврата: $?"
код возврата: 0
[student@student-13 ~]$ getent passwd | awk -F: '$3==1001'
user1:x:1001:1002::/home/user1:/bin/bash
[student@student-13 ~]$ getent passwd user3; echo "код возврата getent: $?"
код возврата getent: 2
[student@student-13 ~]$ ls -ld /home/user1
drwx------ 8 user1 user1 4096 Nov 23 11:50 /home/user1
```

## 6. Восстановление возможности входа

Применённые команды восстановления:

```bash
sudo usermod -U user1          # снять блокировку пароля (выполнена в разделе 3)
sudo usermod -s /bin/bash user1  # вернуть оболочку (выполнена в разделе 3)
sudo chage -E -1 user1         # снять ограничение срока действия учётки
```

Подтверждение, что вход снова разрешён:

```bash
[student@student-13 ~]$ sudo chage -E -1 user1
[student@student-13 ~]$ sudo chage -l user1 | grep 'Account expires'
Account expires                                         : never
[student@student-13 ~]$ sudo passwd -S user1
Password set, yescrypt encryption.
[student@student-13 ~]$ sudo awk -F: '$1=="user1"{print $1":"substr($2,1,2)}' /etc/tcb/user1/shadow
user1:$y
[student@student-13 ~]$ getent passwd user1
user1:x:1001:1002::/home/user1:/bin/bash
[student@student-13 ~]$ su - user1
Password:
[user1@student-13 ~]$ whoami
user1
[user1@student-13 ~]$ exit
logout
```

Блокировка снята (`Password set`, перед хэшем нет `!`), срок учётки снова `never`, оболочка `/bin/bash`, а вход по паролю от обычного пользователя выполняется. Состояние `user1` совпадает с исходным.
