# Настраиваем

1) Какой порт по умолчанию использует SSH?
2) Можно ли изменить его? Если да — как?
3) Какая служба отвечает за обработку входящих SSH-подключений?
4) Какой файл конфигурации отвечает за настройку SSH-сервера?
5) Подключитесь по SSH к предоставленному серверу.
6) Временно разрешите подключение к серверу пользователем `root`, продемонстрируйте работу, а после проверки верните безопасное значение. Объясните, почему постоянный root-login нежелателен.
7) Измените допустимое количество ошибок аутентификации до разрыва соединения и покажите действующую настройку.
8) Создайте пользователя `ssh-user` и подключитесь им к серверу.
9) Ограничьте `ssh-user` возможность подключения к серверу.
10) Покажите и объясните, как именно вы ограничили вход.
11) Что хранится в файле `known_hosts`?
12) Найдите ключ хоста вашего сервера в `known_hosts`.
13) Проверьте конфигурацию `sshd` на ошибки до перезапуска службы.

## Критерий приёмки

В отчёте должны быть:
- используемый порт SSH;
- путь к конфигурационному файлу SSH-сервера;
- команда подключения к серверу;
- фрагмент изменённой конфигурации root-login и подтверждение возврата безопасного значения;
- настройка ограничения количества ошибок аутентификации;
- команды создания `ssh-user`;
- настройка, ограничивающая вход `ssh-user`;
- результат проверки конфигурации `sshd` перед перезапуском;
- запись из `known_hosts`, относящаяся к используемому серверу;
- краткое пояснение назначения `known_hosts`.


## Отчёт по практической работе: Настраиваем


## 1. Порт SSH по умолчанию

Использованная команда:

```bash
sudo ss -tlnp | grep sshd
```

Полученный результат:

```text
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=671,fd=3))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=671,fd=4))
```

SSH-сервер слушает порт **22/tcp** на всех IPv4- и IPv6-адресах.

## 2. Можно ли изменить порт

Да. Порт задаётся директивой `Port` в конфигурационном файле `sshd`. В файле сервера строка закомментирована, поэтому действует значение по умолчанию:

```bash
sudo grep -nEi '^\s*#?\s*Port\b' /etc/openssh/sshd_config
sudo /usr/sbin/sshd -T | grep -i '^port'
```

```text
13:#Port 22
port 22
```

Чтобы сменить порт, нужно раскомментировать строку и указать другой номер (например, `Port 2222`), проверить конфигурацию командой `sshd -t` и перечитать службу. Клиент после этого подключается с ключом `-p <порт>`. В рамках работы порт не менялся.

## 3. Служба обработки входящих SSH-подключений

Использованная команда:

```bash
systemctl status sshd --no-pager
```

Полученный результат (фрагмент):

```text
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-11-24 11:16:55 UTC; 1 day 21h ago
   Main PID: 671 (sshd)
             └─671 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
```

За подключения отвечает служба **`sshd.service`** (демон `/usr/sbin/sshd`), она запущена и включена в автозагрузку.

## 4. Файл конфигурации SSH-сервера

Использованная команда:

```bash
ls -l /etc/openssh/sshd_config /etc/ssh/sshd_config
```

Полученный результат:

```text
ls: cannot access '/etc/ssh/sshd_config': No such file or directory
-rw------- 1 root root 4225 Nov 16 13:37 /etc/openssh/sshd_config
```

Конфигурация SSH-сервера находится в **`/etc/openssh/sshd_config`** (каталога `/etc/ssh/` в этой системе нет).

Перед изменениями сделана резервная копия:

```bash
sudo cp -a /etc/openssh/sshd_config /etc/openssh/sshd_config.bak
```

Значения до изменений (строки в файле и действующие настройки):

```bash
sudo grep -nEi '^\s*#?\s*(Port|PermitRootLogin|MaxAuthTries|AllowUsers|AllowGroups|DenyUsers|Include)\b' /etc/openssh/sshd_config
sudo /usr/sbin/sshd -T | grep -Ei '^(port|permitrootlogin|maxauthtries|allowusers|allowgroups|denyusers)'
```

```text
13:#Port 22
32:#PermitRootLogin without-password
34:#MaxAuthTries 6
110:#AllowGroups wheel users
port 22
maxauthtries 6
permitrootlogin without-password
```

## 5. Подключение к серверу по SSH

Команда подключения (выполняется на моём компьютере, подключение через jump-хост):

```bash
ssh -J student13@ternar.io:8053 student@192.168.6.113
```

Проверка на сервере:

```bash
echo $SSH_CONNECTION; whoami; hostname
```

```text
192.168.6.83 43994 192.168.6.113 22
student
student-13
```

Подключение выполнено: пользователь `student`, хост `student-13`, сервер принимает подключение на `192.168.6.113:22`.

## 6. Временное разрешение входа root и возврат безопасного значения

До изменений действовало `PermitRootLogin without-password` (root по паролю войти не может). Для чистой проверки пользователю `root` был задан известный пароль (с разрешения преподавателя):

```bash
sudo passwd root
```

```text
Enter new password:
Re-type new password:
passwd: all authentication tokens updated successfully.
```

**До изменения** конфигурации попытка входа root по паролю отклоняется даже с верным паролем:

```bash
ssh -o PreferredAuthentications=password root@localhost
```

```text
root@localhost's password:
ssh: Permission denied, please try again.
root@localhost's password:
```

**Временно разрешаем вход root** (строка 32), проверяем конфигурацию и перечитываем службу:

```bash
sudo sed -i '32s/.*/PermitRootLogin yes/' /etc/openssh/sshd_config
sudo grep -n 'PermitRootLogin' /etc/openssh/sshd_config
sudo /usr/sbin/sshd -t && echo "CONFIG OK" && sudo systemctl reload sshd
sudo /usr/sbin/sshd -T | grep -i permitrootlogin
```

```text
32:PermitRootLogin yes
77:# the setting of "PermitRootLogin prohibit-password".
CONFIG OK
permitrootlogin yes
```

**Демонстрация работы** — вход root по паролю:

```bash
ssh -o PreferredAuthentications=password root@localhost
whoami; hostname
exit
```

```text
root@localhost's password:
[root@student-13 ~]# whoami; hostname
root
student-13
[root@student-13 ~]# exit
logout
Connection to localhost closed.
```

**Возвращаем безопасное значение** (`no` — вход root по SSH запрещён полностью):

```bash
sudo sed -i '32s/.*/PermitRootLogin no/' /etc/openssh/sshd_config
sudo grep -n '^PermitRootLogin' /etc/openssh/sshd_config
sudo /usr/sbin/sshd -t && echo "CONFIG OK" && sudo systemctl reload sshd
sudo /usr/sbin/sshd -T | grep -i permitrootlogin
```

```text
32:PermitRootLogin no
CONFIG OK
permitrootlogin no
```

Повторная проверка — вход root по паролю снова отклоняется:

```bash
ssh -o PreferredAuthentications=password root@localhost
```

```text
root@localhost's password:
ssh: Permission denied, please try again.
root@localhost's password:
```

**Почему постоянный root-login нежелателен:**
- имя `root` есть в любой Linux-системе, поэтому атакующему нужно угадать только пароль, а подборы паролей к `root` идут постоянно;
- подобранный или утёкший пароль даёт полный контроль над сервером;
- все действия выполняются от одной учётной записи, и в журнале не видно, кто именно работал. При входе обычным пользователем и повышении прав через `sudo` действия привязаны к конкретному человеку.

## 7. Количество ошибок аутентификации (`MaxAuthTries`)

До изменения действовало значение по умолчанию `6`. Задаём `3`:

```bash
sudo sed -i '34s/.*/MaxAuthTries 3/' /etc/openssh/sshd_config
sudo grep -n '^MaxAuthTries' /etc/openssh/sshd_config
sudo /usr/sbin/sshd -t && echo "CONFIG OK" && sudo systemctl reload sshd
sudo /usr/sbin/sshd -T | grep -i maxauthtries
```

```text
34:MaxAuthTries 3
CONFIG OK
maxauthtries 3
```

Теперь сервер разрывает соединение после 3 неудачных попыток аутентификации (вместо 6), что затрудняет подбор пароля.

## 8. Создание пользователя `ssh-user` и подключение

Команды создания:

```bash
sudo useradd -m -s /bin/bash ssh-user
sudo passwd ssh-user
id ssh-user
```

```text
passwd: all authentication tokens updated successfully.
uid=1003(ssh-user) gid=1003(ssh-user) groups=1003(ssh-user)
```

Пользователь создан с домашним каталогом и оболочкой bash, в административные группы не добавлялся.

Подключение пользователем `ssh-user` (до введения ограничения):

```bash
ssh ssh-user@localhost
whoami; hostname
exit
```

```text
ssh-user@localhost's password:
[ssh-user@student-13 ~]$ whoami; hostname
ssh-user
student-13
[ssh-user@student-13 ~]$ exit
logout
Connection to localhost closed.
```

## 9. Ограничение входа `ssh-user`

В глобальную часть конфигурации (сразу после `MaxAuthTries`) добавлена директива `DenyUsers`:

```bash
sudo sed -i '34a DenyUsers ssh-user' /etc/openssh/sshd_config
sudo grep -n -E '^(MaxAuthTries|DenyUsers)' /etc/openssh/sshd_config
sudo /usr/sbin/sshd -t && echo "CONFIG OK" && sudo systemctl reload sshd
sudo /usr/sbin/sshd -T | grep -i denyusers
```

```text
34:MaxAuthTries 3
35:DenyUsers ssh-user
CONFIG OK
denyusers ssh-user
```

## 10. Как именно ограничен вход

Проверка — попытка входа `ssh-user` с верным паролем:

```bash
ssh ssh-user@localhost
```

```text
ssh-user@localhost's password:
ssh: Permission denied, please try again.
ssh-user@localhost's password:
```

Запись в журнале службы:

```bash
sudo journalctl -u sshd -n 6 --no-pager
```

```text
Nov 26 09:27:31 student-13 sshd[671]: Server listening on 0.0.0.0 port 22.
Nov 26 09:27:31 student-13 sshd[671]: Server listening on :: port 22.
Nov 26 09:27:43 student-13 sshd[10838]: User ssh-user from ::1 not allowed because listed in DenyUsers
Nov 26 09:27:48 student-13 sshd[10838]: pam_tcb(sshd:auth): Authentication failed for UNKNOWN USER from (uid=0)
Nov 26 09:27:50 student-13 sshd[10838]: Failed password for UNKNOWN USER from ::1 port 55396 ssh2
Nov 26 09:27:54 student-13 sshd[10838]: Connection closed by UNKNOWN USER ::1 port 55396 [preauth]
```

**Пояснение.** Директива `DenyUsers ssh-user` запрещает указанному пользователю вход по SSH. `sshd` проверяет её до проверки пароля, поэтому вход невозможен ни по паролю, ни по ключу, даже при верном пароле. Об этом говорит запись `not allowed because listed in DenyUsers`; сообщения `UNKNOWN USER` относятся к тому же запрету (сервер не раскрывает, существует ли такой пользователь). Сама учётная запись в системе остаётся, ограничение действует только на вход по SSH. Остальные пользователи (в том числе `student`) не затронуты.

## 11. Что хранится в `known_hosts`

`known_hosts` — файл клиента SSH (`~/.ssh/known_hosts`), в котором хранятся открытые ключи серверов, к которым пользователь уже подключался. Формат строки: имя хоста, тип ключа, сам ключ. При следующем подключении клиент сверяет ключ, который предъявил сервер, с записью в файле. Если ключ изменился, `ssh` предупреждает о возможной подмене сервера (атака «человек посередине»).

```bash
ls -l ~/.ssh/known_hosts
cat ~/.ssh/known_hosts
```

```text
-rw------- 1 student student 825 Nov 26 09:20 /home/student/.ssh/known_hosts
localhost ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKR53gauDuC9hS70RixRPT097P/5ePUw3nAnMfzq9hnS
```

В файле три строки для хоста `localhost` (типы `ssh-ed25519`, `ssh-rsa`, `ecdsa-sha2-nistp256`); выше приведена строка ed25519, остальные две имеют тот же формат. Запись создана автоматически при первом подключении:

```text
Warning: Permanently added 'localhost' (ED25519) to the list of known hosts.
```

## 12. Ключ хоста сервера в `known_hosts`

Поиск записей по имени хоста:

```bash
ssh-keygen -F localhost
```

```text
# Host localhost found: line 1
localhost ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKR53gauDuC9hS70RixRPT097P/5ePUw3nAnMfzq9hnS
# Host localhost found: line 2
(запись ssh-rsa)
# Host localhost found: line 3
(запись ecdsa-sha2-nistp256)
```

Сравнение с реальным ключом хоста сервера:

```bash
sudo ls /etc/openssh | grep host_
sudo cat /etc/openssh/ssh_host_ed25519_key.pub
ssh-keygen -lf /etc/openssh/ssh_host_ed25519_key.pub
ssh-keygen -lF localhost
```

```text
ssh_host_ecdsa_key
ssh_host_ecdsa_key.pub
ssh_host_ed25519_key
ssh_host_ed25519_key.pub
ssh_host_rsa_key
ssh_host_rsa_key.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKR53gauDuC9hS70RixRPT097P/5ePUw3nAnMfzq9hnS root@student-13
256 SHA256:4aO27sQWp+QRF47OBlyBPm30KSKT4w9LruJnXB1zFOQ root@student-13 (ED25519)
# Host localhost found: line 1
localhost ED25519 SHA256:4aO27sQWp+QRF47OBlyBPm30KSKT4w9LruJnXB1zFOQ
# Host localhost found: line 2
localhost RSA SHA256:zaobAIktmTKoC6L/gJoe/Z/zBzL+7L7m7ulDB0Cr2sc
# Host localhost found: line 3
localhost ECDSA SHA256:TExnd6KEGozFumMxA4ANWhnqjbzLeOqIsPu/Sr/NyM4
```

Запись ed25519 в `known_hosts` полностью совпадает с открытым ключом хоста `/etc/openssh/ssh_host_ed25519_key.pub` сервера `student-13`, отпечатки одинаковы (`SHA256:4aO27sQWp+QRF47OBlyBPm30KSKT4w9LruJnXB1zFOQ`). Значит, запись относится именно к ключу этого сервера.

## 13. Проверка конфигурации `sshd` до перезапуска

Перед каждым `reload` в ходе работы выполнялась проверка `sshd -t`. Итоговая проверка рабочего конфига:

```bash
sudo /usr/sbin/sshd -t; echo "код возврата: $?"
```

```text
код возврата: 0
```

Ошибок нет (при успехе `sshd -t` ничего не выводит). Для демонстрации реакции на ошибку использована копия конфига в `/tmp` (рабочий файл не затрагивался):

```bash
sudo cp /etc/openssh/sshd_config /tmp/sshd_config_test
echo "BadDirective yes" | sudo tee -a /tmp/sshd_config_test
sudo /usr/sbin/sshd -t -f /tmp/sshd_config_test; echo "код возврата: $?"
sudo rm -f /tmp/sshd_config_test
```

```text
BadDirective yes
sshd: /tmp/sshd_config_test: line 136: Bad configuration option: BadDirective
sshd: /tmp/sshd_config_test line 136: Directive 'BadDirective' is not allowed within a Match block
код возврата: 255
```

`sshd -t` указал на строку с ошибкой и вернул ненулевой код. В этом случае службу перезапускать нельзя, пока ошибка не исправлена: с неверным конфигом `sshd` может не запуститься, а при удалённой работе это приведёт к потере доступа. (Вторая строка сообщения появилась потому, что в конце файла уже есть блок `Match`.) Тестовая копия удалена.

## Итоговое состояние конфигурации

```bash
sudo diff /etc/openssh/sshd_config.bak /etc/openssh/sshd_config
sudo /usr/sbin/sshd -T | grep -Ei '^(port|permitrootlogin|maxauthtries|denyusers)'
```

```text
32c32
< #PermitRootLogin without-password
---
> PermitRootLogin no
34c34,35
< #MaxAuthTries 6
---
> MaxAuthTries 3
> DenyUsers ssh-user
port 22
maxauthtries 3
permitrootlogin no
denyusers ssh-user
```

Временное значение `PermitRootLogin yes` в итоговой конфигурации отсутствует. Резервная копия исходного файла сохранена как `/etc/openssh/sshd_config.bak`. Пользователь `ssh-user` и ограничение `DenyUsers` оставлены на сервере.
