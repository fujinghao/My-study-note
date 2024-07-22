

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

## losf命令

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
## grep命令
## awk命令
## cat命令
### 实例
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