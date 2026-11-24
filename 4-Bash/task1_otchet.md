# Григорьев Сергей 241 группа
# Скриптуем по полной

1) Что такое shebang?
2) Обязательно ли исполняемый файл должен иметь расширение `.sh`?
3) Напишите скрипт, который автоматически выполняет основные действия из блока «Работа с файлами».
   3.1) Скрипт должен использовать `set -euo pipefail`.
   3.2) Объясните назначение каждого из этих параметров.
   3.3) Каталог для работы должен передаваться первым аргументом скрипта.
   3.4) Если аргумент не передан — скрипт должен завершаться с понятной ошибкой и ненулевым кодом возврата.
   3.5) Повторный запуск скрипта не должен уничтожать уже существующие данные без предупреждения.
   3.6) Основные действия скрипта должны записываться в лог-файл.
4) Покажите код возврата успешно завершившегося скрипта и скрипта, завершившегося с ошибкой.

## Критерий приёмки

В отчёте должны быть:
- полный текст созданного скрипта;
- указанный shebang;
- использование `set -euo pipefail`;
- пример запуска скрипта с корректным аргументом;
- пример запуска без аргумента и код возврата;
- пример повторного запуска;
- фрагмент созданного лог-файла;
- вывод кода возврата успешного и ошибочного завершения;
- краткое описание назначения `-e`, `-u`, `-o pipefail`.

# Отчёт по практической работе: Скриптуем по полной

## Окружение

# Проверка оболочки и путей к интерпретатору:

```bash
[student@student-13 ~]$ echo "$SHELL"
/bin/bash
[student@student-13 ~]$ bash --version | head -n 1
GNU bash, version 5.2.37(1)-release (x86_64-alt-linux-gnu)
[student@student-13 ~]$ command -v bash env
/usr/bin/bash
/usr/bin/env
[student@student-13 ~]$ mkdir -p ~/scripts && cd ~/scripts && pwd
/home/student/scripts
```

`bash` и `env` найдены, поэтому в скрипте используется `#!/usr/bin/env bash`.

## 1. Что такое shebang

**Shebang** (шебанг, «хэш-банг») — это первая строка скрипта, которая начинается с символов `#!` и указывает путь к интерпретатору, которым нужно выполнять файл:

```bash
#!/usr/bin/env bash
```

Когда скрипт запускается как программа (`./script`), ядро Linux читает первые байты файла. Если там `#!`, ядро запускает указанный интерпретатор и передаёт ему путь к скрипту. Для интерпретатора эта строка является обычным комментарием, поэтому ему она не мешает.

Два распространённых варианта:

| Запись | Особенность |
|--------|-------------|
| `#!/bin/bash` | жёстко задан путь к `bash` |
| `#!/usr/bin/env bash` | `env` ищет `bash` в `PATH`, переносимее между системами |

В работе используется `#!/usr/bin/env bash`.

## 2. Обязательно ли расширение `.sh`

**Нет, не обязательно.** Проверка на файле `hello` без расширения:

```bash
[student@student-13 scripts]$ cat > hello <<'EOF'
> #!/usr/bin/env bash
> echo "Привет из скрипта без расширения"
> echo "Версия bash: $BASH_VERSION"
> EOF
[student@student-13 scripts]$ head -n 1 hello
#!/usr/bin/env bash
[student@student-13 scripts]$ ls -l hello
-rw-r--r-- 1 student student 128 Nov 24 19:42 hello
[student@student-13 scripts]$ ./hello; echo "rc=$?"
-bash: ./hello: Permission denied
rc=126
[student@student-13 scripts]$ bash hello; echo "rc=$?"
Привет из скрипта без расширения
Версия bash: 5.2.37(1)-release
rc=0
[student@student-13 scripts]$ chmod +x hello
[student@student-13 scripts]$ ls -l hello
-rwxr-xr-x 1 student student 128 Nov 24 19:42 hello
[student@student-13 scripts]$ ./hello; echo "rc=$?"
Привет из скрипта без расширения
Версия bash: 5.2.37(1)-release
rc=0
```

Результат:
- без бита исполнения `./hello` не запустился (`Permission denied`, код `126`);
- `bash hello` отработал, так как интерпретатор указан явно и бит исполнения не нужен;
- после `chmod +x` файл без расширения запускается как обычная программа.

Чтобы файл можно было запускать как программу, нужны бит исполнения и корректный shebang. Расширение `.sh` — только соглашение, которое помогает человеку и редакторам кода.

## 3. Скрипт для блока «Работа с файлами»

Скрипт `file_ops.sh` автоматизирует основные действия блока «Работа с файлами»: создание каталогов и файлов, копирование, переименование, перемещение, изменение прав, удаление временного файла и просмотр результата.

| Шаг | Действие | Команда |
|-----|----------|---------|
| 1 | Создаёт каталоги `docs`, `backup`, `archive` | `mkdir -p` |
| 2 | Создаёт тестовые файлы `notes.txt`, `data.csv`, `report.txt` | `printf >` |
| 3 | Копирует `notes.txt` в `backup` | `cp` |
| 4 | Переименовывает `data.csv` в `data_v1.csv` | `mv` |
| 5 | Перемещает `report.txt` в `archive` | `mv` |
| 6 | Меняет права на `notes.txt` (640) | `chmod` |
| 7 | Создаёт и удаляет временный файл `tmp.txt` | `rm` |
| 8 | Выводит итоговый список файлов | `find`, `sort` |

### Полный текст скрипта

```bash
#!/usr/bin/env bash
#
# file_ops.sh - автоматизация основных действий блока «Работа с файлами».
# Использование: ./file_ops.sh <рабочий_каталог>
#
# Что делает скрипт:
#   1. создаёт структуру каталогов (docs, backup, archive);
#   2. создаёт тестовые файлы;
#   3. копирует, переименовывает и перемещает файлы;
#   4. меняет права доступа;
#   5. удаляет только тот временный файл, который сам создал в этом запуске;
#   6. выводит итоговый список файлов.
# Существующие файлы НЕ перезаписываются: если файл уже есть, действие
# пропускается с предупреждением.

set -euo pipefail

SCRIPT_NAME=$(basename "$0")

usage() {
    echo "Использование: $SCRIPT_NAME <рабочий_каталог>" >&2
}

# --- проверка аргумента -------------------------------------------------
if [ $# -lt 1 ] || [ -z "$1" ]; then
    echo "Ошибка: не указан рабочий каталог." >&2
    usage
    exit 1
fi

WORKDIR=$1

if [ -e "$WORKDIR" ] && [ ! -d "$WORKDIR" ]; then
    echo "Ошибка: '$WORKDIR' существует, но не является каталогом." >&2
    exit 2
fi

mkdir -p "$WORKDIR"
LOG_FILE="$WORKDIR/file_ops.log"

# --- логирование (в файл и на экран) ------------------------------------
log() {
    printf '%s [%s] %s\n' "$(date '+%F %T')" "$SCRIPT_NAME" "$*" | tee -a "$LOG_FILE"
}

# --- безопасное создание файла (без перезаписи) -------------------------
create_file() {
    local path=$1 content=$2
    if [ -e "$path" ]; then
        log "ПРЕДУПРЕЖДЕНИЕ: $path уже существует, не перезаписываю"
    else
        printf '%s\n' "$content" > "$path"
        log "Создан файл: $path"
    fi
}

log "=== Старт. Рабочий каталог: $WORKDIR ==="

# 1. Каталоги (mkdir -p ничего не ломает, если каталог уже есть)
mkdir -p "$WORKDIR/docs" "$WORKDIR/backup" "$WORKDIR/archive"
log "Каталоги docs, backup, archive готовы"

# 2. Тестовые файлы
create_file "$WORKDIR/docs/notes.txt" "Заметки по практической работе"
# data.csv и report.txt позже переименовываются/переносятся,
# поэтому перед созданием проверяем и новое место
if [ -e "$WORKDIR/docs/data_v1.csv" ]; then
    log "ПРЕДУПРЕЖДЕНИЕ: data_v1.csv уже есть, data.csv заново не создаю"
else
    create_file "$WORKDIR/docs/data.csv" "id,name,value"
fi
if [ -e "$WORKDIR/archive/report.txt" ]; then
    log "ПРЕДУПРЕЖДЕНИЕ: report.txt уже в archive, заново не создаю"
else
    create_file "$WORKDIR/docs/report.txt" "Отчёт"
fi

# 3. Копирование (в backup)
if [ -e "$WORKDIR/docs/notes.txt" ]; then
    if [ -e "$WORKDIR/backup/notes.txt" ]; then
        log "ПРЕДУПРЕЖДЕНИЕ: backup/notes.txt уже существует, копирование пропущено"
    else
        cp "$WORKDIR/docs/notes.txt" "$WORKDIR/backup/notes.txt"
        log "Скопирован: docs/notes.txt -> backup/notes.txt"
    fi
fi

# 4. Переименование (data.csv -> data_v1.csv)
if [ -e "$WORKDIR/docs/data.csv" ]; then
    if [ -e "$WORKDIR/docs/data_v1.csv" ]; then
        log "ПРЕДУПРЕЖДЕНИЕ: data_v1.csv уже существует, переименование пропущено"
    else
        mv "$WORKDIR/docs/data.csv" "$WORKDIR/docs/data_v1.csv"
        log "Переименован: data.csv -> data_v1.csv"
    fi
fi

# 5. Перемещение (report.txt -> archive)
if [ -e "$WORKDIR/docs/report.txt" ]; then
    if [ -e "$WORKDIR/archive/report.txt" ]; then
        log "ПРЕДУПРЕЖДЕНИЕ: archive/report.txt уже существует, перемещение пропущено"
    else
        mv "$WORKDIR/docs/report.txt" "$WORKDIR/archive/report.txt"
        log "Перемещён: docs/report.txt -> archive/report.txt"
    fi
fi

# 6. Права доступа (операция идемпотентная и данные не уничтожает)
if [ -e "$WORKDIR/docs/notes.txt" ]; then
    chmod 640 "$WORKDIR/docs/notes.txt"
    log "Права docs/notes.txt: 640"
fi

# 7. Временный файл: создаём и удаляем только если создали сами
TMP_FILE="$WORKDIR/docs/tmp.txt"
if [ -e "$TMP_FILE" ]; then
    log "ПРЕДУПРЕЖДЕНИЕ: $TMP_FILE уже существует (не наш файл), не удаляю"
else
    : > "$TMP_FILE"
    log "Создан временный файл: $TMP_FILE"
    rm "$TMP_FILE"
    log "Удалён временный файл: $TMP_FILE"
fi

# 8. Итоговое содержимое (pipefail: сбой find не останется незамеченным)
log "Итоговый список файлов:"
find "$WORKDIR" -type f ! -name 'file_ops.log' | sort | while read -r f; do
    log "  ${f#"$WORKDIR"/}"
done

log "=== Завершено успешно ==="
```

Файл сделан исполняемым, проверены shebang, права и синтаксис (`bash -n` только проверяет синтаксис, скрипт не выполняет):

```bash
[student@student-13 scripts]$ chmod +x ~/scripts/file_ops.sh
[student@student-13 scripts]$ head -n 1 ~/scripts/file_ops.sh
#!/usr/bin/env bash
[student@student-13 scripts]$ ls -l ~/scripts/file_ops.sh
-rwxr-xr-x 1 student student 5511 Nov 24 19:44 /home/student/scripts/file_ops.sh
[student@student-13 scripts]$ bash -n ~/scripts/file_ops.sh && echo "Синтаксис OK"
Синтаксис OK
```

### 3.1–3.2. `set -euo pipefail` и назначение параметров

В начале скрипта (после shebang и комментариев) указана строка:

```bash
set -euo pipefail
```

| Параметр | Назначение |
|----------|------------|
| `-e` (`errexit`) | Скрипт немедленно завершается, если любая команда вернула ненулевой код. Ошибка не остаётся незамеченной, и скрипт не идёт дальше с неверным состоянием. |
| `-u` (`nounset`) | Обращение к неопределённой переменной считается ошибкой. Это защищает от опечаток в именах переменных. |
| `-o pipefail` | Код возврата конвейера (`a \| b \| c`) становится ненулевым, если ошибкой завершилась любая команда в нём, а не только последняя. |

Проверка каждого параметра на практике:

```bash
[student@student-13 scripts]$ bash -c 'echo до; false; echo после'; echo "rc=$?"
до
после
rc=0
[student@student-13 scripts]$ bash -c 'set -e; echo до; false; echo после'; echo "rc=$?"
до
rc=1
[student@student-13 scripts]$ bash -c 'echo "[$UNDEF_VAR]"; echo "без -u идёт дальше"'; echo "rc=$?"
[]
без -u идёт дальше
rc=0
[student@student-13 scripts]$ bash -c 'set -u; echo "[$UNDEF_VAR]"; echo после'; echo "rc=$?"
bash: line 1: UNDEF_VAR: unbound variable
rc=127
[student@student-13 scripts]$ bash -c 'false | true'; echo "rc=$?"
rc=0
[student@student-13 scripts]$ bash -c 'set -o pipefail; false | true'; echo "rc=$?"
rc=1
```

Результат:
- без `-e` команда `false` не останавливает скрипт (напечатано `после`), с `-e` скрипт остановился на ней (`rc=1`);
- без `-u` неопределённая переменная даёт пустую строку, с `-u` скрипт остановлен сообщением `unbound variable` (в bash 5.2.37 на этом сервере код возврата `127`);
- без `pipefail` ошибка `false` в конвейере `false | true` скрыта (`rc=0`), с `pipefail` она видна (`rc=1`).

В скрипте это важно, например, в конвейере `find ... | sort | while read ...`: если `find` завершится с ошибкой, благодаря `pipefail` скрипт не сообщит об успехе.

### 3.3–3.4. Каталог первым аргументом и обработка ошибки

Каталог берётся из `$1`. Проверка количества аргументов выполняется до обращения к `$1`, иначе `set -u` завершил бы скрипт с малопонятным сообщением. Если аргумента нет или он пустой, скрипт печатает понятную ошибку и подсказку в `stderr` и завершается с кодом `1`. Если указанный путь существует, но это не каталог, возвращается код `2`.

```bash
if [ $# -lt 1 ] || [ -z "$1" ]; then
    echo "Ошибка: не указан рабочий каталог." >&2
    usage
    exit 1
fi
```

### 3.5. Защита существующих данных

Скрипт ничего не перезаписывает и не удаляет без предупреждения:

- `mkdir -p` не ругается и не трогает уже существующий каталог;
- файлы создаются функцией `create_file`, которая при существующем файле пишет `ПРЕДУПРЕЖДЕНИЕ` и пропускает действие;
- перед `cp` и `mv` проверяется, что целевого файла ещё нет;
- файлы, которые при первом запуске были переименованы или перемещены (`data_v1.csv`, `archive/report.txt`), при повторном запуске не создаются заново;
- `tmp.txt` удаляется только если скрипт сам создал его в этом запуске.

### 3.6. Лог-файл

Функция `log` пишет каждое действие в файл `<рабочий_каталог>/file_ops.log` (с датой и временем) и одновременно выводит на экран через `tee -a`. Ключ `-a` дописывает в конец, поэтому история запусков сохраняется.

## 4. Запуск скрипта и коды возврата

Перед запуском проверено, что каталога `~/lab` ещё нет (`ls: cannot access '/home/student/lab': No such file or directory`).

### Запуск с корректным аргументом (первый запуск)

```bash
[student@student-13 scripts]$ ./file_ops.sh ~/lab
2026-11-24 19:46:10 [file_ops.sh] === Старт. Рабочий каталог: /home/student/lab ===
2026-11-24 19:46:10 [file_ops.sh] Каталоги docs, backup, archive готовы
2026-11-24 19:46:10 [file_ops.sh] Создан файл: /home/student/lab/docs/notes.txt
2026-11-24 19:46:10 [file_ops.sh] Создан файл: /home/student/lab/docs/data.csv
2026-11-24 19:46:10 [file_ops.sh] Создан файл: /home/student/lab/docs/report.txt
2026-11-24 19:46:10 [file_ops.sh] Скопирован: docs/notes.txt -> backup/notes.txt
2026-11-24 19:46:10 [file_ops.sh] Переименован: data.csv -> data_v1.csv
2026-11-24 19:46:10 [file_ops.sh] Перемещён: docs/report.txt -> archive/report.txt
2026-11-24 19:46:10 [file_ops.sh] Права docs/notes.txt: 640
2026-11-24 19:46:10 [file_ops.sh] Создан временный файл: /home/student/lab/docs/tmp.txt
2026-11-24 19:46:10 [file_ops.sh] Удалён временный файл: /home/student/lab/docs/tmp.txt
2026-11-24 19:46:10 [file_ops.sh] Итоговый список файлов:
2026-11-24 19:46:10 [file_ops.sh]   archive/report.txt
2026-11-24 19:46:10 [file_ops.sh]   backup/notes.txt
2026-11-24 19:46:10 [file_ops.sh]   docs/data_v1.csv
2026-11-24 19:46:10 [file_ops.sh]   docs/notes.txt
2026-11-24 19:46:10 [file_ops.sh] === Завершено успешно ===
[student@student-13 scripts]$ echo "Код возврата: $?"
Код возврата: 0
```

Результат работы: содержимое каталога

```bash
[student@student-13 scripts]$ ls -lR ~/lab
/home/student/lab:
total 16
drwxr-xr-x 2 student student 4096 Nov 24 19:46 archive
drwxr-xr-x 2 student student 4096 Nov 24 19:46 backup
drwxr-xr-x 2 student student 4096 Nov 24 19:46 docs
-rw-r--r-- 1 student student 1371 Nov 24 19:46 file_ops.log

/home/student/lab/archive:
total 4
-rw-r--r-- 1 student student 11 Nov 24 19:46 report.txt

/home/student/lab/backup:
total 4
-rw-r--r-- 1 student student 58 Nov 24 19:46 notes.txt

/home/student/lab/docs:
total 8
-rw-r--r-- 1 student student 14 Nov 24 19:46 data_v1.csv
-rw-r----- 1 student student 58 Nov 24 19:46 notes.txt
```

Файл `notes.txt` получил права 640 (`-rw-r-----`), `data.csv` переименован в `data_v1.csv`, `report.txt` перемещён в `archive`, временный файл создан и удалён.

### Запуск без аргумента и с некорректными аргументами

```bash
[student@student-13 scripts]$ ./file_ops.sh; echo "Код возврата: $?"
Ошибка: не указан рабочий каталог.
Использование: file_ops.sh <рабочий_каталог>
Код возврата: 1
[student@student-13 scripts]$ ./file_ops.sh ""; echo "Код возврата: $?"
Ошибка: не указан рабочий каталог.
Использование: file_ops.sh <рабочий_каталог>
Код возврата: 1
[student@student-13 scripts]$ ./file_ops.sh ~/scripts/hello; echo "Код возврата: $?"
Ошибка: '/home/student/scripts/hello' существует, но не является каталогом.
Код возврата: 2
```

Без аргумента и с пустым аргументом скрипт выдаёт понятную ошибку и код `1`. Если передан путь к файлу, а не к каталогу, возвращается код `2`.

### Повторный запуск

Чтобы доказать, что существующие данные не уничтожаются, в `notes.txt` перед запуском дописана строка-маркер:

```bash
[student@student-13 scripts]$ echo "МАРКЕР: не затирать" >> ~/lab/docs/notes.txt
[student@student-13 scripts]$ cat ~/lab/docs/notes.txt
Заметки по практической работе
МАРКЕР: не затирать
[student@student-13 scripts]$ ./file_ops.sh ~/lab; echo "Код возврата: $?"
2026-11-24 19:47:46 [file_ops.sh] === Старт. Рабочий каталог: /home/student/lab ===
2026-11-24 19:47:46 [file_ops.sh] Каталоги docs, backup, archive готовы
2026-11-24 19:47:46 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: /home/student/lab/docs/notes.txt уже существует, не перезаписываю
2026-11-24 19:47:46 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: data_v1.csv уже есть, data.csv заново не создаю
2026-11-24 19:47:46 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: report.txt уже в archive, заново не создаю
2026-11-24 19:47:46 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: backup/notes.txt уже существует, копирование пропущено
2026-11-24 19:47:46 [file_ops.sh] Права docs/notes.txt: 640
2026-11-24 19:47:46 [file_ops.sh] Создан временный файл: /home/student/lab/docs/tmp.txt
2026-11-24 19:47:46 [file_ops.sh] Удалён временный файл: /home/student/lab/docs/tmp.txt
2026-11-24 19:47:46 [file_ops.sh] Итоговый список файлов:
2026-11-24 19:47:46 [file_ops.sh]   archive/report.txt
2026-11-24 19:47:46 [file_ops.sh]   backup/notes.txt
2026-11-24 19:47:46 [file_ops.sh]   docs/data_v1.csv
2026-11-24 19:47:46 [file_ops.sh]   docs/notes.txt
2026-11-24 19:47:46 [file_ops.sh] === Завершено успешно ===
Код возврата: 0
[student@student-13 scripts]$ cat ~/lab/docs/notes.txt
Заметки по практической работе
МАРКЕР: не затирать
```

Существующие файлы не перезаписаны: скрипт выдал предупреждения, строка-маркер в `notes.txt` осталась на месте, код возврата `0`.

### Фрагмент лог-файла

Лог накапливает записи обоих запусков:

```bash
[student@student-13 scripts]$ wc -l ~/lab/file_ops.log
32 /home/student/lab/file_ops.log
[student@student-13 scripts]$ head -n 9 ~/lab/file_ops.log
2026-11-24 19:46:10 [file_ops.sh] === Старт. Рабочий каталог: /home/student/lab ===
2026-11-24 19:46:10 [file_ops.sh] Каталоги docs, backup, archive готовы
2026-11-24 19:46:10 [file_ops.sh] Создан файл: /home/student/lab/docs/notes.txt
2026-11-24 19:46:10 [file_ops.sh] Создан файл: /home/student/lab/docs/data.csv
2026-11-24 19:46:10 [file_ops.sh] Создан файл: /home/student/lab/docs/report.txt
2026-11-24 19:46:10 [file_ops.sh] Скопирован: docs/notes.txt -> backup/notes.txt
2026-11-24 19:46:10 [file_ops.sh] Переименован: data.csv -> data_v1.csv
2026-11-24 19:46:10 [file_ops.sh] Перемещён: docs/report.txt -> archive/report.txt
2026-11-24 19:46:10 [file_ops.sh] Права docs/notes.txt: 640
[student@student-13 scripts]$ grep ПРЕДУПРЕЖДЕНИЕ ~/lab/file_ops.log
2026-11-24 19:47:46 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: /home/student/lab/docs/notes.txt уже существует, не перезаписываю
2026-11-24 19:47:46 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: data_v1.csv уже есть, data.csv заново не создаю
2026-11-24 19:47:46 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: report.txt уже в archive, заново не создаю
2026-11-24 19:47:46 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: backup/notes.txt уже существует, копирование пропущено
```

В логе 32 строки: записи первого запуска (19:46:10) и второго (19:47:46). Предупреждения повторного запуска записаны в лог.

### Сводка кодов возврата

| Ситуация | Код возврата |
|----------|--------------|
| Успешное выполнение (первый и повторный запуск) | `0` |
| Аргумент не передан или пустой | `1` |
| Указанный путь — не каталог | `2` |

Код возврата последней команды хранится в переменной `$?`; `0` означает успех, любое ненулевое значение — ошибку.
