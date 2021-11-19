# devops-netology

## Домашнее задание к занятию "3.2. Работа в терминале, лекция 2"

1) cd - это встроенная команда Bash. Логично, если указатель на директорию меняется внутри текущей сессии терминала внутренней функцией, в которой она выполняется, не затрагивая другие. 
Даже используя внешний вызов, например, если мы работаем в рамках сессии на удаленном сервере, используя ssh, команда будет работать только с рабочей оболочкой (в данном случае, удаленного сервера) не затрагивая вызвавший еe shell. 
Если команда будет независящей от сессии и их, представим, будет много, то в таком случае ей будет сложно понять, для какой из них нужно ее применить.
2) Согласно документации, wc - команда которая печатает число символов новой строки, слов и байт для каждого файла. wc -l печатает кол-во строк. Альтернативой grep <some_string> <some_file> | wc -l будет являться
grep <some_string> <some_file> -с

```bash
$ touch test.txt
$ echo '123123 test' >> test.txt
$ grep 123 test.txt | wc -l
1
$ grep 123 test.txt -c
1
```

3) Процесс с PID 1 - systemd (узнать можно через команды top или pstree -p)
4) Например, вызов из /dev/pts/0 для перенаправления в dev/pts/1 будет выглядеть следующим образом
```bash
$ ls -l /test 2>/dev/pts/1
```

5) Последовательность команд:

```bash
$ touch test.txt
$ echo '123' test.txt
$ cat < test.txt > test1.txt
$ cat test1.txt
123
```

6) Можно, при использовании команды 

```bash
$ echo Hello >/dev/tty3
```
однако, в графическом режиме за этим не получится наблюдать, нужно переключиться в TTY (Ctrl-Alt-F3 в моем случае).

7) 5>&1 приведет к созданию (если такого не существует) и перенаправлению дескриптора с идентификатором 5 в stdout. Сообщения данного дескриптора передаются туда же, куда и стандартный вывод. echo netology > /proc/$$/fd/5 выведет netology. При выполнении ее выполнении поток stdout команды echo перенаправляется в дескриптор 5, который в свою очередь перенаправляется в stdout.

8) ls -l /test 5>&2 2>&1 1>&5 | grep cannot -c. Перенаправляем поток с дескриптора 5 на stderr, с stderr на stderr на stdout, с stdout на дескриптор 5.

9) cat /proc/$$/environ выводит текущие переменные окружения. В качестве альтернативы можно использовать printenv и env.

10) /proc/<PID>/cmdline - в этом файле хранится командная строка, которой был запущен данный процесс. /proc/<PID>/exe представляет собой символическую ссылку на исполняемый файл, который инициировал запуск процесса.

11) Если я верно понял, процессор на моем компьютере не поддерживает SSE, последовательность комманды.

```bash
$ cd /proc
$ cat cpuinfo | grep sse
```
выводит флаги процессоров, cat cpuinfo | grep SSE* не выводит ничего.
12) Из-за того, что по-умолчанию ожидается пользователь, а не процесс. Как обходное решение, использовать флаг -t.
```bash
ssh -t localhost 'tty'
```
13) Установил screen и reptyr. Запускаем новый pty, запускаем процесс, для примера взял top (PID 29020). Набираем команду screen, ждем запуска. Жмем Ctrl + a c (создаем новый терминал). 
В начале reptyr ругался на права, в итоге поменял значение параметра kernel.yama.ptrace_scope на 0 в файле 10-patrace.conf (судя по описанию в файле 0 - более разрешающий режим).
Набираем reptyr -T 29020, процесс перехватывается в screen, завершается на старом терминале можно его закрывать, в новом терминале pstree -p | grep 29020 
показывает работающий процесс с данным PID.
14) Команда tee считывает стандартный ввод и записывает его одновременно в стандартный вывод и в один или несколько подготовленных файлов.
В данном случае, команда будет работать, т.к. при получении stdin после выполнения echo, который перенаправлен через pipe, то т.к. команда запущена с sudo, tee имеет права на запись в файл.
