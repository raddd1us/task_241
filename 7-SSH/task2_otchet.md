# Конфигурируем для удобства

1) Где хранятся пользовательские и системные настройки SSH-клиента?
2) Создайте пользовательский файл `~/.ssh/config`, если его ещё нет.
3) Добавьте в него отдельный `Host`, чтобы к вашему серверу можно было подключаться без явного указания имени пользователя, адреса и порта в командной строке.
4) Назовите подключение удобным псевдонимом.
5) Проверьте работоспособность подключения командой вида `ssh <ваш-псевдоним>`.
6) Если вы подключаетесь через jump host, добавьте отдельный Host для jump-сервера и настройте `ProxyJump`.
7) Покажите итоговую конфигурацию без паролей и приватных ключей.

## Критерий приёмки

В отчёте должны быть:
- путь к пользовательскому SSH-конфигу;
- содержимое созданного блока `Host`;
- использованные параметры `Host`, `HostName`, `User`, `Port`;
- команда подключения по короткому псевдониму;
- при использовании jump host — соответствующие блоки `Host` и параметр `ProxyJump`;
- подтверждение успешного подключения по настроенному псевдониму.


## Отчёт по практической работе: Конфигурируем для удобства


## 1. Где хранятся настройки SSH-клиента

Версия клиента:

```powershell
PS C:\Windows\System32> ssh -V
OpenSSH_for_Windows_9.5p2, LibreSSL 3.8.2
```

Содержимое пользовательского каталога `.ssh`:

```powershell
PS C:\Windows\System32> dir $HOME\.ssh


    Каталог: C:\Users\vozre\.ssh


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        08.10.2026     11:32            464 id_ed25519
-a----        08.10.2026     11:32            103 id_ed25519.pub
-a----        22.11.2026     20:11           1869 known_hosts
-a----        22.11.2026     20:08           1121 known_hosts.old
```

Проверка системного файла настроек клиента:

```powershell
PS C:\Windows\System32> dir C:\ProgramData\ssh\ssh_config
dir : Не удается найти путь "C:\ProgramData\ssh\ssh_config", так как он не существует.
строка:1 знак:1
+ dir C:\ProgramData\ssh\ssh_config
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\ProgramData\ssh\ssh_config:String) [Get-ChildItem], ItemNotFoundExce
   ption
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetChildItemCommand
```

Проверка каталога `C:\ProgramData\ssh`:

```powershell
PS C:\Windows\System32> dir C:\ProgramData\ssh
PS C:\Windows\System32>
```

**Пояснение:**
- **Пользовательские** настройки SSH-клиента хранятся в `~/.ssh/config`. В Windows `~` — это `C:\Users\vozre`, поэтому путь `C:\Users\vozre\.ssh\config`. В этом же каталоге лежат ключи (`id_ed25519`, `id_ed25519.pub`) и список известных серверов (`known_hosts`).
- **Системные** настройки клиента в OpenSSH для Windows находятся в `C:\ProgramData\ssh\ssh_config` (в Linux и macOS для этого используется `/etc/ssh/ssh_config`). На этой машине каталог `C:\ProgramData\ssh` пуст, файла `ssh_config` нет, то есть системных настроек клиента не задано.
- Порядок применения: параметры из командной строки главнее пользовательского `~/.ssh/config`, а он главнее системного файла.

## 2. Создание пользовательского файла `~/.ssh/config`

Файла `config` в каталоге `.ssh` не было, поэтому создаём пустой файл (команда создаёт его только если он отсутствует):

```powershell
PS C:\Windows\System32> if (-not (Test-Path $HOME\.ssh\config)) { New-Item -ItemType File -Path $HOME\.ssh\config }


    Каталог: C:\Users\vozre\.ssh


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        26.11.2026     13:44              0 config


PS C:\Windows\System32> dir $HOME\.ssh\config


    Каталог: C:\Users\vozre\.ssh


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        26.11.2026     13:44              0 config
```

Файл `C:\Users\vozre\.ssh\config` создан, размер 0 байт.

## 3–4, 6. Блоки `Host`, псевдоним и `ProxyJump`

До настройки подключение выполнялось командой, в которой нужно явно указывать пользователей, адреса и порт:

```powershell
ssh -J student13@ternar.io:8053 student@192.168.6.113
```

В `config` добавлены два блока:
- `jump13` — псевдоним jump-хоста;
- `lab13` — псевдоним сервера, который через `ProxyJump` использует `jump13`.

Запись конфигурации (кодировка `ascii`, чтобы PowerShell 5.1 не добавил BOM в начало файла):

```powershell
PS C:\Windows\System32> @'
>> Host jump13
>>     HostName ternar.io
>>     User student13
>>     Port 8053
>>
>> Host lab13
>>     HostName 192.168.6.113
>>     User student
>>     Port 22
>>     ProxyJump jump13
>> '@ | Set-Content -Path $HOME\.ssh\config -Encoding ascii
```

Проверка того, какие параметры ssh применит к псевдониму `lab13` (подключение при этом не выполняется):

```powershell
PS C:\Windows\System32> ssh -G lab13 | Select-String "^(hostname|user|port|proxyjump) "

user student
hostname 192.168.6.113
port 22
proxyjump jump13
```

**Использованные параметры:**

| Параметр | Значение в `jump13` | Значение в `lab13` | Назначение |
|----------|---------------------|--------------------|------------|
| `Host` | `jump13` | `lab13` | псевдоним, который указывается в команде `ssh` |
| `HostName` | `ternar.io` | `192.168.6.113` | реальный адрес или имя хоста |
| `User` | `student13` | `student` | имя пользователя для входа |
| `Port` | `8053` | `22` | порт SSH-сервера |
| `ProxyJump` | — | `jump13` | подключаться через jump-хост, описанный блоком `jump13` |

**Пояснение:** благодаря блоку `lab13` не нужно указывать пользователя, адрес и порт в командной строке. Параметр `ProxyJump jump13` заменяет ключ `-J student13@ternar.io:8053` из прежней команды: ssh сам находит блок `jump13` и берёт оттуда адрес, пользователя и порт jump-хоста.

## 5. Проверка подключения по псевдониму

```powershell
PS C:\Windows\System32> ssh lab13
student13@ternar.io's password:
student@192.168.6.113's password:
Last login: Thu Nov 26 09:06:50 2026 from 192.168.6.83
[student@student-13 ~]$ hostname
student-13
[student@student-13 ~]$ whoami
student
[student@student-13 ~]$ exit
logout
Connection to 192.168.6.113 closed.
PS C:\Windows\System32>
```

**Пояснение:** команда `ssh lab13` подключилась к серверу `student-13` под пользователем `student` через jump-хост. Первый запрос пароля относится к jump-хосту (`student13@ternar.io`), второй — к самому серверу (`student@192.168.6.113`). Вход выполнен успешно, `hostname` вернул `student-13`, `whoami` вернул `student`.

## 7. Итоговая конфигурация

Содержимое пользовательского SSH-конфига (паролей и приватных ключей в нём нет, только параметры подключения):

```powershell
PS C:\Windows\System32> Get-Content $HOME\.ssh\config
Host jump13
    HostName ternar.io
    User student13
    Port 8053

Host lab13
    HostName 192.168.6.113
    User student
    Port 22
    ProxyJump jump13
```

Путь к файлу: `C:\Users\vozre\.ssh\config`.
