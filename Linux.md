

## netstat命令

**netstat -tunlp** 用于显示 tcp，udp 的端口和进程等相关情况。

netstat 查看端口占用语法格式：

```bash
netstat -tunlp | grep 端口号
```

- -t (tcp) 仅显示tcp相关选项
- -u (udp)仅显示udp相关选项
- -n 拒绝显示别名，能显示数字的全部转化为数字(不加的话端口号可能不会显示为数字而是进程名)
- -l 仅列出在Listen(监听)的服务状态
- -p 显示建立相关链接的程序名

例如查看 8000 端口的情况，使用以下命令：

```bash
# netstat -tunlp | grep 8000
tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      26993/nodejs   
```

## lsof命令

```bash
lsof -i:8080：查看8080端口占用
lsof abc.txt：显示开启文件abc.txt的进程
lsof -c abc：显示abc进程现在打开的文件
lsof -c -p 1234：列出进程号为1234的进程所打开的文件
lsof -g gid：显示归属gid的进程情况
lsof +d /usr/local/：显示目录下被进程开启的文件
lsof +D /usr/local/：同上，但是会搜索目录下的目录，时间较长
lsof -d 4：显示使用fd为4的进程
lsof -i -U：显示所有打开的端口和UNIX domain文件
```

## ls命令

```shell
ls -l                    # 以长格式显示当前目录中的文件和目录
ls -a                    # 显示当前目录中的所有文件和目录，包括隐藏文件
ls -lh                   # 以人类可读的方式显示当前目录中的文件和目录大小
ls -t                    # 按照修改时间排序显示当前目录中的文件和目录
ls -R                    # 递归显示当前目录中的所有文件和子目录
ls -l /etc/passwd        # 显示/etc/passwd文件的详细信息
```
## ps命令
**常用参数**
```
-a：显示所有用户的进程，包括其他用户拥有的进程。
-u：以用户为主的格式显示进程状态。
-x：显示所有进程，不只是会话中的进程。
-e：显示所有进程，包括系统守护进程。
-f：以完整格式显示进程状态。
-l：以长格式显示进程状态。
-C：根据进程名称进行过滤。
-p：查看指定进程号的进程状态。
-t：指定要显示的终端名称。
```
#### ps
```
 Develop>ps
   PID TTY          TIME CMD
 80816 pts/0    00:00:00 bash
 85353 pts/0    00:00:00 ps
```
默认的 ps 命令显示当前用户在当前终端会话中运行的进程。其输出通常包括以下字段：
```
PID: 进程ID
TTY: 终端类型
TIME: 进程使用的CPU时间
CMD: 运行的命令
```
#### ps -aux
```
 Develop>ps -aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.0  34396  4816 ?        Ss   Jul19   2:02 /sbin/init
root          2  0.0  0.0      0     0 ?        S    Jul19   0:00 [kthreadd]
root          3  0.0  0.0      0     0 ?        I<   Jul19   0:00 [rcu_gp]
root          4  0.0  0.0      0     0 ?        I<   Jul19   0:00 [rcu_par_gp]
root          6  0.0  0.0      0     0 ?        I<   Jul19   0:00 [kworker/0:0H-kb]
root          9  0.0  0.0      0     0 ?        I<   Jul19   0:00 [mm_percpu_wq]

USER: 进程的所有者
PID: 进程 ID
%CPU: CPU 使用率
%MEM: 内存使用率
VSZ: 虚拟内存大小
RSS: 驻留内存大小
TTY: 终端类型
STAT: 进程状态
START: 进程启动时间
TIME: 使用的 CPU 时间
COMMAND: 运行的命令
```
#### ps -ef
```
Develop>ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 Jul19 ?        00:02:03 /sbin/init
root          2      0  0 Jul19 ?        00:00:00 [kthreadd]
root          3      2  0 Jul19 ?        00:00:00 [rcu_gp]
root          4      2  0 Jul19 ?        00:00:00 [rcu_par_gp]
root          6      2  0 Jul19 ?        00:00:00 [kworker/0:0H-kb]
root          9      2  0 Jul19 ?        00:00:00 [mm_percpu_wq]
root         10      2  0 Jul19 ?        00:00:00 [ksoftirqd/0]
root         11      2  0 Jul19 ?        00:09:06 [rcu_sched]
root         12      2  0 Jul19 ?        00:00:00 [migration/0]

UID: 用户 ID，即进程的所有者
PID: 进程 ID
PPID: 父进程 ID
C: CPU 使用率
STIME: 进程启动时间
TTY: 终端类型
TIME: 使用的 CPU 时间
CMD: 运行的命令

 Develop>ps -elf
F S UID         PID   PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S root          1      0  0  80   0 -  8599 ep_pol Jul19 ?        00:02:03 /sbin/init
1 S root          2      0  0  80   0 -     0 kthrea Jul19 ?        00:00:00 [kthreadd]
1 I root          3      2  0  60 -20 -     0 rescue Jul19 ?        00:00:00 [rcu_gp]

F: 进程标志，表示进程的状态和调度信息。
S: 进程状态，例如 S（睡眠），R（运行），Z（僵尸）等。
UID: 用户 ID，表示进程的所有者。
PID: 进程 ID。
PPID: 父进程 ID。
C: CPU 使用率。
PRI: 进程优先级。
NI: 尼斯值，影响进程优先级。
ADDR: 内存地址。
SZ: 内存大小。
WCHAN: 如果进程在睡眠，显示正在等待的内核函数。
STIME: 进程启动时间。
TTY: 终端类型。
TIME: 使用的 CPU 时间。
CMD: 运行的命令。
```

ps -aux和ps -elf都是查看进程的命令，但是它们提供的信息以及格式略有不同。ps -aux主要关注的是进程的资源占用情况，而ps -elf则更多的关注进程的状态和从属关系。

#### 进一步过滤
查看包含sshd的进程
```
 Develop>ps -aux |grep sshd
root        973  0.0  0.0   6692   312 pts/0    S+   12:20   0:00 grep sshd
root      28245  0.0  0.0  30812  4824 ?        Ss   10:49   0:00 sshd: develop@pts/2
root     127830  0.0  0.0  24084  4320 ?        Ss   Jul19   0:00 sshd: /usr/sbin/sshd -D -e -E /opt/nsfocus/log/sshd/sshd.log [listener] 0 of 10-60 startups
root     131037  0.0  0.0  30812  4672 ?        Rs   12:20   0:00 sshd: develop@pts/0
```
查看包含sshd的进程，以及第一行参数
```
# ps -aux | grep -E '(^USER|sshd)'：匹配以 "USER" 开头的行和包含 "sshd" 的行。
 Develop>ps -aux | grep -E '(^USER|sshd)'
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root      22455  0.2  0.0  30812  4672 ?        Ss   12:29   0:00 sshd: develop@pts/0
root      22659  0.0  0.0   6696   312 pts/0    S+   12:29   0:00 grep -E (^USER|sshd)
root      28245  0.0  0.0  30812  4824 ?        Ss   10:49   0:00 sshd: develop@pts/2
root     127830  0.0  0.0  24084  4320 ?        Ss   Jul19   0:00 sshd: /usr/sbin/sshd -D -e -E /opt/nsfocus/log/sshd/sshd.log [listener] 0 of 10-60 startups

# 在这里，NR==1 表示第一行（标题行），/python/ 表示匹配包含 python 的行。
 Develop>ps -aux | awk 'NR==1 || /sshd/'
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root      22455  0.0  0.0  30812  4672 ?        Ss   12:29   0:00 sshd: develop@pts/0
root      24344  0.0  0.0   9888  2244 pts/0    R+   12:29   0:00 awk NR==1 || /sshd/
root      28245  0.0  0.0  30812  4824 ?        Ss   10:49   0:00 sshd: develop@pts/2
root     127830  0.0  0.0  24084  4320 ?        Ss   Jul19   0:00 sshd: /usr/sbin/sshd -D -e -E /opt/nsfocus/log/sshd/sshd.log [listener] 0 of 10-60 startups

```
## pstree

显示所有运行中的进程的树状图:

```shell
fujh_wsl2@Fantasy:~$ pstree
systemd─┬─2*[agetty]
        ├─cron
        ├─dbus-daemon
        ├─init-systemd(Ub─┬─SessionLeader───Relay(285)───bash───pstree
        │                 ├─init───{init}
        │                 ├─login───bash
        │                 └─{init-systemd(Ub}
        ├─mysqld───36*[{mysqld}]
        ├─networkd-dispat
        ├─redis-server───4*[{redis-server}]
        ├─rsyslogd───3*[{rsyslogd}]
        ├─sshd
        ├─systemd───(sd-pam)
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-resolve
        ├─systemd-udevd
        └─unattended-upgr───{unattended-upgr}
```

显示每个进程的PID:

```shell
fujh_wsl2@Fantasy:~$ pstree -p
systemd(1)─┬─agetty(183)
           ├─agetty(185)
           ├─cron(119)
           ├─dbus-daemon(121)
           ├─init-systemd(Ub(2)─┬─SessionLeader(283)───Relay(285)(284)───bash(285)───pstree(491)
           │                    ├─init(7)───{init}(8)
           │                    ├─login(286)───bash(349)
           │                    └─{init-systemd(Ub}(9)
           ├─mysqld(197)─┬─{mysqld}(204)
           │             ├─{mysqld}(205)
           │             ├─{mysqld}(206)
           │             ├─{mysqld}(207)
           │             ├─{mysqld}(208)
           │             ├─{mysqld}(209)
           │             ├─{mysqld}(210)
           │             ├─{mysqld}(211)
           │             ├─{mysqld}(212)
           │             ├─{mysqld}(213)
           │             ├─{mysqld}(215)
           │             ├─{mysqld}(216)
           │             ├─{mysqld}(217)
           │             ├─{mysqld}(218)
           │             ├─{mysqld}(219)
           │             ├─{mysqld}(220)
           │             ├─{mysqld}(228)
           │             ├─{mysqld}(229)
           │             ├─{mysqld}(230)
           │             ├─{mysqld}(234)
           │             ├─{mysqld}(235)
           │             ├─{mysqld}(236)
           │             ├─{mysqld}(237)
           │             ├─{mysqld}(238)
           │             ├─{mysqld}(239)
           │             ├─{mysqld}(240)
           │             ├─{mysqld}(244)
           │             ├─{mysqld}(245)
           │             ├─{mysqld}(246)
           │             ├─{mysqld}(247)
           │             ├─{mysqld}(248)
           │             ├─{mysqld}(249)
           │             ├─{mysqld}(253)
           │             ├─{mysqld}(254)
           │             ├─{mysqld}(255)
           │             └─{mysqld}(257)
           ├─networkd-dispat(132)
           ├─redis-server(139)─┬─{redis-server}(192)
           │                   ├─{redis-server}(193)
           │                   ├─{redis-server}(194)
           │                   └─{redis-server}(195)
           ├─rsyslogd(140)─┬─{rsyslogd}(173)
           │               ├─{rsyslogd}(174)
           │               └─{rsyslogd}(175)
           ├─sshd(176)
           ├─systemd(343)───(sd-pam)(344)
           ├─systemd-journal(37)
           ├─systemd-logind(157)
           ├─systemd-resolve(110)
           ├─systemd-udevd(64)
           └─unattended-upgr(191)───{unattended-upgr}(198)
```

查看某个进程的进程树:

```shell
fujh_wsl2@Fantasy:~$ pstree -p 139
redis-server(139)─┬─{redis-server}(192)
                  ├─{redis-server}(193)
                  ├─{redis-server}(194)
                  └─{redis-server}(195)
```

## 查看某个进程的线程

```shell
fujh_wsl2@Fantasy:~$ ps -T -p 139
    PID    SPID TTY          TIME CMD
    139     139 ?        00:00:02 redis-server
    139     192 ?        00:00:00 bio_close_file
    139     193 ?        00:00:00 bio_aof_fsync
    139     194 ?        00:00:00 bio_lazy_free
    139     195 ?        00:00:00 jemalloc_bg_thd
    
fujh_wsl2@Fantasy:~$ top -H -p 139
top - 15:38:46 up 49 min,  1 user,  load average: 0.00, 0.00, 0.00
Threads:   5 total,   0 running,   5 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.1 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7765.2 total,   6846.1 free,    766.5 used,    152.5 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   6792.9 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    139 redis     20   0   67244  17312   8112 S   0.3   0.2   0:03.22 redis-server
    192 redis     20   0   67244  17312   8112 S   0.0   0.2   0:00.00 bio_close_file
    193 redis     20   0   67244  17312   8112 S   0.0   0.2   0:00.00 bio_aof_fsync
    194 redis     20   0   67244  17312   8112 S   0.0   0.2   0:00.00 bio_lazy_free
    195 redis     20   0   67244  17312   8112 S   0.0   0.2   0:00.00 jemalloc_bg_thd
```



## perf

## grep命令

语法:

```shell
grep [options] pattern [files]
pattern - 表示要查找的字符串或正则表达式。
files - 表示要查找的文件名，可以同时查找多个文件，如果省略 files 参数，则默认从标准输入中读取数据。

常用选项：：
-i：忽略大小写进行匹配。
-v：反向查找，只打印不匹配的行。
-n：显示匹配行的行号。
-r：递归查找子目录中的文件。
-l：只打印匹配的文件名。
-c：只打印匹配的行数。
```

示例：

```shell
1、在文件 file.txt 中查找字符串 "hello"，并打印匹配的行：
grep hello file.txt

2、在文件夹 dir 中递归查找所有文件中匹配正则表达式 "pattern" 的行，并打印匹配行所在的文件名和行号：
grep -r -n pattern dir/

3、在标准输入中查找字符串 "world"，并只打印匹配的行数：
echo "hello world" | grep -c world

4、在当前目录中，查找后缀有 file 字样的文件中包含 test 字符串的文件，并打印出该字符串的行。此时，可以使用如下命令：
grep test *file 
结果如下所示：
$ grep test test* #查找前缀有“test”的文件包含“test”字符串的文件  
testfile1:This a Linux testfile! #列出testfile1 文件中包含test字符的行  
testfile_2:This is a linux testfile! #列出testfile_2 文件中包含test字符的行  
testfile_2:Linux test #列出testfile_2 文件中包含test字符的行 

5、以递归的方式查找符合条件的文件。例如，查找指定目录/etc/acpi 及其子目录（如果存在子目录的话）下所有文件中包含字符串"update"的文件，并打印出该字符串所在行的内容，使用的命令为：
grep -r update /etc/acpi 

6、反向查找。前面各个例子是查找并打印出符合条件的行，通过"-v"参数可以打印出不符合条件行的内容。
查找文件名中包含 test 的文件中不包含test 的行，此时，使用的命令为：
grep -v test *test*
```



## awk命令



## tcpdump



## chmod

只有文件所有者和超级用户可以修改文件或目录的权限。可以使用绝对模式（八进制数字模式），符号模式指定文件的权限。

<div align=center><img src=".\fig\chmod.webp"height="465"/> </div>

示例：

```shell
将文件 file1.txt 设为所有人皆可读取 :
chmod ugo+r file1.txt
或者
chmod a+r file1.txt

将文件 file1.txt 与 file2.txt 设为该文件拥有者，与其所属同一个群体者可写入，但其他以外的人则不可写入 :
chmod ug+w,o-w file1.txt file2.txt


chmod a=rwx file 和 chmod 777 file 效果相同

chmod ug=rwx,o=x file 和 chmod 771 file 效果相同


```



## tee

Linux tee命令用于读取标准输入的数据，并将其内容输出成文件：

```she
fujh_wsl2@Fantasy:~/test_linux$ ls
fujh_wsl2@Fantasy:~/test_linux$ tee test1
hello
hello
^C
fujh_wsl2@Fantasy:~/test_linux$ cat test1
hello
```

## find

语法：

```shell
find [路径] [匹配条件] [动作]
路径:是要查找的目录路径，可以是一个目录或文件名，也可以是多个路径，多个路径之间用空格分隔，如果未指定路径，则默认为当前目录。

匹配条件:
-name pattern：按文件名查找，支持使用通配符 * 和 ?。
-type type：按文件类型查找，可以是 f（普通文件）、d（目录）、l（符号链接）等。
-user username：按文件所有者查找。
-group groupname：按文件所属组查找。
-size [+-]size[cwbkMG]：按文件大小查找，支持使用 + 或 - 表示大于或小于指定大小

动作: 可选的，用于对匹配到的文件执行操作，比如删除、复制等。
-amin n：查找在 n 分钟内被访问过的文件。
-cmin n：查找在 n 分钟内状态发生变化的文件（例如权限）。
-ctime n：查找在 n*24 小时内状态发生变化的文件（例如权限）。
-mmin n：查找在 n 分钟内被修改过的文件。
-mtime n：查找在 n*24 小时内被修改过的文件。
关于时间 n 参数的说明：

+n：查找比 n 天前更早的文件或目录。

-n：查找在 n 天内更改过属性的文件或目录。

n：查找在 n 天前（指定那一天）更改过属性的文件或目录。

```



示例：

```shell
find . -name file.txt #查找当前目录下名为 file.txt 的文件：
find . -name "*.c"  #将当前目录及其子目录下所有文件后缀为 .c 的文件列出来:
find . -type f    #将当前目录及其子目录中的所有文件列出：
find /home -size +1M #查找 /home 目录下大于 1MB 的文件：
find /var/log -mtime +7 #查找 /var/log 目录下在 7 天前修改过的文件：

```



## which

which指令会在环境变量$PATH设置的目录里查找符合条件的文件。

```shell
$ which bash #使用指令"which"查看指令"bash"的绝对路径
```



## whereis

该指令会在特定目录中查找符合条件的文件。这些文件应属于原始代码、二进制文件，或是帮助文件。该指令只能用于查找二进制文件、源代码文件和man手册页，一般文件的定位需使用locate命令。

```shell
参数：
-b 　只查找二进制文件。
-B<目录> 　只在设置的目录下查找二进制文件。
-f 　不显示文件名前的路径名称。
-m 　只查找说明文件。
-M<目录> 　只在设置的目录下查找说明文件。
-s 　只查找原始代码文件。
-S<目录> 　只在设置的目录下查找原始代码文件。
-u 　查找不包含指定类型的文件。
```

示例：

使用指令"whereis"查看指令"bash"的位置，输入如下命令：

```shell
$ whereis bash 
bash: /usr/bin/bash /usr/share/man/man1/bash.1.gz
$ whereis -b bash 
bash: /usr/bin/bash
$ whereis -m bash 
bash: /usr/share/man/man1/bash.1.gz
```



## cat命令

 示例：

查看文件内容：显示文件 filename 的内容。
```
cat filename # cat file1 file2...可以显示多个
```
创建文件：将标准输入重定向到文件 filename该文件的内容。(如果没有则创建)
```
cat > filename # 覆盖的方式
cat >> filename # 追加的方式
```
连接文件：将 file1 和 file2 的内容合并到 file3 中。
```
cat file1 file2 > file3 # file3中文本的顺序为file1的内容加上file2的内容
```
使用管道：将 cat 命令的输出作为另一个命令的输入。
```
cat filename | command
```
查看文件的最后几行：显示文件 filename 的最后 10 行。
```
cat filename | tail -n 10
```
使用 -n 选项显示行号：显示文件 filename 的内容，并在每行的前面加上行号。
```
cat -n filename # 使用cat -b选项则仅显示非空行的行号
```
## tail

**tail -f filename** 会把 filename 文件里的最尾部的内容显示在屏幕上，并且不断刷新，只要 filename 更新就可以看到最新的文件内容。

```shell
命令格式: tail [参数] [文件]  

参数：
-f 循环读取
-q 不显示处理信息
-v 显示详细的处理信息
-c<数目> 显示的字节数
-n<行数> 显示文件的尾部 n 行内容
--pid=PID 与-f合用,表示在进程ID,PID死掉之后结束
-q, --quiet, --silent 从不输出给出文件名的首部
-s, --sleep-interval=S 与-f合用,表示在每次反复的间隔休眠S秒
```

```she
要显示 notes.log 文件的最后 10 行，请输入以下命令：
tail notes.log         # 默认显示最后 10 行

要跟踪名为 notes.log 的文件的增长情况，请输入以下命令：
tail -f notes.log
此命令显示 notes.log 文件的最后 10 行。当将某些行添加至 notes.log 文件时，tail 命令会继续显示这些行。 显示一直继续，直到您按下（Ctrl-C）组合键停止显示。

tail -n +20 notes.log：
这个命令显示 notes.log 文件中从第20行开始的所有行。+ 符号告诉 tail 从文件的指定行号开始显示，而不是显示最后 n 行。在这个例子中，+20 表示从第20行开始显示，直到文件的末尾。

tail -n 20 notes.log：
这个命令显示 notes.log 文件的最后20行。如果没有指定 + 符号，-n 选项默认显示文件的最后 n 行。在这个例子中，20 表示显示最后20行。

显示文件 notes.log 的最后 10 个字符:
tail -c 10 notes.log
```





## du

```
-a或-all 显示目录中个别文件的大小。
-h或--human-readable 以K，M，G为单位，提高信息的可读性。
-s或--summarize 仅显示指定目录或文件的总大小，而不显示其子目录的大小。。
--exclude=<目录或文件> 略过指定的目录或文件。
--max-depth=<目录层数> 超过指定层数的目录后，予以忽略。
```

```
[fujinghao@74205a35316a ~]$ ls
client-odbc          inotify                    odbc                            openGauss-connector-odbc-5.0.0.zip  self-security-suite.v1.0      study     tmp
client-psycopg2      inotify-tools-3.13         odbc-sor                        psutil-6.0.0                        self-security-suite.v1.0.tar  template  xl2tpd-1.1.11
client-psycopg2.tar  inotify-tools-3.13.tar.gz  openGauss-connector-odbc-5.0.0  psutil-6.0.0.tar.gz                 server-5.0.1                  test
```

```
[fujinghao@74205a35316a ~]$ du -sh
6.7G    .
```

```
[fujinghao@74205a35316a ~]$ du -sh *
88M     client-odbc
41M     client-psycopg2
12M     client-psycopg2.tar
4.0K    inotify
1.8M    inotify-tools-3.13
384K    inotify-tools-3.13.tar.gz
9.9M    odbc
4.0K    odbc-sor
101M    openGauss-connector-odbc-5.0.0
4.2M    openGauss-connector-odbc-5.0.0.zip
5.7M    psutil-6.0.0
500K    psutil-6.0.0.tar.gz
10M     self-security-suite.v1.0
8.7M    self-security-suite.v1.0.tar
2.9G    server-5.0.1
16K     study
344K    template
20K     test
3.5G    tmp
664K    xl2tpd-1.1.11
```

```
[fujinghao@74205a35316a ~]$ du -ah test/
12K     test/test
4.0K    test/test.c
20K     test/
```

```
[fujinghao@74205a35316a ~]$ du -sh test/
20K     test/
```

```
[fujinghao@74205a35316a ~]$ du -sh server-5.0.1/*
4.0K    server-5.0.1/add_empty_dir.sh
401M    server-5.0.1/gauss_arm
326M    server-5.0.1/gauss_x86
4.0K    server-5.0.1/nsbuild.json
4.0K    server-5.0.1/nsecos_svn.info
4.0K    server-5.0.1/README.md
```

```
[fujinghao@74205a35316a ~]$ du -sh server-5.0.1/* --exclude=server-5.0.1/gauss_arm
4.0K    server-5.0.1/add_empty_dir.sh
326M    server-5.0.1/gauss_x86
4.0K    server-5.0.1/nsbuild.json
4.0K    server-5.0.1/nsecos_svn.info
4.0K    server-5.0.1/README.md
[fujinghao@74205a35316a ~]$ du -ah server-5.0.1/* --max-depth=1
4.0K    server-5.0.1/add_empty_dir.sh
4.0K    server-5.0.1/gauss_arm/prepare_chroot.sh
4.0K    server-5.0.1/gauss_arm/prepare_start.sh
4.0K    server-5.0.1/gauss_arm/start_gauss_db.sh
401M    server-5.0.1/gauss_arm/gauss
4.0K    server-5.0.1/gauss_arm/stop_gauss_db.sh
4.0K    server-5.0.1/gauss_arm/remove_gauss.sh
401M    server-5.0.1/gauss_arm
326M    server-5.0.1/gauss_x86/gauss
326M    server-5.0.1/gauss_x86
4.0K    server-5.0.1/nsbuild.json
4.0K    server-5.0.1/nsecos_svn.info
4.0K    server-5.0.1/README.md
```

```
[fujinghao@74205a35316a lib]$ du -sh * | sort -hr
17M     postgresql
5.1M    libdssapi.so
4.7M    libdms.so
4.3M    libxgboost.so
2.8M    libcrypto.so.1.1
2.8M    libcrypto.so
2.0M    libpq_ce.so.5.5
1.5M    libxml2.so.2.9.13
1.5M    libxml2.so.2
1.5M    libxml2.so
1.5M    libstdc++.so.6
1008K   libkrb5_gauss.so.3.3
972K    libiconv.so.2.6.1
```

## df

显示文件系统的磁盘使用情况统计：

```shell
$ df 
Filesystem     1K-blocks    Used     Available Use% Mounted on 
/dev/sda6       29640780 4320704     23814388  16%     / 
udev             1536756       4     1536752    1%     /dev 
tmpfs             617620     888     616732     1%     /run 
none                5120       0     5120       0%     /run/lock 
none             1544044     156     1543888    1%     /run/shm 

```

## free

```shell
语法:
free [-bkmotV][-s <间隔秒数>]
-b 　以Byte为单位显示内存使用情况。
-k 　以KB为单位显示内存使用情况。
-m 　以MB为单位显示内存使用情况。
-h 　以合适的单位显示内存使用情况，最大为三位数，自动计算对应的单位值。单位有：
B = bytes
K = kilos
M = megas
G = gigas
T = teras
-o 　不显示缓冲区调节列。
-s<间隔秒数> 　持续观察内存使用状况。
-t 　显示内存总和列。
-V 　显示版本信息。

fujh_wsl2@Fantasy:~/test_linux$ free
               total        used        free      shared  buff/cache   available
Mem:         7951548      787008     7001164        2928      163376     6950276
Swap:        2097152           0     2097152


fujh_wsl2@Fantasy:~/test_linux$ free -th
               total        used        free      shared  buff/cache   available
Mem:           7.6Gi       769Mi       6.7Gi       2.0Mi       159Mi       6.6Gi
Swap:          2.0Gi          0B       2.0Gi
Total:         9.6Gi       769Mi       8.7Gi


```

