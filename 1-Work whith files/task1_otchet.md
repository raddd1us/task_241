# Григорьев Сергей 241 группа
# Работа в консоли

1) Переместиться между директориями.
2) Вывести список файлов в директории.
3) Вывести список всех файлов, включая скрытые.
4) Создать папку с несколькими уровнями подпапок.
5) Внутри папки создать файл и записать в него несколько строк текста.
6) Переместить файл из одной директории в другую.
7) Скопировать файл из одной директории в другую.
8) Переименовать файл.
9) Сравнить содержимое двух файлов.
10) Отсортировать содержимое файла по возрастанию и убыванию.
11) Найти созданный файл по имени, не переходя вручную по всей структуре каталогов.
12) Удалить созданные в ходе задания файлы и каталоги.

## Критерий приёмки

В отчёте должны быть:
- команды, использованные для создания каталогов и файлов;
- итоговая структура созданных каталогов;
- команды перемещения, копирования и переименования файла;
- команда сравнения двух файлов и её результат;
- команды сортировки по возрастанию и убыванию;
- команда поиска файла по имени и результат;
- подтверждение удаления созданных в ходе задания объектов;
- краткое пояснение разницы между абсолютным и относительным путём.

# Отчёт по практической работе: Работа в консоли

Работа выполнена на сервере `student-13` под пользователем `student`.

## 1. Переместиться между директориями

Использованные команды:

```bash
cd ~
pwd
cd /var/log
pwd
cd ..
pwd
cd -
pwd
cd /
pwd
```

Полученный результат:

```bash
[student@student-13 /]$ cd ~
[student@student-13 ~]$ pwd
/home/student
[student@student-13 ~]$ cd /var/log
[student@student-13 log]$ pwd
/var/log
[student@student-13 log]$ cd ..
[student@student-13 var]$ pwd
/var
[student@student-13 var]$ cd -
/var/log
[student@student-13 log]$ pwd
/var/log
[student@student-13 log]$ cd /
[student@student-13 /]$ pwd
/
```

**Разбор команд:**
- **cd** (change directory) — смена текущей рабочей директории.
- **pwd** — вывод полного пути текущего каталога.
- **~** — домашний каталог текущего пользователя (`/home/student`).
- **/var/log** — переход по абсолютному пути.
- **..** — родительский каталог (уровень выше).
- **-** — возврат в каталог, где пользователь был до последнего перехода (команда сама выводит этот путь).
- **/** — корень файловой системы.

---

## 2. Вывести список файлов в директории

Использованная команда:

```bash
ls -l
```

Полученный результат:

```bash
[student@student-13 ~]$ ls -l
total 0
```

**Разбор параметров:**
- **ls** (list) — вывод содержимого каталога.
- **-l** (long format) — подробный формат: права, владелец, размер, дата изменения.

В домашнем каталоге нет видимых (нескрытых) файлов и папок, поэтому список пуст.

---

## 3. Вывести список всех файлов, включая скрытые

Использованная команда:

```bash
ls -la
```

Полученный результат:

```bash
[student@student-13 ~]$ ls -la
total 60
drwx------ 8 student student 4096 Sep 15 07:00 .
drwxr-xr-x 3 root    root    4096 Sep 14 22:38 ..
-rw------- 1 student student  257 Nov 22 16:31 .bash_history
-rw------- 1 student student  217 Jul 26  2021 .bash_logout
-rw------- 1 student student  259 Jul 26  2021 .bash_profile
-rw------- 1 student student  188 Jul 26  2021 .bashrc
drwx------ 2 student student 4096 Jul 26  2021 .cache
drwx------ 2 student student 4096 Jul 26  2021 .config
drwx------ 3 student student 4096 Oct  7  2025 .local
-rw------- 1 student student   17 Jul 26  2021 .lpoptions
drwx------ 3 student student 4096 Oct  7  2025 .mutt
-rw------- 1 student student  121 Jul 26  2021 .rpmmacros
drwx------ 2 student student 4096 Oct  7  2025 .ssh
-rwx------ 1 student student  319 Jul 26  2021 .xprofile
drwx------ 2 student student 4096 Jul 26  2021 .xsession.d
```

**Разбор параметров:**
- **-a** (--all) — показывает все объекты, включая скрытые (имена которых начинаются с точки), а также записи `.` (текущий каталог) и `..` (родительский).
- **-la** — сочетание подробного формата и показа скрытых объектов.

Без `-a` каталог выглядел пустым (`total 0`), а на самом деле в нём 13 скрытых объектов.

---

## 4. Создать папку с несколькими уровнями подпапок

Использованная команда:

```bash
mkdir -p ~/lab1_dir/level2/level3
```

Команда выполнилась без вывода и ошибок.

**Разбор параметров:**
- **mkdir** (make directory) — создание каталогов.
- **-p** (--parents) — создаёт всю цепочку вложенных каталогов сразу; промежуточные `lab1_dir` и `level2` создаются автоматически.

---

## 5. Внутри папки создать файл и записать в него несколько строк текста

Использованные команды:

```bash
cat > ~/lab1_dir/level2/level3/source.txt <<'EOF'
банан
яблоко
вишня
апельсин
EOF
cat ~/lab1_dir/level2/level3/source.txt
wc -l ~/lab1_dir/level2/level3/source.txt
```

Полученный результат:

```bash
[student@student-13 ~]$ cat > ~/lab1_dir/level2/level3/source.txt <<'EOF'
> банан
> яблоко
> вишня
> апельсин
> EOF
[student@student-13 ~]$ cat ~/lab1_dir/level2/level3/source.txt
банан
яблоко
вишня
апельсин
[student@student-13 ~]$ wc -l ~/lab1_dir/level2/level3/source.txt
4 /home/student/lab1_dir/level2/level3/source.txt
```

**Разбор параметров:**
- **<<'EOF'** — Here-Document: ввод нескольких строк до стоп-слова `EOF`.
- **>** — перенаправление вывода в файл (создаёт файл или перезаписывает его).
- **wc -l** — подсчёт строк в файле.

При первой попытке в файл попала лишняя пустая строка, поэтому файл был перезаписан этой же командой. В итоговом файле ровно 4 строки.

---

## 6. Переместить файл из одной директории в другую

Использованные команды:

```bash
mv ~/lab1_dir/level2/level3/source.txt ~/lab1_dir/level2/source.txt
ls ~/lab1_dir/level2/level3 ~/lab1_dir/level2
```

Полученный результат:

```bash
[student@student-13 ~]$ mv ~/lab1_dir/level2/level3/source.txt ~/lab1_dir/level2/source.txt
[student@student-13 ~]$ ls ~/lab1_dir/level2/level3 ~/lab1_dir/level2
/home/student/lab1_dir/level2:
level3  source.txt

/home/student/lab1_dir/level2/level3:
```

**Разбор:** **mv** (move) переносит файл. Первый аргумент — откуда, второй — куда. Файл ушёл из `level3` в `level2`, каталог `level3` теперь пуст.

---

## 7. Скопировать файл из одной директории в другую

Использованные команды:

```bash
cp ~/lab1_dir/level2/source.txt ~/lab1_dir/copied_file.txt
ls ~/lab1_dir
```

Полученный результат:

```bash
[student@student-13 ~]$ cp ~/lab1_dir/level2/source.txt ~/lab1_dir/copied_file.txt
[student@student-13 ~]$ ls ~/lab1_dir
copied_file.txt  level2
```

**Разбор:** **cp** (copy) создаёт копию. Первый аргумент — исходный файл, второй — путь и имя копии. Оригинал остался в `level2`.

---

## 8. Переименовать файл

Использованные команды:

```bash
mv ~/lab1_dir/copied_file.txt ~/lab1_dir/final_doc.txt
ls ~/lab1_dir
```

Полученный результат:

```bash
[student@student-13 ~]$ mv ~/lab1_dir/copied_file.txt ~/lab1_dir/final_doc.txt
[student@student-13 ~]$ ls ~/lab1_dir
final_doc.txt  level2
```

**Разбор:** отдельной команды переименования нет, используется `mv`: если новый путь указывает в тот же каталог с другим именем, файл переименовывается.

---

## 9. Сравнить содержимое двух файлов

Использованные команды:

```bash
diff ~/lab1_dir/level2/source.txt ~/lab1_dir/final_doc.txt
echo $?
```

Полученный результат:

```bash
[student@student-13 ~]$ diff ~/lab1_dir/level2/source.txt ~/lab1_dir/final_doc.txt
[student@student-13 ~]$ echo $?
0
```

**Разбор:**
- **diff** — построчное сравнение двух файлов.
- **echo $?** — вывод кода завершения последней команды: `0` — различий нет, `1` — файлы различаются.

Вывод `diff` пустой, код возврата `0`: содержимое файлов полностью идентично (второй файл — копия первого).

---

## 10. Отсортировать содержимое файла по возрастанию и убыванию

Использованные команды:

```bash
sort ~/lab1_dir/final_doc.txt
sort -r ~/lab1_dir/final_doc.txt
```

Полученный результат:

```bash
[student@student-13 ~]$ sort ~/lab1_dir/final_doc.txt
апельсин
банан
вишня
яблоко
[student@student-13 ~]$ sort -r ~/lab1_dir/final_doc.txt
яблоко
вишня
банан
апельсин
```

**Разбор:**
- **sort** — сортировка строк по возрастанию (по алфавиту).
- **-r** (--reverse) — обратный порядок (по убыванию).

---

## 11. Найти созданный файл по имени, не переходя вручную по всей структуре каталогов

Использованные команды:

```bash
find ~/lab1_dir -name "final_doc.txt"
tree ~/lab1_dir || find ~/lab1_dir | sort
```

Полученный результат:

```bash
[student@student-13 ~]$ find ~/lab1_dir -name "final_doc.txt"
/home/student/lab1_dir/final_doc.txt
[student@student-13 ~]$ tree ~/lab1_dir || find ~/lab1_dir | sort
-bash: tree: command not found
/home/student/lab1_dir
/home/student/lab1_dir/final_doc.txt
/home/student/lab1_dir/level2
/home/student/lab1_dir/level2/level3
/home/student/lab1_dir/level2/source.txt
```

**Разбор параметров:**
- **find** — поиск объектов по дереву каталогов.
- **~/lab1_dir** — каталог, с которого начинается поиск.
- **-name "final_doc.txt"** — поиск по точному имени.

Файл найден автоматически: `/home/student/lab1_dir/final_doc.txt`.

**Итоговая структура созданных каталогов.** Утилита `tree` на сервере не установлена (`command not found`), поэтому структура получена через `find`:

```text
/home/student/lab1_dir
/home/student/lab1_dir/final_doc.txt
/home/student/lab1_dir/level2
/home/student/lab1_dir/level2/level3
/home/student/lab1_dir/level2/source.txt
```

То есть в `lab1_dir` лежат файл `final_doc.txt` и каталог `level2`, а в `level2` — пустой каталог `level3` и файл `source.txt`.

---

## 12. Удалить созданные в ходе задания файлы и каталоги

Использованные команды:

```bash
rm -ri ~/lab1_dir
ls -l ~/lab1_dir
ls -la ~
```

Полученный результат:

```bash
[student@student-13 ~]$ rm -ri ~/lab1_dir
rm: descend into directory '/home/student/lab1_dir'? y
rm: descend into directory '/home/student/lab1_dir/level2'? y
rm: remove regular file '/home/student/lab1_dir/level2/source.txt'? y
rm: remove directory '/home/student/lab1_dir/level2/level3'? y
rm: remove directory '/home/student/lab1_dir/level2'? y
rm: remove regular file '/home/student/lab1_dir/final_doc.txt'? y
rm: remove directory '/home/student/lab1_dir'? y
[student@student-13 ~]$ ls -l ~/lab1_dir
ls: cannot access '/home/student/lab1_dir': No such file or directory
[student@student-13 ~]$ ls -la ~
total 60
drwx------ 8 student student 4096 Nov 22 16:44 .
drwxr-xr-x 3 root    root    4096 Sep 14 22:38 ..
-rw------- 1 student student  257 Nov 22 16:31 .bash_history
-rw------- 1 student student  217 Jul 26  2021 .bash_logout
-rw------- 1 student student  259 Jul 26  2021 .bash_profile
-rw------- 1 student student  188 Jul 26  2021 .bashrc
drwx------ 2 student student 4096 Jul 26  2021 .cache
drwx------ 2 student student 4096 Jul 26  2021 .config
drwx------ 3 student student 4096 Oct  7  2025 .local
-rw------- 1 student student   17 Jul 26  2021 .lpoptions
drwx------ 3 student student 4096 Oct  7  2025 .mutt
-rw------- 1 student student  121 Jul 26  2021 .rpmmacros
drwx------ 2 student student 4096 Oct  7  2025 .ssh
-rwx------ 1 student student  319 Jul 26  2021 .xprofile
drwx------ 2 student student 4096 Jul 26  2021 .xsession.d
```

**Разбор параметров:**
- **rm** (remove) — удаление файлов и каталогов.
- **-r** (recursive) — рекурсивное удаление каталога вместе с содержимым.
- **-i** (interactive) — запрос подтверждения перед удалением каждого объекта.

Каталог `lab1_dir` удалён: `ls` сообщает `No such file or directory`. В домашнем каталоге остались только исходные скрытые файлы.

---

## Разница между абсолютным и относительным путём

- **Абсолютный путь** начинается от корневого каталога `/` и указывает полный маршрут к объекту. Он одинаково работает из любого текущего каталога. Пример: `/home/student/lab1_dir/final_doc.txt`.
- **Относительный путь** строится от текущей рабочей директории и не начинается со слеша. Пример: если мы находимся в `/home/student`, тот же файл задаётся как `lab1_dir/final_doc.txt`, а из `lab1_dir/level2` — как `../final_doc.txt`.
