# Григорьев Сергей 241 группа
# Перенаправляем

1) Как работают операторы `>` и `>>`?
2) Что такое стандартные потоки `stdin`, `stdout`, `stderr`?
3) Вывести содержимое файла, не используя текстовые редакторы.
4) Создать файл с содержимым, не используя текстовый редактор.
5) На примере подходящих команд:
   5.1) перенаправить только `stdout` в файл;
   5.2) перенаправить только `stderr` в файл;
   5.3) перенаправить `stdout` и `stderr` в один файл;
   5.4) перенаправить `stdout` и `stderr` в разные файлы.
6) Чем отличаются `stdout` и `stderr`?
7) Что такое `stdin`? Покажите пример передачи данных команде через стандартный ввод.
8) Как отправить весь ненужный вывод команды в `/dev/null`?
9) Чем отличается конвейер `|` от перенаправления `>`?
10) Соберите команду из двух или трёх программ через `|`, чтобы результат одной команды обрабатывался следующей.

## Критерий приёмки

В отчёте должны быть:
- примеры использования `>` и `>>`;
- пример перенаправления только `stdout`;
- пример перенаправления только `stderr`;
- пример объединения `stdout` и `stderr`;
- пример раздельного перенаправления `stdout` и `stderr`;
- пример передачи данных через `stdin`;
- пример использования `/dev/null`;
- пример конвейера из двух или более команд;
- краткое описание назначения `stdin`, `stdout`, `stderr` и файловых дескрипторов 0, 1, 2.

# Отчет по практической работе: Перенаправление ввода-вывода и конвейеры


## 1. Операторы `>` и `>>`

- `>` перенаправляет вывод команды в файл. Если файл существует, он перезаписывается, если нет, создаётся.
- `>>` дописывает вывод в конец файла, сохраняя прежнее содержимое.

Использованные команды:

```bash
mkdir -p ~/redirect_lab && cd ~/redirect_lab
pwd
echo "Первая строка" > output.txt
cat output.txt
echo "Вторая строка" >> output.txt
cat output.txt
```

Полученный результат:

```text
[student@student-13 ~]$ mkdir -p ~/redirect_lab && cd ~/redirect_lab
[student@student-13 redirect_lab]$ pwd
/home/student/redirect_lab
[student@student-13 redirect_lab]$ echo "Первая строка" > output.txt
[student@student-13 redirect_lab]$ cat output.txt
Первая строка
[student@student-13 redirect_lab]$ echo "Вторая строка" >> output.txt
[student@student-13 redirect_lab]$ cat output.txt
Первая строка
Вторая строка
```

`>` создал файл с одной строкой, `>>` добавил вторую строку, первая сохранилась.

## 2. Стандартные потоки `stdin`, `stdout`, `stderr`

- `stdin` (дескриптор 0) — стандартный поток ввода: данные, которые программа получает на обработку (по умолчанию с клавиатуры).
- `stdout` (дескриптор 1) — стандартный поток вывода: штатный результат работы программы (по умолчанию на экран терминала).
- `stderr` (дескриптор 2) — стандартный поток ошибок: сообщения об ошибках и диагностика (по умолчанию тоже на экран терминала).

## 3. Вывод содержимого файла без текстовых редакторов

Использованная команда: `cat output.txt` (результат см. в пункте 1: команда вывела обе строки файла).

## 4. Создание файла с содержимым без текстового редактора

Использованные команды:

```bash
echo "Пример создания файла без редактора" > new_file.txt
cat new_file.txt
cat > multi.txt <<'EOF'
строка один
строка два
строка три
EOF
cat multi.txt
```

Полученный результат:

```text
[student@student-13 redirect_lab]$ echo "Пример создания файла без редактора" > new_file.txt
[student@student-13 redirect_lab]$ cat new_file.txt
Пример создания файла без редактора
[student@student-13 redirect_lab]$ cat > multi.txt <<'EOF'
> строка один
> строка два
> строка три
> EOF
[student@student-13 redirect_lab]$ cat multi.txt
строка один
строка два
строка три
```

Однострочный файл создан через `echo` и `>`, многострочный через heredoc (`<<'EOF'`).

## 5. Перенаправление потоков

### 5.1. Только `stdout`

```bash
ls -l output.txt 1> stdout_only.txt
cat stdout_only.txt
```

```text
[student@student-13 redirect_lab]$ ls -l output.txt 1> stdout_only.txt
[student@student-13 redirect_lab]$ cat stdout_only.txt
-rw-r--r-- 1 student student 52 Nov 22 20:18 output.txt
```

На экран ничего не вышло, весь `stdout` попал в файл.

### 5.2. Только `stderr`

```bash
ls nonexistent_file.txt 2> stderr_only.txt
cat stderr_only.txt
```

```text
[student@student-13 redirect_lab]$ ls nonexistent_file.txt 2> stderr_only.txt
[student@student-13 redirect_lab]$ cat stderr_only.txt
ls: cannot access 'nonexistent_file.txt': No such file or directory
```

Сообщение об ошибке записано в файл через `2>`, на экране пусто.

### 5.3. `stdout` и `stderr` в один файл

```bash
ls -l output.txt nonexistent_file.txt &> combined.txt
cat combined.txt
```

```text
[student@student-13 redirect_lab]$ ls -l output.txt nonexistent_file.txt &> combined.txt
[student@student-13 redirect_lab]$ cat combined.txt
ls: cannot access 'nonexistent_file.txt': No such file or directory
-rw-r--r-- 1 student student 52 Nov 22 20:18 output.txt
```

Оператор `&>` собрал оба потока в один файл. Ошибка стоит выше результата, потому что `ls` сначала проверяет аргументы и сообщает об ошибке, а потом печатает список.

### 5.4. `stdout` и `stderr` в разные файлы

```bash
ls -l output.txt nonexistent_file.txt 1> success.log 2> error.log
cat success.log
cat error.log
```

```text
[student@student-13 redirect_lab]$ ls -l output.txt nonexistent_file.txt 1> success.log 2> error.log
[student@student-13 redirect_lab]$ cat success.log
-rw-r--r-- 1 student student 52 Nov 22 20:18 output.txt
[student@student-13 redirect_lab]$ cat error.log
ls: cannot access 'nonexistent_file.txt': No such file or directory
```

Результат попал в `success.log`, ошибка в `error.log`.

## 6. Чем отличаются `stdout` и `stderr`

`stdout` предназначен для полезного результата команды, `stderr` для сообщений об ошибках и диагностики. Это два независимых потока: как показано в пункте 5, их можно направить в разные файлы. Благодаря этому результат можно обрабатывать дальше, а ошибки сохранять отдельно или скрывать.

## 7. `stdin` и передача данных через стандартный ввод

`stdin` (дескриптор 0) — поток, из которого программа читает входные данные. Оператор `<` подаёт на него содержимое файла.

```bash
grep "строка" < output.txt
wc -l < output.txt
```

```text
[student@student-13 redirect_lab]$ grep "строка" < output.txt
Первая строка
Вторая строка
[student@student-13 redirect_lab]$ wc -l < output.txt
2
```

`grep` и `wc` прочитали данные не из аргумента, а из `stdin`.

## 8. Отправка ненужного вывода в `/dev/null`

`/dev/null` — специальный файл, который отбрасывает всё, что в него записано.

```bash
ls nonexistent_file.txt &> /dev/null
echo $?
ls -l output.txt nonexistent_file.txt 2> /dev/null
```

```text
[student@student-13 redirect_lab]$ ls nonexistent_file.txt &> /dev/null
[student@student-13 redirect_lab]$ echo $?
2
[student@student-13 redirect_lab]$ ls -l output.txt nonexistent_file.txt 2> /dev/null
-rw-r--r-- 1 student student 52 Nov 22 20:18 output.txt
```

`&> /dev/null` скрыл и результат, и ошибку. Код возврата `2` показывает, что команда всё равно завершилась с ошибкой. `2> /dev/null` скрыл только ошибку, результат остался на экране.

## 9. Чем отличается конвейер `|` от перенаправления `>`

- Конвейер `|` передаёт `stdout` одной команды на `stdin` следующей, промежуточный файл не нужен.
- Перенаправление `>` сохраняет вывод команды в файл на диске.

Пример для сравнения:

```bash
ls | sort -r
ls > list.txt
cat list.txt
```

```text
[student@student-13 redirect_lab]$ ls | sort -r
success.log
stdout_only.txt
stderr_only.txt
output.txt
new_file.txt
multi.txt
error.log
combined.txt
[student@student-13 redirect_lab]$ ls > list.txt
[student@student-13 redirect_lab]$ cat list.txt
combined.txt
error.log
list.txt
multi.txt
new_file.txt
output.txt
stderr_only.txt
stdout_only.txt
success.log
```

В первом случае вывод `ls` обработал `sort -r` и результат сразу попал на экран. Во втором вывод сохранён в `list.txt`. Сам `list.txt` попал в список, потому что оболочка создаёт файл до запуска `ls`.

## 10. Конвейер из нескольких программ

```bash
cat output.txt multi.txt | grep "строка" | wc -l
```

```text
[student@student-13 redirect_lab]$ cat output.txt multi.txt | grep "строка" | wc -l
5
```

`cat` вывел содержимое двух файлов (5 строк), `grep` оставил строки со словом «строка», `wc -l` их посчитал. Результат: 5.

## Удаление рабочего каталога

```bash
cd ~ && command rm -r ~/redirect_lab && ls -d ~/redirect_lab
```

```text
[student@student-13 ~]$ cd ~ && command rm -r ~/redirect_lab && ls -d ~/redirect_lab
ls: cannot access '/home/student/redirect_lab': No such file or directory
```

Рабочий каталог `redirect_lab` со всеми созданными файлами удалён. Команда `command rm` вызывает `rm` в обход псевдонима: на сервере `rm` запрашивает подтверждение для каждого файла.















