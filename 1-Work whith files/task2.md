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

## 1. Операторы перенаправления `>` и `>>`
- **`>` (перезапись)** — перенаправляет поток вывода в файл. Если файл существовал, его содержимое полностью перезаписывается. Если файла не было, он создаётся.
- **`>>` (добавление)** — перенаправляет поток вывода в конец файла. Существующие данные сохраняются, новые строки дописываются в конец.

**Пример использования `>`:**
```bash
echo "Первая строка" > output.txt
cat output.txt
```

**Вывод консоли:**
```bash
raddd1us@ubuntu:~$ echo "Первая строка" > output.txt
raddd1us@ubuntu:~$ cat output.txt
Первая строка
```

**Пример использования `>`:**
```bash
echo "Вторая строка" >> output.txt
cat output.txt
```

**Вывод консоли:**
```bash
raddd1us@ubuntu:~$ echo "Вторая строка" >> output.txt
raddd1us@ubuntu:~$ cat output.txt
Первая строка
Вторая строка
```

## 2. Стандартные потоки и файловые дескрипторы
*В ОС Linux всё представлено в виде файлов. Для работы с вводом и выводом процессов используются стандартные файловые дескрипторы:*
 - stdin (Standard Input, дескриптор 0) — стандартный поток ввода. По умолчанию считывает данные с клавиатуры.
 - stdout (Standard Output, дескриптор 1) — стандартный поток вывода. Используется для вывода штатных результатов работы программы (по умолчанию выводится на экран терминала).
 - stderr (Standard Error, дескриптор 2) — стандартный поток ошибок. Используется для вывода сообщений об ошибках и диагностической информации (по умолчанию также выводится на экран терминала)

**Главное отличие stdout от stderr: stdout предназначен для полезных данных команды, тогда как stderr используется исключительно для служебных сообщений и ошибок. Разделение потоков позволяет фильтровать или сохранять логи ошибок отдельно от основных результатов выполнения программы.**

**Пример перенаправления только stdout в файл:**
```bash
ls -l output.txt 1> stdout_only.txt
cat stdout_only.txt
```

**Вывод консоли:**
```bash
raddd1us@ubuntu:~$ ls -l output.txt 1> stdout_only.txt
raddd1us@ubuntu:~$ cat stdout_only.txt
-rw-r--r-- 1 raddd1us raddd1us 28 Oct  5 15:30 output.txt
```

**Пример перенаправления только stderr в файл**
```bash
ls nonexistent_file.txt 2> stderr_only.txt
cat stderr_only.txt
```

**Вывод консоли:**
```bash
raddd1us@ubuntu:~$ ls nonexistent_file.txt 2> stderr_only.txt
raddd1us@ubuntu:~$ cat stderr_only.txt
ls: cannot access 'nonexistent_file.txt': No such file or directory
```

**Пример объединения stdout и stderr в один файл**
*Для слияния потоков вывода и ошибок в один файл применяется оператор &>*
```bash
raddd1us@ubuntu:~$ ls nonexistent_file.txt 2> stderr_only.txt
raddd1us@ubuntu:~$ cat stderr_only.txt
ls: cannot access 'nonexistent_file.txt': No such file or directory
```

**Вывод консоли:**
```bash
raddd1us@ubuntu:~$ ls -l output.txt nonexistent_file.txt &> combined.txt
raddd1us@ubuntu:~$ cat combined.txt
ls: cannot access 'nonexistent_file.txt': No such file or directory
-rw-r--r-- 1 raddd1us raddd1us 28 Oct  5 15:30 output.txt
```

**Пример раздельного перенаправления stdout и stderr в разные файлы**
```bash
ls -l output.txt nonexistent_file.txt 1> success.log 2> error.log
cat success.log
cat error.log
```

**3. Вывод содержимого файла без текстовых редакторов**
*Просмотр содержимого файла выполняется с помощью утилиты cat:*
```bash
cat output.txt
```

**Вывод консоли:**
```bash
raddd1us@ubuntu:~$ cat output.txt
Первая строка
Вторая строка
```

**4. Создание файла с содержимым без текстового редактора**
*Создание файла с помощью перенаправления потока вывода утилиты echo:*

```bash
echo "Hello, Linux World!" > new_file.txt
cat new_file.txt
```

**Вывод консоли**
```bash
raddd1us@ubuntu:~$ echo "Hello, Linux World!" > new_file.txt
raddd1us@ubuntu:~$ cat new_file.txt
Hello, Linux World!
```

**5. Перенаправление потоков**
**5.1. Перенаправить только stdout в файл**
*При явном перенаправлении stdout используется запись 1> (или просто >):*
```bash
ls -l output.txt 1> stdout_only.txt
cat stdout_only.txt
```

**Вывод консоли:**
```bash
raddd1us@ubuntu:~$ ls -l output.txt 1> stdout_only.txt
raddd1us@ubuntu:~$ cat stdout_only.txt
-rw-r--r-- 1 raddd1us raddd1us 28 Oct  5 15:30 output.txt
```

**5.2. Перенаправить только stderr в файл**
*Для перенаправления ошибок используется запись 2>:*
```bash
ls nonexistent_file.txt 2> stderr_only.txt
cat stderr_only.txt
```

**Вывод консоли:**
```bash
raddd1us@ubuntu:~$ ls nonexistent_file.txt 2> stderr_only.txt
raddd1us@ubuntu:~$ cat stderr_only.txt
ls: cannot access 'nonexistent_file.txt': No such file or directory
```

**5.3. Перенаправить stdout и stderr в один файл**
*Для объединения потоков вывода и ошибок в один файл используется оператор &>:*
```bash
ls -l output.txt nonexistent_file.txt &> combined.txt
cat combined.txt
```

**Вывод консоли:**
```bash
raddd1us@ubuntu:~$ ls -l output.txt nonexistent_file.txt &> combined.txt
raddd1us@ubuntu:~$ cat combined.txt
ls: cannot access 'nonexistent_file.txt': No such file or directory
-rw-r--r-- 1 raddd1us raddd1us 28 Oct  5 15:30 output.txt
```

**5.4. Перенаправить stdout и stderr в разные файлы**
*Одновременное перенаправление потоков вывода и ошибок в два разных файла:*
```bash
ls -l output.txt nonexistent_file.txt 1> success.log 2> error.log
cat success.log
cat error.log
```

**Вывод консоли:**
```bash
raddd1us@ubuntu:~$ ls -l output.txt nonexistent_file.txt 1> success.log 2> error.log
raddd1us@ubuntu:~$ cat success.log
-rw-r--r-- 1 raddd1us raddd1us 28 Oct  5 15:30 output.txt
raddd1us@ubuntu:~$ cat error.log
ls: cannot access 'nonexistent_file.txt': No such file or directory
```

**6. Передача данных через стандартный ввод (stdin)**
*Передача данных из файла в команду cat через оператор перенаправления ввода <:*
```bash
cat < output.txt
```

**Вывод консоли:**
```bash
raddd1us@ubuntu:~$ cat < output.txt
Первая строка
Вторая строка
```

**7. Использование /dev/null (глушение ненужного вывода)**
*Специальное виртуальное устройство /dev/null уничтожает любые перенаправленные в него данные.*

*Отправка потока ошибок stderr в /dev/null:*
```bash
ls nonexistent_file.txt 2> /dev/null
```

*Полное глушение и stdout, и stderr:*
```bash
ls -l output.txt nonexistent_file.txt &> /dev/null
```





















