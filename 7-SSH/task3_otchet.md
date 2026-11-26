# Ключи

1) Что такое SSH-ключи и зачем они нужны?
2) Как создать пару ключей?
3) Создайте пару публичный/приватный ключ `ed25519`. Где хранятся эти файлы?
4) Скопируйте публичный ключ на ваш сервер. В каком файле на сервере он будет храниться?
5) Подключитесь к серверу. Запрашивается ли пароль учётной записи?
6) Проверьте права на каталог `~/.ssh` и файл `authorized_keys`.
7) Запретите парольную SSH-аутентификацию для обычных пользователей, оставив вход по ключам.
8) Не закрывая текущую рабочую сессию, откройте вторую консоль и убедитесь, что вход по ключу работает.
9) Проверьте, что парольная аутентификация действительно запрещена.

## Критерий приёмки

В отчёте должны быть:
- команда генерации ключа `ed25519`;
- пути к публичному и приватному ключам;
- команда копирования публичного ключа на сервер;
- путь к `authorized_keys`;
- права на `~/.ssh` и `authorized_keys`;
- фрагмент конфигурации, отключающий парольную аутентификацию;
- подтверждение успешного входа по ключу;
- подтверждение отказа при попытке парольной аутентификации;
- краткое пояснение, почему приватный ключ нельзя передавать другим пользователям.


# Отчёт по практической работе: Ключи


## 1. Что такое SSH-ключи и зачем они нужны

SSH-ключи — это пара связанных криптографических файлов:

- **приватный ключ** хранится только у владельца и никому не передаётся;
- **публичный ключ** можно свободно копировать на серверы, куда нужен вход.

При подключении сервер проверяет, что клиент владеет приватным ключом, соответствующим публичному ключу из `authorized_keys`. Сам приватный ключ по сети не передаётся.

Зачем нужны:
- вход без ввода пароля учётной записи;
- безопасность выше, чем у паролей: ключ невозможно подобрать перебором так же, как слабый пароль;
- можно полностью отключить парольный вход на сервере и закрыть атаки перебором паролей;
- удобство автоматизации (скрипты, резервное копирование).

## 2. Как создать пару ключей

Пара создаётся утилитой `ssh-keygen`:

```bash
ssh-keygen -t ed25519 -f <путь_к_ключу> -C "<комментарий>"
```

- `-t ed25519` — тип ключа (современный алгоритм на эллиптических кривых);
- `-f` — путь и имя приватного ключа (публичный получит суффикс `.pub`);
- `-C` — комментарий для опознания ключа.

Утилита также предлагает задать passphrase — парольную фразу, которая шифрует приватный ключ на диске.

## 3. Создание пары `ed25519` и расположение файлов

Сначала проверено содержимое `~/.ssh`:

```powershell
Get-ChildItem $env:USERPROFILE\.ssh
```

```text
    Каталог: C:\Users\vozre\.ssh

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        26.11.2026     13:46            158 config
-a----        08.10.2026     11:32            464 id_ed25519
-a----        08.10.2026     11:32            103 id_ed25519.pub
-a----        22.11.2026     20:11           1869 known_hosts
-a----        22.11.2026     20:08           1121 known_hosts.old
```

Пара `id_ed25519` уже существовала, поэтому, чтобы её не перезаписать, для лабораторной создана отдельная пара `id_ed25519_lab13` (passphrase оставлена пустой).

Команда генерации:

```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\id_ed25519_lab13" -C "student@lab13"
```

```text
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\vozre\.ssh\id_ed25519_lab13
Your public key has been saved in C:\Users\vozre\.ssh\id_ed25519_lab13.pub
The key fingerprint is:
SHA256:F0dj/qOF+FK943CBXxAW7p5vBp/247eWSRFCRfqDyQc student@lab13
The key's randomart image is:
+--[ED25519 256]--+
|            =.*+ |
|           + =.o |
|          . oE+ .|
|           +.*=o |
|        S o ++B+o|
|         . o =oB.|
|          . + Oo+|
|           . + X=|
|              ==B|
+----[SHA256]-----+
```

Проверка созданных файлов и публичного ключа:

```powershell
Get-ChildItem $env:USERPROFILE\.ssh\id_ed25519_lab13*
Get-Content $env:USERPROFILE\.ssh\id_ed25519_lab13.pub
```

```text
    Каталог: C:\Users\vozre\.ssh

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        26.11.2026     13:50            399 id_ed25519_lab13
-a----        26.11.2026     13:50             96 id_ed25519_lab13.pub

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIER/wF13CM2htUzv3FaaL8Nh5B2cA//D0xmO+41/stYw student@lab13
```

**Где хранятся файлы:**
- приватный ключ: `C:\Users\vozre\.ssh\id_ed25519_lab13`
- публичный ключ: `C:\Users\vozre\.ssh\id_ed25519_lab13.pub`

(на Linux это был бы каталог `~/.ssh/`).

## 4. Копирование публичного ключа на сервер

На Windows нет утилиты `ssh-copy-id`, поэтому ключ передан через `ssh`: команда создаёт на сервере `~/.ssh` с правами 700, дописывает строку ключа в конец `~/.ssh/authorized_keys` (лишний `\r` от PowerShell вырезается через `tr -d '\r'`) и выставляет права 600.

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519_lab13.pub | ssh lab13 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && tr -d '\r' >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

```text
student13@ternar.io's password:
student@192.168.6.113's password:
```

(при успехе команда ничего не выводит; пароли запрошены, так как ключ ещё не был установлен). Проверка на сервере:

```powershell
ssh lab13 "ls -ld ~/.ssh; ls -l ~/.ssh/authorized_keys; cat ~/.ssh/authorized_keys"
```

```text
student13@ternar.io's password:
student@192.168.6.113's password:
drwx------ 2 student student 4096 Nov 26 09:20 /home/student/.ssh
-rw------- 1 student student 261 Nov 26 09:52 /home/student/.ssh/authorized_keys
# OpenSSH authorized_keys file format is described in sshd(8) manual page.
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKX7URpoSsk8SmoppBWK3zdOWHi47jFGS5rfEZ0Hz7tl insec@arz
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIER/wF13CM2htUzv3FaaL8Nh5B2cA//D0xmO+41/stYw student@lab13
```

**Где хранится ключ на сервере:** `/home/student/.ssh/authorized_keys` (по одной строке на ключ). Наш ключ `student@lab13` добавлен последней строкой; уже присутствовавший ключ `insec@arz` не изменялся.

## 5. Подключение к серверу по ключу

Ключ указан явно (`-i`), а `IdentitiesOnly=yes` запрещает клиенту предлагать другие ключи:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519_lab13" -o IdentitiesOnly=yes lab13 "hostname; whoami"
```

```text
student13@ternar.io's password:
student-13
student
```

**Запрашивается ли пароль:** пароль запросил только jump-хост `ternar.io` (это отдельная машина с парольным входом). Пароль учётной записи `student` на сервере `192.168.6.113` **не запрашивался**: вход выполнен по ключу.

## 6. Права на `~/.ssh` и `authorized_keys`

Фактический вывод (получен в п. 4):

```text
drwx------ 2 student student 4096 Nov 26 09:20 /home/student/.ssh
-rw------- 1 student student 261 Nov 26 09:52 /home/student/.ssh/authorized_keys
```

- `~/.ssh`: `drwx------` (700) — доступ только у владельца;
- `authorized_keys`: `-rw-------` (600) — читать и писать может только владелец.

Если права слишком широкие, `sshd` (при `StrictModes`) может отказаться использовать ключи, а другие пользователи смогли бы подменить файл и добавить свой ключ.

## 7. Запрет парольной SSH-аутентификации

Изменение конфигурации `sshd` может отрезать доступ, поэтому порядок был таким: проверить текущее состояние, сделать резервную копию, изменить конфиг, проверить синтаксис и только потом перезапустить службу. Рабочая сессия на сервере всё время оставалась открытой.

**Текущее состояние (до изменений):**

```bash
sudo /usr/sbin/sshd -T | grep -Ei '^(passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication|usepam|permitrootlogin|authorizedkeysfile|maxauthtries) '
```

```text
usepam yes
maxauthtries 3
permitrootlogin no
pubkeyauthentication yes
passwordauthentication yes
kbdinteractiveauthentication no
authorizedkeysfile /etc/openssh/authorized_keys/%u /etc/openssh/authorized_keys2/%u .ssh/authorized_keys .ssh/authorized_keys2
```

Пароль был разрешён (`passwordauthentication yes`). В каталоге `/etc/openssh/sshd_config.d` обнаружен файл `99-student-password.conf`:

```bash
cat /etc/openssh/sshd_config.d/99-student-password.conf
```

```text
PasswordAuthentication yes
KbdInteractiveAuthentication yes
PubkeyAuthentication yes
UsePAM yes
```

Основной конфиг (читается только от root):

```bash
sudo grep -nEi '^\s*(PasswordAuthentication|KbdInteractiveAuthentication|UsePAM|PubkeyAuthentication|Include|Match)' /etc/openssh/sshd_config
```

```text
128:PasswordAuthentication yes
131:Match User student
132:    PasswordAuthentication yes
133:    KbdInteractiveAuthentication yes
```

Для пользователя `student` пароль разрешён отдельным блоком `Match User student`, поэтому правки одной глобальной строки недостаточно. Подтверждение через `sshd -T -C` (проверка с учётом `Match`):

```bash
sudo /usr/sbin/sshd -T -C user=student,host=localhost,addr=127.0.0.1 | grep -Ei '^(passwordauthentication|kbdinteractiveauthentication)'
```

```text
passwordauthentication yes
kbdinteractiveauthentication yes
```

**Резервная копия конфигурации:**

```bash
sudo cp -p /etc/openssh/sshd_config /etc/openssh/sshd_config.bak_keys
```

Фрагмент основного конфига до правки (управляемый блок отвечает за пароль для `student`):

```text
PasswordAuthentication yes

# BEGIN managed student password authentication
Match User student
    PasswordAuthentication yes
    KbdInteractiveAuthentication yes
    AuthenticationMethods any
# END managed student password authentication
```

**Изменение конфигурации.** В `sshd` для большинства параметров действует первое найденное значение, поэтому управляемый блок не правился: глобальная строка заменена на `no`, а собственный блок `Match User student` вставлен перед управляемым.

```bash
sudo sed -i 's/^PasswordAuthentication yes$/PasswordAuthentication no/' /etc/openssh/sshd_config
sudo sed -i '/^# BEGIN managed student password authentication/i # lab keys: only public key for student (before managed block, first match wins)\nMatch User student\n    PasswordAuthentication no\n    KbdInteractiveAuthentication no\n    AuthenticationMethods publickey\n' /etc/openssh/sshd_config
sudo /usr/sbin/sshd -t && echo "SYNTAX OK"
sudo /usr/sbin/sshd -T -C user=student,host=localhost,addr=127.0.0.1 | grep -Ei '^(passwordauthentication|kbdinteractiveauthentication|authenticationmethods)'
```

```text
SYNTAX OK
sshd: reprocess config line 140: "any" must appear alone in AuthenticationMethods
```

Простая проверка синтаксиса прошла, но проверка с учётом `Match` показала ошибку: `AuthenticationMethods` накапливает значения из всех подходящих блоков, а `any` не может стоять вместе с `publickey`. Служба в этот момент не перезапускалась, поэтому доступ не пострадал. Строка `AuthenticationMethods publickey` удалена из нашего блока. Запрет пароля от этого не ослаб: `PasswordAuthentication no` и `KbdInteractiveAuthentication no` в нашем блоке стоят первыми и побеждают.

```bash
sudo sed -i '/^    AuthenticationMethods publickey$/d' /etc/openssh/sshd_config
sudo /usr/sbin/sshd -t && echo "SYNTAX OK"
sudo sed -n '126,142p' /etc/openssh/sshd_config
sudo /usr/sbin/sshd -T -C user=student,host=localhost,addr=127.0.0.1 | grep -Ei '^(passwordauthentication|kbdinteractiveauthentication|authenticationmethods|pubkeyauthentication)'
```

```text
SYNTAX OK
#Match Group !secured,*
#       AuthorizedKeysFile /etc/openssh/authorized_keys/%u .ssh/authorized_keys
PasswordAuthentication no

# lab keys: only public key for student (before managed block, first match wins)
Match User student
    PasswordAuthentication no
    KbdInteractiveAuthentication no

# BEGIN managed student password authentication
Match User student
    PasswordAuthentication yes
    KbdInteractiveAuthentication yes
    AuthenticationMethods any
# END managed student password authentication
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
authenticationmethods any
```

**Фрагмент конфигурации, отключающий парольную аутентификацию:**

```text
PasswordAuthentication no

Match User student
    PasswordAuthentication no
    KbdInteractiveAuthentication no
```

Итог для `student`: пароль и keyboard-interactive отключены, вход по публичному ключу включён (`pubkeyauthentication yes`), поэтому единственный доступный способ входа — ключ. Значение `authenticationmethods any` остаётся от управляемого блока и ограничений не создаёт.

**Применение конфигурации** (только после успешной проверки):

```bash
sudo systemctl restart sshd
systemctl status sshd --no-pager | head -n 5
```

```text
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-11-26 10:00:28 UTC; 5s ago
 Invocation: 2e2bb411f99a4459be99655552d02781
    Process: 11254 ExecStartPre=/usr/bin/ssh-keygen -A (code=exited, status=0/SUCCESS)
```

## 8. Проверка входа по ключу из второй консоли

Рабочая сессия на сервере оставалась открытой. Во второй консоли PowerShell выполнено подключение по ключу:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519_lab13" -o IdentitiesOnly=yes lab13 "hostname; whoami"
```

```text
student13@ternar.io's password:
student-13
student
```

Вход по ключу работает и после перезапуска `sshd`: пароль запросил только jump-хост `ternar.io`, а на сервере `student-13` пароль не запрашивался.

## 9. Проверка запрета парольной аутентификации

Клиенту запрещено использовать ключи и разрешён только пароль:

```powershell
ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password lab13 "hostname"
```

```text
student13@ternar.io's password:
student@192.168.6.113: Permission denied (publickey).
```

Пароль запросил только jump-хост (его настройки не менялись). Сервер `192.168.6.113` не предложил ввести пароль и отклонил попытку: `Permission denied (publickey)`. Это значит, что парольная аутентификация на сервере действительно запрещена, допускается только публичный ключ.

## Почему приватный ключ нельзя передавать другим пользователям

Приватный ключ — это то, что доказывает серверу личность владельца. Любой, у кого есть копия приватного ключа, сможет подключиться к любому серверу, на котором установлен соответствующий публичный ключ, от имени владельца и без пароля (если у ключа нет passphrase). Скомпрометированный ключ придётся отзывать на всех серверах, а действия злоумышленника под этим ключом нельзя отличить от действий владельца. Поэтому приватный ключ хранится только у владельца с правами доступа только для него (при возможности защищается passphrase), а на серверы копируется исключительно публичный ключ, раскрытие которого безопасно.
