---
title: shell-cmd
layout: default
parent: Linux
has_children: false
---

## 常用命令，[归档参考](/assets/file/Reference.md)

### xargs

```shell

# 可以将输出转换为其他命令的输入参数（非命令的输入）
# -d    界定符，同 cut -d
# -n    每行 Item 数量
# -I    指定一个替换字符，常用"{}"
# -t    打印出最终要执行的命令，然后直接执行，不需要用户确认
# -P    指定 xargs 后的子进程并发数，若为 0 则不限制，将自动根据参数数量 fork 相应个子进程

# 尽可能的并发多个 wget 进程进行文件下载:
cat ips.txt | xargs -n 1 -P 0 -I {} wget -q -e http_proxy={} -O {} "http://domain/path/to/url"


# 比如并发下载
urls=(
    "http://example1.com"
    "http://example2.com"
    "http://example3.com"
)

num=${#urls[@]}
printf "%s\n" "${urls[@]}" | xargs -n 1 -P $num curl -O
```

### wait

```shell

# wait 阻塞当前进程的执行，直至指定的子进程执行结束后才继续执行
# 使用 wait 可以在 bash 脚本"多进程"执行模式下起到一些特殊控制的作用
# 若 wait 后不带任何进程号或作业号，则阻塞当前进程的执行，直至当前进程的所有子进程都执行结束后才继续执行

# 格式
# wait [进程号或作业号]
# 如 wait 23 or wait %1
# —————————————————————————————————————————————————————————————————————————————————————————— 

#！/bin/sh  
echo "1"  
sleep 5&  
echo "3"  
echo "4"  

wait
# 会等待 wait 所在 SHELL 的所有子进程执行结束，本例中是 sleep 5 这句

echo "5"
```

### uuidgen

```shell
# 生成 UUID 值
#  -r, --random     generate random-based uuid
#  -t, --time       generate time-based uuid
#  -V, --version    output version information and exit
#  -h, --help       display this help and exit

uuidgen -t
# 19df1690-96aa-11eb-93e1-286ed489f459

```

### tree

```shell
# 通过树的形式列出当前目录-L 代表深度如果为1则只列出一层目录
tree -L 1 .
```

### trap

```shell
# trap 是内建命令，用于在脚本中指定信号如何处理
trap -p             # 将当前的 trap 设置打印出来
trap -l             # 打印所有信号
trap "cmd" signals  # 当接收到指定信号时执行指定命令
trap signals        # 若没有明亮部分则默认将信号处理复原，如: trap INT 即恢复 Ctrl+C 退出
trap "" signals     # 忽略信号，可多个，如: trap "" INT 表明忽略 SIGINT 信号 => 按 Ctrl+C 也不能使脚本退出
trap "cmd" EXIT     # 脚本退出时执行 cmd
trap "cmd" ERR      # 当命令出错，退出码非0，执行cmd
trap "cmd" RETURN   # 当从 SHELL 函数返回、或用 SOURCE 执行另一脚本文件时执行 commands
trap -- signals     # 删除捕获
```

### split

```shell
split:
    -a          # 指定后缀长度
    -d          # 用数字作后缀
    -b          # 根据容量分割，单位是 byte，如 split -b 10k -d -a 3 dateFile
    -l n        # 根据行数分割，合并时 cat file* >> file


# 数字方式命名，每份 30M（若按行分隔则使用 -l），排序后缀占 3 位
split -d -b 30m -a 3 abc.tar.gz
# 合并文件
cat x00* > abc.tar.gz
```

### rename

```shell
# 提供用字符串替换的方式批量的更改文件名
#   -v   显示修改的文件名
#   -n   不执行任何的操作，主要用来测试（查看效果）
#   -f   强制修改
# —————————————————————————————————————————————————————————————————————————————————————————— 

rename "s/AA/aa/" *             # 把文件名中的 AA 替换成 aa
rename "s/\.html/\.php/" *      # 修改文件名中的 .html 后缀替换为 .php
rename "s/$/\.txt/" *           # 批量添加文件后缀
rename 's/ //g' *               # 去除文件名的空格
```

### mktemp

```shell
# 生成临时文件
FILE_PATH=$(mktemp)
ll $FILE_PATH
# -rw------- 1 root root 0 Mar 26 15:32 /tmp/tmp.R5Cg3RC8Ka

# 生成临时目录
DIR_PATH=$(mktemp -d)
ll $DIR_PATH
# total 0

# 生成临时目录（内存）
mktemp --tmpdir=/dev/shm 2>> /dev/null
```

### hexdump

```shell
# 查看任何文件的十六进制编码，常用于查看二进制文件
-n  # Length 只格式化输入文件的前length个字节
-C  # 输出规范的十六进制和 ASCII 码（常用）
-c  # 单字节字符显示
-b  # 单字节八进制显示
-d  # 双字节十进制显示
-o  # 双字节八进制显示
-x  # 双字节十六进制显示
-s  # 从偏移量开始输出
-v  # 不压缩重复行


hexdump -Cv /tmp/file1.txt
# 00000000  49 20 6e 65 65 64 20 74  6f 20 62 75 79 20 61 70  |I need to buy ap|
# 00000010  70 6c 65 73 2e 0a 49 20  6e 65 65 64 20 74 6f 20  |ples..I need to |
# 00000020  72 75 6e 20 74 68 65 20  6c 61 75 6e 64 72 79 2e  |run the laundry.|
# 00000030  0a 49 20 6e 65 65 64 20  74 6f 20 77 61 73 68 20  |.I need to wash |
# 00000040  74 68 65 20 64 6f 67 2e  0a 49 20 6e 65 65 64 20  |the dog..I need |
# 00000050  74 6f 20 67 65 74 20 74  68 65 20 63 61 72 20 64  |to get the car d|
# 00000060  65 74 61 69 6c 65 64 2e  0a                       |etailed..|
# 00000069

# 第一列表示文件文件偏移量，第二列以两个字节为一组的十六进制
# 注意并非所有十六进制值都有对应的可打印 ASCII 字符，这种情况下将显示点 . 或其他占位符
# ——————————————————————————————————————————————————————————————————————————————————————————

# 双字节十进制显示
echo -n A | hexdump -d
# 0000000   00065

echo -n A | hexdump -C -d
# 00000000  41                                                |A|
# 0000000   00065
# 0000001
```

###  fock

```bash

# flock [-sxon] [-w timeout] lockfile [-c]command...
# flock [-sxun] [-w timeout] fd

# Args ...
# -s  为共享锁，在定向为某文件的FD上设置共享锁而未释放锁的时间内，其他进程在该文件FD上设置独占锁的请求失败，试图在定向为此文件的FD上设置共享锁的请求会成功
# -x  为独占或排他锁，在定向为某文件FD设置独占锁而未释放锁的时间内其他进程试图在此文件FD上设置共享锁或独占锁都会失败。只要未设置-s参数，此参数默认被设置
# -u  手动解锁，一般情况不必须，当FD关闭时系统会自动解锁，此参数用于脚本命令一部分需要异步执行，一部分可以同步执行的情况
# -n  为非阻塞模式，当试图设置锁失败，采用非阻塞模式，直接返回1，并继续执行下面语句。
# -w  设置阻塞超时，当超过设置的秒数，就跳出阻塞，返回1，并继续执行下面语句。
# -o  必须是使用第一种格式时才可用，表示当执行command前关闭设置锁的FD，以使command的子进程不保持锁。
# -c  执行其后的 comand

# crontab 借助 flock 防止重复执行
0 23 * * * (flock -xn ./test.lock -c "sh /root/test.sh") # -n 为非阻塞模式

# —————————————————————————————————————————————————————————————————————————————————————————— 

#！/bin/bash
{
  flock –n 3
  [$? –eq 1] && { echo fail ;exit}
  echo succeed
  # Doyour task here …
} 3 <> mylockfile


# 首先使用<>打开mylockfile 并定向到文件描述符3，而定向文件描述符是先于命令执行的，因此假如要执行的语句段中需要读写mylockfile文件
# 例如想获得上一个脚本实例的pid，并将此次的脚本实例的pid写入mylockfile
# 此时直接用>打开mylockfile会清空上次存入的内容，而用<打开mylockfile当它不存在时会导致一个错误
# 其次以非阻塞的方式打开mylockfile，flock –n 3，其中3是mylockfile的文件描述符，若能够正常获取锁（默认是排它锁），则继续往下执行，否则返回1
# 该返回码能够在接下来的语句中[ $? –eq 1 ] 进行判断，若为1，则输出失败并退出当前脚本
```

### find

```shell
-name       # 按文件名查找: find -name "*.txt" cp -ap {} {}.bkup \;
-iname      # 按文件名查找: find -iname "MyCProgram.c"（不匹配大小写，相当于 grep -rl -i "MyCProgram.c" .）
-maxdepth   # 最大目录搜索深度: find / -maxdepth 10 -exec basename {} \; (仅输出指定目录深度下的文件名 )
-mindepth   # 最小目录搜索深度: find / -maxdepth 5 -mindepth 2 -name "*.conf" -exec ls -l {} \;
-not        # 相反匹配: find -maxdepth 1 -not -iname "被排除的文件名"
-inum       # 使用文件系统inode号查找文件: find -inum 804180 -exec rm {} \;
-perm       # 按对象权限类型搜索（格式: -perm -u=rwx / -perm 0777 ）
-type       # 搜索文件类型（f,d,s,c,b,...）: find . -type f -iname "*.mp3" -exec rename "s/ /_/g" {} \;
-empty      # 内容为空的文件: find / -empty （当搜索路径被指定为绝对路径时其输出也是绝对路径！）
-size       # 指定小于或大于特定大小的文件: find / -size +60k -exec ls -hl {} \;
-mmin       # 指定小于或大于分钟范围内容被修改过的文件
-mtime      # 指定小于或大于天数范围内容被修改过的文件
-amin       # 指定小于或大于分钟范围被访问过的: find -mmin -/+60 -exec ls -l {} \;
-atime      # 指定小于或大于天数范围被访问过的: find -atime -/+1
-cmin       # 指定小于或大于分钟范围被修改过内容的
-ctime      # 指定小于或大于天数范围被修改过内容的
-never      # 查找比某一文件修改时间还要新的文件: find -newer /etc/passwd
-anewer     # 查找比某一文件访问时间还要新的文件
-cnever     # 查找比某一文件状态改变时间还要新的文件
-xdev       # 仅在当前文件系统中搜索: find / -xdev -name "*.log"
-user       # 指定用户的文件: find / -user root
-nouser     # 无属主的
-nogroup    # 无属组的
-readable   # 可读的
-writable   # 可写的
-fstype     # 属于特定文件类型的
-gid        # 特定属 GID 的文件
-regex      # 按正则进行搜索: find . -regex ".*/[0-9]*/.c" -print
-prune      # 不进入子目录搜索（仅对当前目录）
-exec       # 对搜索结果执行的操作: find -name "*.html" -exec ./bash_script.sh '{}' \;
-execdir    #
-ok         #
-okdir      #
-path       # 排除 /proc 目录: find / ! -path "/proc" -type f -name '*.jar'
-delete     # 对搜索结果执行删除动作，例如 find /tmp -depth -name core -type f -delete

# ——————————————————————————————————————————————————————————————————————————————————————————

# 输出文件大小并按大小排序
find /tmp -type f -print0 | xargs -0 ls -Ssh1 --color

# 查找特定时间范围内的文件 
find /log/ -newermt '2013-08-08' ! -newermt '2013-09-01' -type f

find test \(-path test/test4 -o -path test/test3 \) -prune -o -name "*.log" -print

# 查找某目录下超过固定大小的文件，并清空其文件内容
find /data/logs/apache/ -type f -size +1G -exec sh -c "> {}" \;


```

### date

```shell
%F  # 即: yyyy-mm-dd（+%Y-%m-%d）
%y  # 年份的最后两位数字 (00.99)
%Y  # 完整年份 (0000-9999)
%m  # 月份 (01-12)
%d  # 日 (01-31)
%H  # 小时(00-23)
%M  # 分钟(00-59)
%S  # 秒(00-60)
%b  # 月份 (Jan-Dec)
%B  # 月份 (January-December)
%D  # 直接显示日期 (mm/dd/yy)
%T  # 直接显示时间 (24小时制)
%h  # 同 %b
%n  # 下一行
%t  # 跳格
%I  # 小时(01-12)
%k  # 小时(0-23)
%l  # 小时(1-12)
%p  # 显示本地 AM 或 PM
%r  # 直接显示时间 (12小时制，格式为 hh:mm:ss [AP]M)
%s  # 从1970年1月1日 00:00:00 UTC 到目前为止的秒数
%X  # 相当于 %H:%M:%S
%Z  # 显示时区
%a  # 星期几 (Sun-Sat)
%A  # 星期几 (Sunday-Saturday)
%c  # 直接显示日期与时间
%j  # 一年中的第几天 (001-366)
%U  # 一年中的第几周 (00-53) (以 Sunday 为一周的第一天的情形)
%w  # 一周中的第几天 (0-6)
%W  # 一年中的第几周 (00-53) (以 Monday 为一周的第一天的情形)
%x  # 直接显示日期 (mm/dd/yy)
%N  # 纳秒

# —————————————————————————————————————————————————————————————————————————————————————————— Example

date --date='@2147483647'
# Convert seconds since the epoch (1970-01-01 UTC) to a date

date "+%Y-%m-%d %H:%M:%S.%3N"
# 毫秒精度 2024-05-25 14:48:32.886

echo '2024-05-25 14:16:33.323' | date "+%H:%M:%S.%3N" -f -
# 从标准输入读取时间戳文本，并输出指定格式的时间戳

date "+now time: %y-%m-%d %H:%M:%S"
# now time: 17-08-15 22:24:36

date "+三年前的此刻是: %y-%m-%d %H:%M:%S" -d "-3 years"
# 三年前的此刻是: 14-08-15 22:25:52

date "+三个月后时间是: %y-%m-%d %H:%M:%S" -d "+3 months"
# 三个月后时间是: 17-11-15 22:26:38

date "+十天之后时间是: %y-%m-%d %H:%M:%S" -d "+10 days"
# 十天之后时间是: 17-08-25 22:27:22

date -d "+8 hours 2021-08-27 08:08:08" "+%d-%m-%Y %H:%M:%S"
date -d "-8 hours 2021-08-27 08:08:08" "+%d-%m-%Y %H:%M:%S"
# 根据给定的时间加减8小时

ps -p $PID --no-headers -o lstart 2>>/dev/null | xargs -I {} date -d {} "+%d-%m-%Y %H:%M:%S"
# 根据给定的时间格式化输出

# 设置系统时间的几个例子（足够智能）
date -s "20171027 20:49:30"
date -s "20:49:30 20171027"
date -s "20:49:30 2017/10/27"
date -s "20:49:30 2017-10-27"
date -s 2021-10-5
date -s 16:30

# 一年中的第几天
date "+%j"
# 300

# Unix 时间戳转换
date -d '@1615909456'
# Tue Mar 16 23:44:16 CST 2021

# Unix 时间差
date -d '1970-01-01 00:00:01' +%s
# -28799

# 偷懒的方法
date "+%F %T"
# 2017-10-27 20:52:33

# 昨天日期（1天前）
date -d last-day +%Y-%m-%d
date -d "1 days ago" +%Y-%m-%d
date -d '-1 days' +%Y-%m-%d

# 下周一日期
date -d 'next monday' +%Y-%m-%d

# 明天日期
date -d next-day +%Y-%m-%d
date -d '1 days' +%Y-%m-%d

# 上个月的今天
date -d last-month +%Y-%m-%d

# 下个月的今天
date -d next-month +%Y-%m-%d

```

### dirname & dirname

```shell
pwd
# /etc/sysconfig/modules

basename `pwd`
# modules

dirname `pwd`
# /etc/sysconfig
```

### grep

```shell

# -a                  不忽略二进制的数据
# -A                  显示匹配行及其向下 N 行，例如 grep -A 3 -F "string..."
# -B                  显示匹配行及其向上 N 行
# -C                  显示匹配行及其上下 N 行
# -e                  使用正则搜索
# -E                  使用扩展正则搜索
# -o                  只显示被匹配的模式内容，例如 grep -o -e "x\{1,2\}.*"
# -c                  统计输出被匹配行的次数
# -n                  显示匹配行的行号
# -d                  说明要查找的对象是目录
# -r                  对目录进行递归式的搜索
# -f                  从文件中读取要配匹配的模式，例如统计 file2 中有而 file1 中没有的行: grep -vFf file1 file2
# -F                  将范本样式视为固定字串（非正则速度快，相当于 fgrep，例如统计两个文件共有的行: grep -Ff file1 file2
# -h                  仅输出匹配内容而不显示被匹配的文件名
# -H                  输出匹配内容的同时也输出被匹配文件名
# -l                  输出被匹配的文件的文件名，常用于针对路径的递归搜索，例如 grep -rl "关键字" .
# -L                  输出未被匹配的文件名，常用于针对路径的递归搜索，例如多文件查询: grep -L "str" logs1.log logs2.log
# -i                  忽略关键字大小写的差异
# -q                  不输出任何信息，仅用于脚本中判断是否匹配到相关内容`
# -s                  不显示错误信息
# -v                  排除查找（输出没有被匹配的行）
# -y                  此参数的效果和指定 -i 参数相同
# -w、--word-regexp   只显示全字符合的列
# -x、--line-regexp   只显示全列符合的列
# --color             对匹配进行高亮显示
# --exclude           过滤不需要匹配的文件类型，例如 grep -rl "keyword" . --exclude *.log
# --include           指定匹配的文件类型

# 常用参数: -A -B -C -o -E -P -F -w -i -rl -n -v -c -q

# —————————————————————————————————————————————————————————————————————————————————————————— 

# 统计 F2 文件和 F1 文件中都存在的
grep -Fv F2 F1

# 统计 F2 文件中有而 F1 文件中没有的
grep -Fv F2 F1

# —————————————————————————————————————————————————————————————————————————————————————————— PERL

# 使用 perl 正则
grep -P '^\d{3}-\d{2}-\d{4}$' file.txt

# 使用 perl 正则，借助零宽断言
echo 'test:123abc123' | grep -oP '(?<=\d{3})\w+(?=\d{3})' 
# abc

# 使用 perl 正则，借助 (?P<name> ...) 的语法来定义命名分组，并通过 \g<name> 或 \k<name> 来引用命名分组!
echo 'test:123abc123' | grep -oP '(?P<X>\d{3})\w+(\g<X>)'
# 123abc123

echo 'test:123abc123' | grep -oP '(?P<X>\d{3})[[:alpha:]]+(\g<X>)' 
# 123abc123

echo 'test:123abc123' | grep -oP '(?P<X>\d{3})\w+(\g<X>)' | sed -E 's/[[:alpha:]]+//g'
# 123123

# —————————————————————————————————————————————————————————————————————————————————————————— 

cat <<'EOF' | /bin/bash
strings="test:123abc123"
if [[ "$strings" =~ (123) ]]; then
    echo -n "${BASH_REMATCH[0]}"
    echo -n "${BASH_REMATCH[1]}"
    echo
fi
EOF

# 输出 123123

```

### printf

```shell
printf 'number: %-+8.2f \t string: %s \n ' $number $string
#
# 格式符:
#     %d,%i     十进制整数
#     %s        字符串
#     %f        浮点数，例如 %-8.2f 表示左对齐且宽度为 8，并在其后保留两位小数（符号 + 表示显示数据类型的符号）
#     %u        不带正负号的十进制值
#     %%        表示 % 自身
#
# 转义:
#   \f      换页
#   \t      水平制表
#   \v      垂直制表
#   \b      后退
#   \n      换行（输出时默认不会换行）
#   \r      回车
#   \\      自身

# 以 0 填充长度为 5
printf "%05d\n" 15
# 00015

printf "[ %05d ]\n" {00..05}
# [ 00000 ]
# [ 00001 ]
# [ 00002 ]
# [ 00003 ]
# [ 00004 ]
# [ 00005 ]
```

### sed

```shell


sed [options] 'command' file(s)
sed [options] -f scriptfile file(s)

# Args ...
#   -n      不输出模式空间中未被改变的内容
#   -i      就地编辑原文件，并且若提供了后缀则进行备份，例如 sed -i.bak 's/1/2/g' file
#   -e      在同一行中执行多条编辑命令，例如 sed -e 's/1/2/g' -e 's/a/b/g' 或 sed -r 's/1/2/g;s/a/b/g'
#   -r,-E   使用扩展正则
#   -f      调用脚本文件
#   --debug

# command parameter ...
#   i       在当前行上方插入文本，即 Insert
#   c       修改当前行整行的文本，即 Change
#   a       在当前行下方插入文本，即 Append
#   p       输出模式空间中的内容，即 Print，如 -n '1~2p' 输出奇数行（从第1行算起每2行输出1次）
#   d       删除当前行（删除模式空间，进行下一个循环）
#   e       将替换后的新字符串作为命令执行，如 echo 'cmd hello' | sed 's/cmd/echo/e' 将输出 hello
#   r       从指定文件的内容读入到该行，如 r fileName
#   w       将模式空间的内容追加到文件，如 w fileName
#   W       将模式空间第一行内容追加到文件，如 W fileName
#   D       将模式空间第一行内容删除并重新执行脚本顶端的命令
#   s       替换指定字符
#   l       列表不能打印字符的清单
#   h       用模式空间的数据覆盖掉保持空间
#   H       将模式空间的数据追加到保持空间，例如 '1,2H;$G' 表示将第 1 至第 2 行全部追加到文件末尾
#   g       将保持空间的数据覆盖掉模式空间
#   G       将保持空间的数据追加到模式空间，例如 '1{h;d};$G' 表示剪切第一行内容并将其追加到最后一行
#   x       替换两个空间中的数据，例如 '/A/h;/B/x' 将关键字 A 与关键字 B 所在行互换
#   n       读取下一行并使用后续命令处理新的行而非从头开始，例如 -n 'p;n' 表示打印奇数行
#   N       将下一行追加到当前模式空间并在二者间嵌入换行符，且改变当前处理的行号
#   P       输出模式空间中第一行的内容
#   =       打印当前行号，例如 -n '/root/=' 表示输出 root 关键字所在的行号，相当于 grep -n 'root'
#   b       lable 分支到脚本中带有标记的地方，若分支不存在则分支到脚本末尾
#   t       label if 分支，从末行开始，若满足或 T/t 将导致分支到带有标号的命令处，或到脚本的末尾
#   T       label 错误分支，从末行开始，若发生错误或者 T/t 将导致分支到带有标号的命令处，或到脚本的末尾
#   !       排除/取反，对未被选定的部分执行后面的操作
#   #       把注释扩展到下一个换行符以前
#   q       退出执行，如 '10q' 表示打印第 10 行后退出
#   g       全部替换，例如 's/A/B/g' 表示将所有 A 改为 B、's/A/B/2' 表示将所有 A 改为 B 且仅替换 2 次
#   y       将单个字符转为其他字符（不使用正则），例如 'y/123/abc/g' 表示将1替换为a、2替换为b、3替换为...
#   &       被匹配模式匹配住的所有内容，例如 's/123/abc&/g' 表示将 123 替换为 abc123
#   \1      此处数字即匹配模式中分组的下标，例如 -E 's/(123).*(789)/\2\1/' 表示将 789 与 123 的位置互换

# —————————————————————————————————————————————————————————————————————————————————————————— 
# examples


sed '/^$|#/!p'                  # 不输出空行或注释行
sed '2,$d'                      # 从第 2 行删到末尾
sed '/ccc/{x;p;x;}'             # 在匹配行前加入空行
sed '{N;s/\n/\t/;}'             # 将偶数行与其上面的奇数行合为一行，其中 N 将第 2 行追加到模式空间，此时模式空间 2 行，然后替换换行符
sed 10q                         # 输出前 10 行
sed '{$!N;$!d;}'                # 输出后 2 行
sed '$d'                        # 删除最后一行
sed '{n;d;}'                    # 删除偶数行
sed '{n;G;}'                    # 在偶数行后插入空行
sed '{n;n;G;}'                  # 在第 3、6、9、12、… 行后插入空行
sed G                           # 在每行后面增加一行空行
sed 'G;G'                       # 在每行后面增加两行空行
sed '/^$/d'                     # 删除空白行
sed -n '1,5p'                   # 输出第 1 至 5 行
sed -n '5,/^test/p'             # 输出第 5 行到以 test 开头的行之间的数据
sed -n '/test/,/check/p'        # 输出含 test 的行到 check 的行之间的数据
sed -nE '/^.{65}/p'             # 输出含 65 个或以上字符的行
sed '$!N;s/\n/ /'               # 将 2 行链接生成一行，模拟 paste
sed -n '/regexp/{n;p;}'         # 查找 regexp 并仅将匹配行的下一行输出
sed -n '/regexp/{g;1!p;};h'     # 查找 regexp 并仅将匹配行的上一行输出
sed -n 's/foo/bar/3'            # 将 foo 替换成 bar，只替换第 3 次被匹配的关键字
sed -n 's/foo/bar/3g'           # 将 foo 替换成 bar，从第 3 次被匹配的位置开始往后全部替换
sed '/baz/!s/foo/bar/g'         # 将 foo 替换成 bar，且只在没有出现 baz 关键字时替换
sed -n '3~7p'                   # 从第 3 行开始，每 7 行显示一次，或 sed -n '3,${p;n;n;n;n;n;n;}'
gsed '0~8d'                     # 删除 8 的倍数行
sed -n '$='                     # 模拟 wc -l
sed '{1!G;h;$!d;}'              # 模拟 tac 命令进行逆序输出
sed '$!N;$!D'                   # 显示后两行，模拟 tail -2
sed -e :a -e '$q;N;11,$D;ba'    # 模拟 tail 显示文件中的最后 10 行
sed '/^$/d;G'                   # 将原所有空行删除并在每行后增加一空行,这样在输出的文本中每一行后面将有且只有一空行
sed 's|[0-9]|<&>|g'             # 把文件中的数字都替换成 <num> 样式（右侧的 g 表示全局替换，sed 默认仅替换第 1 个匹配）
sed 's|[0-9]|<&>|2'             # 把前面正则表达式找到的第二列以后内容进行替换
sed 'n;d'                       # 将第一个脚本所产生的所有空行删除（即删除所有偶数行）和 ed '/^$/d' 的效果一样。
sed = filename                  # 对文件中的所有行编号（行号在左，文字右端对齐）
echo 1a2b | sed -r 's/.(.)../\1/g'              # 输出 a
sed -e :a -e '$d;N;2,10ba' -e 'P;D'             # 删除最后 10 行
sed '/\n/!G;s/\(.\)\(.*\n\)/&\2\1/;//D;s/.//'   # 模拟 rev 命令倒置行字串输出
sed '/./=' filename | sed '/./N; s/\n/ /'       # 对文件中的所有行编号，但只显示非空白行的行号。
sed '/\n/!G;s/\(.\)\(.*\n\)/&\2\1/;//D;s/.//'   # 将行中的字符逆序排列，第一个字成为最后一字，……（模拟"rev"）
sed '$!N;s/\n/ /'                               # 将每两行连接成一行，类似 "paste"
sed -e :a -e '/\\$/N; s/\\\n//; ta'             # 如果当前行以反斜杠 \ 结束，则将下一行并到当前行末尾并去掉原来行尾的反斜杠
sed '$!N;$!D'                                   # 模拟 tail -2 命令显示文件中的最后 2 行
sed -e '/./{H;$!d;}' -e 'x;/AAA/b' -e '/BBB/b' -e '/CCC/b' -e d       # 显示含 "AAA"、"BBB"、"CCC" 3 者任 1 字串的段落
sed '$!N; s/^\(.*\)\n\1$/\1/; t; D'             # 删除除重复行外的所有行，模拟 uniq -d
sed 's/\(.*\)foo/\1bar/'                        # 替换最后一个 foo
sed 's/\(.*\)foo\(.*foo\)/\1bar\2/'             # 替换倒数第二个 foo

```

### awk

```shell

# awk 用于处理文本和数据，执行逐行扫描，寻找匹配模式的行并执行自定义操作，支持自定义函数和动态正则表达式等先进功能 ...
# 若未指定模式，则所有的行都被处理
# 若未指定动作，则把输出到标准输出

awk -F 'pattern' 'condition{action}' filename

# 若不指定 fileName 或指定 - 则处理标准输入
# 特殊模式（若存在多个则以定义时的顺序处理）
#   BEGIN{}  处理前先执行（可不加文件参数）
#   END{}    处理完再执行

# —————————————————————————————————————————————————————————————————————————————————————————— 内置变量

# ARGC            命令行参数数量
# ARGV            命令行参数组成的数组
# $0              完整一行记录
# $n              当前记录的第 n 个字段，字段分隔符由 FS 变量定义
# ARGIND          命令行中当前文件的位置，从 0 开始
# CONVFMT         数字转换格式，默认 %.6g
# ENVIRON         环境变量数组，字典类型的关联数组
# ERRNO           最后一个系统错误的描述
# FIELDWIDTHS     定义字段宽度的列表，默认为空格分隔 => FIELDWIDTHS="4 2 2 2 2 2" 表示$1宽度是4、$2是2、$3是2 ....
# FILENAME        当前文件名
# IGNORECASE      非 0 值则为真，匹配时是否忽略大小写
# FS              字段分隔符，默认是任何空格
# RS              一条记录的分隔符，默认 \n
# NF              当前记录的字段数，列数
# NR              当前行号，亦代表当前针对所有文件已处理的累计总行数
# FNR             与 NR 类似，但它是针对当前非第一个文件中已处理的行数，所以仅当处理第一个文件时满足 NR==FNR
# OFMT            数字的输出格式，默认 %.6g
# OFS             输出字段分隔符，默认是空格
# ORS             输出记录分隔符，默认是 \n
# RLENGTH         由 match 函数所匹配的字符串长度
# RSTART          由 match 函数所匹配的字符串的第一个位置
# SUBSEP          数组中下标的分隔符（默认 /034)
# ...

# —————————————————————————————————————————————————————————————————————————————————————————— 内建函数

# sub(r,s)        相当于 sed 's/r/s/'
# substr(s,p)     返回字符串 s 中从 p 开始的后缀部分
# substr(s,p,n)   返回字符串 s 中从 p 开始长度为 n 的后缀部分，例如 substr($1,1,5) 表示 取第一个字段的第1-5个字符
# gsub(r,s)       在整个 $0 中用 r 替代 s，相当于 sed 's///g'
# gsub(r,s,t)     在整个 t 中用 r 替代 s
# index(s,t)      返回 s 中字符串 t 的第一位置
# length(s)       返回 s 长度
# match(s,r)      测试 s 是否包含匹配 r 的字符串
# split(s,a,FS)   以 FS 为分隔符将字符串 s 分成序列 a
# sprint(fmt,exp) 返回经 fmt 格式化后的 exp
# toupper()       大写转换
# tolower()       小写转换
# systime()       返回从1970年1月1日到当前时间（不计闰年）的秒数
# strftime()      使用 C 库中的 strftime 函数格式化时间
# asort(a,b)      对数组 a 的值排序并存入数组 b，但排序后的下标改为从 1 到数组的长度
# asorti(a,b)     对数组 a 的索引排序，并把排序后的下标存入新生成的数组 b 中
# delete(a[i])    删除数组 a 中的元素
# int()           返回截断至整数的值
# rand()          大于等于 0 且小于 1 的随机数
# strtonum(str)   将字符串转为数字
# close(...)      关闭由 print、printf 语句打开的或调用 getline 函数打开的文件或管道，如果打算写文件，并稍后在同一程序中读取，则 close 是必需的
# ...

# ——————————————————————————————————————————————————————————————————————————————————————————

# 变量不需要事先声明，变量值可以有数字和/或字符串，会自动根据上下文的不同而呈现出数字或字符串
# 数组可以用一个以上的下标来建立索引，从而实现多维数组，但本质上还是一维

# 假设当前输入记录为 "1|2|3|4|"，FS、OFS 都为 | 则此时字段数为 4（NF为4）
# 如果执行 $10 = "abc" 则会创建 $5、$6、$7、$8、$9、$10 变量，其中除了 $10 为指定的值 "abc" 以外其它变量 $5 到 $9 都是空串
# 并且 NF 自动改为 10，$0 也被改为从 $1 到 $10 对应的值 1|2|3|4||||||abc

# exit 语句执行时首先（按照在代码中出现的顺序）调用所有 END 模式的操作，然后以 Expression 参数指定的退出码来终止 awk，如果 exit 出现在 END{} 中则不调用后续操作
# next 语句停止处理当前记录，继续处理下一个记录
# ——————————————————————————————————————————————————————————————————————————————————————————

# 可以使用 SHELL 重定向符实现输出重定向，下例若第1个域的值等于 100 则将其追加到文件中
awk '$1=100{print $1 > "output_file"}'

# 文件读入可放于前面
< filename awk '{print NR}'

# 求和
awk '{sum+=$1}END{print sum}'

# 平均
awk '{sum+=$1}END{print sum/NR}'

# 最大值
awk 'BEGIN{max=0}{if($1>max){max=$1}}END{print max}'

# 最小值
awk 'BEGIN {min=9999999} {if($1<min){min=$1}}END{print min}'

# 随机数
awk 'BEGIN{srand();R=int(100*rand());print R;}'

# 传递变量给 awk
varName=123 && awk -vX=${varName} 'BEGIN{print X}'

# printf是bash命令，也是awk函数，其风格与 C 相同
awk 'BEGIN{printf("%8.3f %8.3f %8.3f",a,b,c)}'

# 遍历外部数组
awk -f 'BEGIN{for(i=1;i<ARGC;i++)print ARGV[i]}' ${extarr[@]}

# 根据第 6 列进行去重操作
awk '!arr[$6]++' file

# 计算次数
awk '{name[$0]+=$1};END{for(i in name) print i,name[i]}'

# 输出奇数行
awk 'NR%2' file
awk 'i=!i' file

# 若两域相加大于 100 则输出
awk '$1+$2<100'

# 输出匹配行的下一行
seq 10 | awk '/4/{getline;{print $0}}'

# 每三行添加一个换行符或内容
awk '$0;NR%3==0{printf "\n"}'
awk '{print NR%3?$0:$0"\n"}'
sed '4~3s/^/\n/'

# 判断
awk '{print ($1>$2)?"第1排"$1:"第2排"$2}'
awk '{max=($1>$2)?$1:$2; print max}'
awk '{if($6>50){count++;print $3}else{x+5;print $2}}'

# 判断
awk '{
    gsub(/\$/,"");gsub(/,/,"");
    if ($4>1000&&$4<2000) c1+=$4;
    else if ($4>2000&&$4<3000) c2+=$4;
    else if ($4>3000&&$4<4000) c3+=$4;
    else c4+=$4; }
    END{printf "c1=[%d];c2=[%d];c3=[%d];c4=[%d]\n",c1,c2,c3,c4
}"'

# 循环
awk '{i=1;while(i<=NF){print NF,$i;i++}}'
awk '{for(i=1;i<=NF;i++){print NF,$i}}'

# 按第2和第3列，删除重复的，仅保留一行
awk '!arr[$2$3]++'

# 输出两个文件中第1列相同的行
awk -F',' 'NR==FNR{a[$1]=$0;next}NR>FNR{if($1 in a)print $0"\n"a[$1]}' 1.log 2.log

# 可以指定多个列分隔符，支持正则
awk -F'[:|]'  '{print $2}'
awk -F'[ ]+'  '{print $9}'
awk -F'[ :]+' '{print $NF"\t"$(NF-2)}'

# 按某个位置的字符分隔的方法
awk -F":" '{for(i=1;i<=3;i++)printf("%s:",$i)}'
awk -F':' '{print $1 ":" $2 ":" $3; print $4}'
awk -F':' '{print $1 ":" $2 ":" $3; for(i=1;i<=3;i++)$i=""; print}'

# 字符串分割
awk 'BEGIN{info="this is a test";split(info,tA," ");print length(tA);for(k in tA){print k,tA[k];}}'

# 执行命令并判断返回状态
awk 'BEGIN{if(system("grep root /etc/passwd &>/dev/null")==0)print"yes!";}'

# 将多行转多列
awk '{for(i=1;i<=NF;i++) a[i,NR]=$i } END{ for(i=1;i<=NF;i++) {for(j=1;j<=NR;j++) printf a[i,j] " ";print ""}}'

# 统计连接个数
netstat -an|awk -v A=$IP -v B=$PORT 'BEGIN{print "Clients\tGuest_ip"}$4~A":"B{split($5,ip,":");a[ip[1]]++}END{for(i in a)print a[i]"\t"i|"sort -nr"}'

# 两文件匹配
awk 'BEGIN{printf "what is your name?";getline name < "/dev/tty" } $1 ~name {print "FOUND" name " on line ", NR "."} END{print "see you," name "."}' file

# break
for(x=3;x<=NF;x++){
    if ($x<0) {print "Bottomed out!"; break}
}

# continue
for(x=3;x<=NF;x++){
    if ($x==0) {print "Get next item"; continue}
}

# 小于60直接跳过，不做处理
{
    if ($4<60)
        next
    print $0
}

# 函数可以接受以逗号分隔的多个参数，参数不是强制性的，另外自定义函数不需要放在调用者之前，它可以放在任何地方，如果有返回值也直接使用 return
# function function_name(argument1, argument2, ...)
# {
#     body ...
# }

# 定义可传值函数
awk '
function fmt(FMT) {
    Nfmt=FMT
    gsub(/[0-9]+%/, "%", FMT)
    gsub(/%.*[a-zA-Z]/, "", Nfmt)
    for(i=1; i<=Nfmt; i++) txt=txt FMT
    return txt
}'

# 通过 exit 实现达到某条件时退出，但仍执行 END 操作
awk '{gsub(/\$/,"");gsub(/,/,""); if($4>3000&&$4<4000){exit}else{c4+=$4}};END{printf "c1=[%d];c2=[%d];c3=[%d];c4=[%d]\n",c1,c2,c3,c4}"'

# 通过 next 实现达到某条件时跳过该行，对下一行执行操作
awk '{gsub(/\$/,"");if($4>3000){next}else{c4+=$4}} END{printf "c4=[%d]\n",c4}"'

# 把三个文件的内容全部写到 fileall
awk '{print FILENAME,$0}' file1 file2 file3 > fileall
# 把合并后的文件重新分拆为三个文件，内容与原文件一致
awk ' $1!=previous {close(previous);previous=$1} {print substr($0,index($0," ") +1)>$1}' fileall

# 通过管道把命令执行结果送给 getline 并赋给变量 d 然后打印
awk 'BEGIN{"date"|getline d; print d}'

# 通过 getline 交互输入 name 并显示
awk 'BEGIN{system("echo \"Input your name:\\c\""); getline d;print "\nYour name is",d,"\b!\n"}'

# 输出包含 050x_ 的用户名
awk 'BEGIN{FS=":";while(getline < "/etc/passwd" > 0){if($1~"050[0-9]_") print $1}}'

# 显示文件的全路径
type file | awk -F "/" '{for(i=1;i<NF;i++) {if(i==NF-1){ printf "%s",$i } else { printf "%s/",$i }}}'

# 用空字符串替换 $ 符号和 , 符号再将结果输出到文件
awk 'gsub(/\$/,"");gsub(/,/,""); cost+=$4; END{print "The total is $" cost > "filename"}'

# 取得文件第1个域的最大值
awk 'BEGIN {max=100;print "max=" max}{max=($1>max?$1:max);print $1,"Now max is "max}'

# syntax1?syntax2:syntax3
if (syntax1)
    syntax2
else
    syntax3

# 显示日期
awk 'BEGIN {
    for(j=1;j<=12;j++)
    {
        flag=0;
        printf "\n%d月份\n",j;
        for(i=1;i<=31;i++)
        {
            if (j==2&&i>28) flag=1;
            if ((j==4||j==6||j==9||j==11)&&i>30) flag=1;
            if (flag==0) {printf "%02d%02d ",j,i}
        }
    }
}'

# ——————————————————————————————————————————————————————————————————————————————————————————

awk '/101/'                                         # 输出包含101的行
awk '/101/,/105/'                                   # 101~105行
awk '/root/,/mysql/'                                #
awk '$1>=5&&$2=="good"'                             #
awk '$1*$2>100'
awk '$2>5&&$2<=15'
awk '$1 !~ /ly$/'                                   # 显示所有第一个字段不以 ly 结尾的行
awk '$3 < 40'                                       # 如果第三个字段值小于 40 才输出
awk '/tom/,/suz/'                                   # 输出 tom 到 suz 之间的所有行
awk '{print NR" "$0}'                               # 加行号
awk '{$1=$3=""}1'                                   # 输出除第 1 和第 3 列之外的所有列
awk '{print NR,NF,$1,$NF,}'                         # 显示当前记录号、域数和每行的第一个和最后一个域
awk '/101/{print $1,$2+10}'                         # 显示匹配行的第一、二个域加10
awk '/101/{print $1 $2}'                            # 显示匹配行的第一、二个域，但显示时域间没有分隔符
df | awk '$4>1000000 '                              # 通过管道符读取数据，输出第4个域满足条件的行
seq 5 | awk 'NR==2{sub('/.*/',"txt\n&")}{print}'    # 在指定行的前后加一行
awk 'BEGIN{FS="[:\t|]"} {print $1,$2,$3}'           # 设置输入分隔符 FS
awk -F '[ :\t|]' '{print $1}'                       # 按照正则表达式的值做为分隔符，这里代表空格、:、TAB、|同时做为分隔符
awk -f awkfile file                                 # 读取 awkfile 文件中的代码进行控制
awk '$1~/101/{print $1}'                            # 输出第一个域匹配101的行（记录）
awk 'BEGIN{OFS="%"}{print $1,$2}'                   # 通过设置输出分隔符 OFS 的值来修改输出格式
awk '{$1=='Chi'{$3='China';print}'                  # 找到匹配行后先将第3个域替换后再显示该行（记录）
awk 'BEGIN{ while( "ls" | getline) print }'
awk '{ print $1, $2 | "sort" }'
awk 'NR==1{s=$0;next}{print s,$0}'                              # 把第一行内容放到每行的前面
awk -F: '$1!~/mail/{print $1}' /etc/passwd                      # 不匹配
awk -F, 'NF=3'                                                  # 以逗号做分隔，只输出前3列的字段（重要）
awk -F: 'NF==4' /etc/passwd                                     # 只显示有4个字段的行
awk '$1 = 100 {print $1 >> "output_file" }'                     # 使用重定向符进行重定向输出
awk '/keyword/{a=NR+2}a==NR{print}'                             # 取关键字下第几行
awk 'gsub(/liu/,"aaaa",$1){print $0}'                           # 只打印匹配替换后的行
awk '{$1="";$2="";$3="";print}'                                 # 去掉前三列
echo aada:aaba | awk '/d/&&/b/{print}'                          # 同时匹配两条件
echo Ma asdas  | awk '$1~/^[a-Z][a-Z]$/{print }'                # 第一个域匹配正则
awk 'length($1)=="4"{print $1}'                                 # 字符串位数
awk '{if($2>3){system("touch "$1)}}'                            # 执行系统命令
awk '{sub(/Mac/,"Macintosh",$0);print}'                         # 用 Macintosh 替换 Mac
awk '{gsub(/Mac/,"MacIntosh",$1); print}'                       # 第一个域内用 Macintosh 替换 Mac
awk '{if(NR==3)F=1}{if(F){i++;if(i%7==1)print}}'                # 从第3行开始每7行显示一次
awk '{if(NF<1){print i;i=0} else {i++;print $0}}'               # 显示空行分割各段的行数
awk '{b[$1]=b[$1]$2}END{for(i in b){print i,b[i]}}'             # 列叠加
awk '{b=a;a=$1; if(NR>1){print a-b}}'                           # 当前行减上一行
awk '{a[NR]=$1}END{for (i=1;i<=NR;i++){print a[i]-a[i-1]}}'     # 当前行减上一行
awk 'BEGIN{ "date" | getline d; split(d,mon) ; print mon[2]}'   # 将d设为数组mon，打印数组中第2个元素
awk 'BEGIN{for(n=0;n++<9;){for(i=0;i++<n;)printf i"x"n"="i*n" ";print ""}}'     # 乘法口诀

```

### cksum

```shell
# 输出文件的 CRC 校验和和字节计数（使用 CRC 算法）
cksum /etc/passwd
# 3512668765 995 /etc/passwd
```