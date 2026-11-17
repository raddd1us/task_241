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
 
## 1. Что такое shebang
 
**Shebang** (шебанг, «хэш-банг») — это первая строка скрипта, которая начинается с символов `#!` и указывает путь к интерпретатору, которым нужно выполнять файл:
 
```bash
#!/usr/bin/env bash
```
 
Когда скрипт запускается как программа (`./script`), ядро Linux читает первые байты файла. Если там `#!`, ядро запускает указанный интерпретатор и передаёт ему путь к скрипту. Без shebang запуск `./script` выполнит файл текущей оболочкой, что может дать непредсказуемый результат. Для остальных строк `#` — это обычный комментарий, поэтому shebang не мешает интерпретатору.
 
Два распространённых варианта:
 
| Запись | Особенность |
|--------|-------------|
| `#!/bin/bash` | жёстко задан путь к `bash` |
| `#!/usr/bin/env bash` | `env` ищет `bash` в `PATH`, переносимее между системами |
 
В работе используется `#!/usr/bin/env bash`.
 
## 2. Обязательно ли расширение `.sh`
 
**Нет, не обязательно.** В Linux тип файла определяется не расширением, а его содержимым и правами доступа. Чтобы файл можно было запустить как программу, нужны две вещи:
 
1. бит исполнения (`chmod +x`);
2. корректный shebang в первой строке.
Расширение `.sh` — только соглашение, которое помогает человеку и редакторам кода понять, что это shell-скрипт. Проверка: тот же скрипт, скопированный в файл без расширения, запускается так же:
 
```bash
raddd1us@ubuntu:~/scripts$ cp file_ops.sh file_ops
raddd1us@ubuntu:~/scripts$ chmod +x file_ops
raddd1us@ubuntu:~/scripts$ ls -l file_ops
-rwxr-xr-x 1 raddd1us raddd1us 5508 Nov 17 18:53 file_ops
raddd1us@ubuntu:~/scripts$ ./file_ops ~/lab
...
2026-11-17 18:53:23 [file_ops] === Завершено успешно ===
```
 
Если бит исполнения не установлен, файл всё равно можно выполнить, передав его интерпретатору явно: `bash file_ops.sh ~/lab`. В этом случае shebang игнорируется.
 
## 3. Скрипт для блока «Работа с файлами»
 
Скрипт `file_ops.sh` автоматизирует основные действия из блока «Работа с файлами»: создание каталогов и файлов, копирование, переименование, перемещение, изменение прав, просмотр результата и удаление.
 
Что делает скрипт по шагам:
 
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
#   5. выводит содержимое каталогов;
#   6. удаляет только тот временный файл, который сам создал в этом запуске.
# Существующие файлы НЕ перезаписываются: если файл уже есть, действие
# пропускается с предупреждением.
 
set -euo pipefail
 
SCRIPT_NAME=$(basename "$0")
 
usage() {
    echo "Использование: $SCRIPT_NAME <рабочий_каталог>" >&2
}
 
# --- 3.3 / 3.4: проверка аргумента --------------------------------------
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
 
# --- 3.6: логирование (в файл и на экран) -------------------------------
log() {
    printf '%s [%s] %s\n' "$(date '+%F %T')" "$SCRIPT_NAME" "$*" | tee -a "$LOG_FILE"
}
 
# --- 3.5: безопасное создание файла (без перезаписи) --------------------
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
 
Сделаем скрипт исполняемым и проверим shebang:
 
```bash
raddd1us@ubuntu:~/scripts$ chmod +x file_ops.sh
raddd1us@ubuntu:~/scripts$ head -n 1 file_ops.sh
#!/usr/bin/env bash
raddd1us@ubuntu:~/scripts$ ls -l file_ops.sh
-rwxr-xr-x 1 raddd1us raddd1us 5508 Nov 17 18:53 file_ops.sh
```
 
### 3.1–3.2. `set -euo pipefail` и назначение параметров
 
В начале скрипта (после shebang и комментариев) указана строка:
 
```bash
set -euo pipefail
```
 
| Параметр | Назначение | Пример |
|----------|------------|--------|
| `-e` (`errexit`) | Скрипт немедленно завершается, если любая команда вернула ненулевой код. Ошибка не остаётся незамеченной, и скрипт не идёт дальше с неверным состоянием. | `set -e; false; echo "не выполнится"` |
| `-u` (`nounset`) | Обращение к неопределённой переменной считается ошибкой. Это защищает от опечаток в именах переменных, например от `rm -rf "$DIR/"` при пустом `$DIR`. | `set -u; echo "$UNDEF_VAR"` → `unbound variable` |
| `-o pipefail` | Код возврата конвейера (`a \| b \| c`) становится ненулевым, если ошибкой завершилась любая команда в нём, а не только последняя. | `false \| true` → код 0, а с `pipefail` → код 1 |
 
Проверка на практике:
 
```bash
raddd1us@ubuntu:~$ bash -c 'set -e; echo до; false; echo после'; echo "rc=$?"
до
rc=1
 
raddd1us@ubuntu:~$ bash -c 'echo "$UNDEF_VAR"; echo "без -u идёт дальше"'; echo "rc=$?"
 
без -u идёт дальше
rc=0
 
raddd1us@ubuntu:~$ bash -c 'set -u; echo "$UNDEF_VAR"; echo после'; echo "rc=$?"
bash: line 1: UNDEF_VAR: unbound variable
rc=127
 
raddd1us@ubuntu:~$ bash -c 'false | true'; echo "rc=$?"
rc=0
 
raddd1us@ubuntu:~$ bash -c 'set -o pipefail; false | true'; echo "rc=$?"
rc=1
```
 
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
- `tmp.txt` удаляется только если скрипт сам создал его в этом запуске. Если такой файл уже был, он не удаляется.
### 3.6. Лог-файл
 
Функция `log` пишет каждое действие в файл `<рабочий_каталог>/file_ops.log` (с датой и временем) и одновременно выводит на экран через `tee -a`. Ключ `-a` дописывает в конец, поэтому история запусков сохраняется.
 
## 4. Запуск скрипта и коды возврата
 
### Запуск с корректным аргументом (первый запуск)
 
```bash
raddd1us@ubuntu:~/scripts$ ./file_ops.sh ~/lab
2026-11-17 18:53:17 [file_ops.sh] === Старт. Рабочий каталог: /home/raddd1us/lab ===
2026-11-17 18:53:17 [file_ops.sh] Каталоги docs, backup, archive готовы
2026-11-17 18:53:17 [file_ops.sh] Создан файл: /home/raddd1us/lab/docs/notes.txt
2026-11-17 18:53:17 [file_ops.sh] Создан файл: /home/raddd1us/lab/docs/data.csv
2026-11-17 18:53:17 [file_ops.sh] Создан файл: /home/raddd1us/lab/docs/report.txt
2026-11-17 18:53:17 [file_ops.sh] Скопирован: docs/notes.txt -> backup/notes.txt
2026-11-17 18:53:17 [file_ops.sh] Переименован: data.csv -> data_v1.csv
2026-11-17 18:53:17 [file_ops.sh] Перемещён: docs/report.txt -> archive/report.txt
2026-11-17 18:53:17 [file_ops.sh] Права docs/notes.txt: 640
2026-11-17 18:53:17 [file_ops.sh] Создан временный файл: /home/raddd1us/lab/docs/tmp.txt
2026-11-17 18:53:17 [file_ops.sh] Удалён временный файл: /home/raddd1us/lab/docs/tmp.txt
2026-11-17 18:53:17 [file_ops.sh] Итоговый список файлов:
2026-11-17 18:53:17 [file_ops.sh]   archive/report.txt
2026-11-17 18:53:17 [file_ops.sh]   backup/notes.txt
2026-11-17 18:53:17 [file_ops.sh]   docs/data_v1.csv
2026-11-17 18:53:17 [file_ops.sh]   docs/notes.txt
2026-11-17 18:53:17 [file_ops.sh] === Завершено успешно ===
raddd1us@ubuntu:~/scripts$ echo "Код возврата: $?"
Код возврата: 0
```
 
### Запуск без аргумента
 
```bash
raddd1us@ubuntu:~/scripts$ ./file_ops.sh
Ошибка: не указан рабочий каталог.
Использование: file_ops.sh <рабочий_каталог>
raddd1us@ubuntu:~/scripts$ echo "Код возврата: $?"
Код возврата: 1
```
 
Дополнительно проверена передача пути, который является файлом, а не каталогом:
 
```bash
raddd1us@ubuntu:~/scripts$ touch ~/afile
raddd1us@ubuntu:~/scripts$ ./file_ops.sh ~/afile
Ошибка: '/home/raddd1us/afile' существует, но не является каталогом.
raddd1us@ubuntu:~/scripts$ echo "Код возврата: $?"
Код возврата: 2
```
 
### Результат работы: содержимое каталога
 
```bash
raddd1us@ubuntu:~/scripts$ ls -lR ~/lab
/home/raddd1us/lab:
total 16
drwxr-xr-x 2 raddd1us raddd1us 4096 Nov 17 18:53 archive
drwxr-xr-x 2 raddd1us raddd1us 4096 Nov 17 18:53 backup
drwxr-xr-x 2 raddd1us raddd1us 4096 Nov 17 18:53 docs
-rw-r--r-- 1 raddd1us raddd1us 2763 Nov 17 18:53 file_ops.log
 
/home/raddd1us/lab/archive:
total 4
-rw-r--r-- 1 raddd1us raddd1us 11 Nov 17 18:53 report.txt
 
/home/raddd1us/lab/backup:
total 4
-rw-r--r-- 1 raddd1us raddd1us 58 Nov 17 18:53 notes.txt
 
/home/raddd1us/lab/docs:
total 8
-rw-r--r-- 1 raddd1us raddd1us 14 Nov 17 18:53 data_v1.csv
-rw-r----- 1 raddd1us raddd1us 58 Nov 17 18:53 notes.txt
```
 
### Повторный запуск
 
Существующие файлы не перезаписаны, скрипт выдаёт предупреждения и завершается успешно:
 
```bash
raddd1us@ubuntu:~/scripts$ ./file_ops.sh ~/lab
2026-11-17 18:53:23 [file_ops.sh] === Старт. Рабочий каталог: /home/raddd1us/lab ===
2026-11-17 18:53:23 [file_ops.sh] Каталоги docs, backup, archive готовы
2026-11-17 18:53:23 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: /home/raddd1us/lab/docs/notes.txt уже существует, не перезаписываю
2026-11-17 18:53:23 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: data_v1.csv уже есть, data.csv заново не создаю
2026-11-17 18:53:23 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: report.txt уже в archive, заново не создаю
2026-11-17 18:53:23 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: backup/notes.txt уже существует, копирование пропущено
2026-11-17 18:53:23 [file_ops.sh] Права docs/notes.txt: 640
2026-11-17 18:53:23 [file_ops.sh] Создан временный файл: /home/raddd1us/lab/docs/tmp.txt
2026-11-17 18:53:23 [file_ops.sh] Удалён временный файл: /home/raddd1us/lab/docs/tmp.txt
2026-11-17 18:53:23 [file_ops.sh] Итоговый список файлов:
2026-11-17 18:53:23 [file_ops.sh]   archive/report.txt
2026-11-17 18:53:23 [file_ops.sh]   backup/notes.txt
2026-11-17 18:53:23 [file_ops.sh]   docs/data_v1.csv
2026-11-17 18:53:23 [file_ops.sh]   docs/notes.txt
2026-11-17 18:53:23 [file_ops.sh] === Завершено успешно ===
raddd1us@ubuntu:~/scripts$ echo "Код возврата: $?"
Код возврата: 0
```
 
Содержимое `notes.txt` осталось прежним (файл не перезаписан):
 
```bash
raddd1us@ubuntu:~/scripts$ cat ~/lab/docs/notes.txt
Заметки по практической работе
```
 
### Фрагмент лог-файла
 
Лог накапливает записи обоих запусков:
 
```bash
raddd1us@ubuntu:~/scripts$ head -n 9 ~/lab/file_ops.log
2026-11-17 18:53:17 [file_ops.sh] === Старт. Рабочий каталог: /home/raddd1us/lab ===
2026-11-17 18:53:17 [file_ops.sh] Каталоги docs, backup, archive готовы
2026-11-17 18:53:17 [file_ops.sh] Создан файл: /home/raddd1us/lab/docs/notes.txt
2026-11-17 18:53:17 [file_ops.sh] Создан файл: /home/raddd1us/lab/docs/data.csv
2026-11-17 18:53:17 [file_ops.sh] Создан файл: /home/raddd1us/lab/docs/report.txt
2026-11-17 18:53:17 [file_ops.sh] Скопирован: docs/notes.txt -> backup/notes.txt
2026-11-17 18:53:17 [file_ops.sh] Переименован: data.csv -> data_v1.csv
2026-11-17 18:53:17 [file_ops.sh] Перемещён: docs/report.txt -> archive/report.txt
2026-11-17 18:53:17 [file_ops.sh] Права docs/notes.txt: 640
 
raddd1us@ubuntu:~/scripts$ grep ПРЕДУПРЕЖДЕНИЕ ~/lab/file_ops.log
2026-11-17 18:53:23 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: /home/raddd1us/lab/docs/notes.txt уже существует, не перезаписываю
2026-11-17 18:53:23 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: data_v1.csv уже есть, data.csv заново не создаю
2026-11-17 18:53:23 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: report.txt уже в archive, заново не создаю
2026-11-17 18:53:23 [file_ops.sh] ПРЕДУПРЕЖДЕНИЕ: backup/notes.txt уже существует, копирование пропущено
```
 
### Сводка кодов возврата
 
| Ситуация | Код возврата |
|----------|--------------|
| Успешное выполнение (первый и повторный запуск) | `0` |
| Аргумент не передан | `1` |
| Указанный путь — не каталог | `2` |
 
Код возврата последней команды хранится в переменной `$?`; `0` означает успех, любое ненулевое значение — ошибку.
