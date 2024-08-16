---
title: shell-variables
layout: default
parent: Linux
has_children: false
---

### 判断操作

```bash

# 二元比较用于配合 test、[ ]、[[ ]] 进行测试 ...

# 算数比较
-eq     # 等于
-ne     # 不等于
-lt     # 小于
-le     # 小于等于
-gt     # 大于
-ge     # 大于等于

# 算数比较，针对双括号 (( ... )) 结构
>       # 大于
>=      # 大于等于
<       # 小于
<=      # 小于等于
==      # 等于

# 字串比较: 若在 [[ ... ]] 测试则不需要转义
==      # 等于
!=      # 不等于
\<      # 小于（ASCII）
\>      # 大于（ASCII）
-z      # 字符串为空
-n      # 字符串不为空

# 文件类型测试
-e          # 文件是否存在
-s          # 文件大小不为 0
-f          # 是文件
-d          # 是目录
-p          # 是管道
-S          # 是 socket
-r          # 有读权限
-w          # 有写权限
-x          # 执行权限
-h          # 符号链接
-L          # 符号链接
-b          # 块设备
-c          # 字符设备
-g          # 设置了 sgid 标记
-u          # 设置了 suid 标记
-k          # 设置了粘贴位
-t          # 文件与一个终端相关联
-N          # 从这个文件最后一次被读取之后，又被修改过
-O          # 这个文件的属主是当前用户
-G          # 文件组 id 与用户所属组相同

F1 -ef F2   # 文件 F1 和文件 F2 都是同一个文件的硬链接
F1 -nt F2   # 文件 F1 比文件 F2 新
F1 -ot F2   # 文件 F1 比文件 F2 旧

-a          # 逻辑与
-o          # 逻辑或
!           # 逻辑否（反转测试结果）

# 测试两个文件是否均可写
[ -w result.txt -a -w score.txt ] && echo OK

```

### 变量的间接引用、变量追加

```bash

aa=120
type=aa
echo ${!type}set        # 提取 type 的值再将其作为变量名，此处即获取 $aa 的值（其实就是间接引用，类似指针的味道）
# 120set

# 下例两者相等
var=$var$append
var+=$append

```

### 变量的处理

```bash

${#var}                     # 获取变量的长度
${var#str}                  # 若从头开始的数据符合 str 则将符合的最短数据切除，${var#*str} 从左开始删除第一个 str 及其左边所有
${var##str}                 # 若从头开始的数据符合 str 则将符合的最长数据切除
${var%str}                  # 若从尾向前的数据符合 str 则将符合的最短数据切除，${var%str*} 从右开始删除第一个 str 及其右边所有
${var%%str}                 # 若从尾向前的数据符合 str 则将符合的最长数据切除
${var/oldStr/newStr}        # 若符合 oldStr 则第一个旧字符串会被 newStr 替代
${var//oldStr/newStr}       # 若符合 oldStr 则全部的旧字符串会被 newStr 替代
${var:0:5}                  # 取字串索引从 0 起并向后共 5 个字符的内容（偏移）
${var:7}                    # 从字串索引第 8 个字符到最后（从第 7 位开始截取）
${var:2:-2}                 # 从字串索引第 2 位开始截取、从最后截取 2 位
${var:0-7:3}                # 从右向左第几 7 个字符起再向其右边取共 3 个字符
${var:-"value"}             # 若变量未定义或为空则临时用 value 代替，如 ${NO_DEFINE:-~} 未定义时输出为 /root
${var:="value"}             # 若变量未定义或为空则赋值为 value 如 ${username=$(date)}
${var:+"value"}             # 若变量不为空则返回 value，若为空则仍返回空值（反向判断）
${var:?"errorString"}       # 若变量未定义或为空则输出标准错误并退出
${var/old/new}              # 替换一次
${var//old/new}             # 替换全部，字符串判断 [[ $string == *My* ]] && echo ok
${!varprefix*}              # 匹配所有以 varprefix 开头的变量
${!varprefix@}              # 匹配所有以 varprefix 开头的变量
${string/#substr/replace}   # 若前缀匹配 substr 则用 replace 取代 substr
${string/%substr/replace}   # 若后缀匹配 substr 则用 replace 取代 substr
$(command)                  # 命令执行后的结果（它比 `command...` 的格式更稳定）
$((.....))                  # 算数表达式的结果
unset var                   # 销毁变量、函数

$(($RANDOM%10))             # 生成 0~9 之间的随机数
# —————————————————————————————————————————————————————————————————————————————————————————— 大小写转换

# 大小写字母转换，如果变量值的首字母匹配模式 pattern（通配符匹配，只能是一个字符，可以是 ? * [...] 或一个英文字母，多个字符不起作用）
${var^pattern}              # 将首字母转大写
${var^^pattern}             # 将所有匹配字母转大写
${var,pattern}              # 将首字母转小写
${var,,pattern}             # 将所有匹配字母转小写

var_5='hello WORLD'
var_6='HELLO world'
echo ${var_5^[a-z]}         # Hello WORLD
echo ${var_5^^*}            # HELLO WORLD
echo ${var_5^^}             # HELLO WORLD
echo ${var_6,}              # hELLO world
echo ${var_6,,[A-Z]}        # hello world

# Tips ...
# 针对数组变量使用 @、* 的情况和前述相同，大小写转换将作用于每个参数

# —————————————————————————————————————————————————————————————————————————————————————————— 进制转换 

echo $((2#0000011))         # 计算 2 进制数据的 10 进制表示，结果为 3
echo $((8#12322))           # 计算 8 进制数据的 10 进制表示，结果为 5330

```

### 环境变量、特殊变量

- [参考1](https://linux.die.net/Bash-Beginners-Guide/sect_03_02.html)
- [参考2](https://www.gnu.org/software/bash/manual/html_node/Variable-Index.html)
- [参考3](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html)

```bash

# 环境变量分 set、env 两种
# set     用来显示环境变量、私有变量，它能显示当前 SHELL 的变量（私有变量），包括当前用户的变量（环境变量）
# env     用来显示环境变量，它是当前用户的变量（环境变量）

# export 用来显示和设置环境变量 ...
# 不同类的 SHELL 有不同的私有变量 bash、ksh、csh 每种 SHELL 私有变量都不一样
# SHELL 在初始化时会在执行 /etc/profile、~/.bash_profile 等初始化脚本，它们都定义了若干环境变量，这些变量会在创建子进程时传递给子进程
# 常见的环境变量设置文件:
#   ● /etc/profile
#   ● /etc/profile.d/
#   ● ~/.profile
#   ● /etc/bashrc
#   ● ~/.bashrc
#   ● /etc/rc.local、...

# 定义环境变量
export VAR=VALUE

# 执行 env 命令查看当前环境变量，常用系统环境变量如下（其中有部分不是环境变量）
_                   # 上条命令的最后一个参数
$BASH               # 展开为调用 BASH 实例时使用的全路径名
$BASH_VERSION       # BASH 版本
$BASHPID            # BASH 进程的进程 ID
$BASHOPTS           # 当前 SHELL 的参数，可以用 shopt 命令修改
$BASH_SUBSHELL      # 记录 BASH 进程实例中多个子 SHELL "subshell" 嵌套深度的累加器，例如使用 (command...) 的场景
$SHLVL              # 记录 SHELL 的嵌套层数，启动第一个 SHELL 时值为 1，若在其中调用脚本则脚本中其值为 2，类似的还有 $BASH_SUBSHELL
$SHELLOPTS          # 已开启的 SHELL 选项，如 braceexpand、hashall、monitor 等
$SHELL              # 内部变量 SHELL 通过 BASH 的值确定当前 SHELL 类型
$HOSTNAME           # 主机名
$SECONDS            # 记录脚本从执行到结束所经过的秒数，调试时有用
$EDITOR             # 内置编辑器 emacs、gmacs、vi 的路径名
$EUID               # 在 SHELL 启动时被初始化的当前用户的 UID
$GROUPS             # 当前用户所属的 GID
$HISTTIMEFORMAT     # 定义 history 命令显示的时间格式
$HISTIGNORE         # 忽略历史中的特定命令，如 HISTIGNORE="pwd:ls:ls -ltr:"
$HISTCONTROL        # 将值设为 ignoredups 可以将连续（重复）执行的命令剔除，仅保留一条、将值设为 erasedups 可清除整个命令历史中的重复条目
$HISTFILE           # 保存命令行历史的文件，默认路径 ~/.bash_history
$HISTSIZE           # 记录在命令行历史文件中的命令数，默认 500
$HOME               # 默认为主目录，这是未指定路径时 cd 的默认路径
$IFS                # 内部字段的域分隔符，通常是空格、制表、换行，用于命令替换、循环结构、读取输入时的字段划分
$IFS                # SHELL 把 IFS 中的每个字符都作为一个分隔符，并用它们来将参数扩展的结果切分成 word
$LANG               # 用来为没有以 LC_ 开头的变量明确选取的种类确定 locale 类
$OLDPWD             # 上一个工作目录，与 cd - 的作用相同
$PATH               # 命令搜索路径，它的值是由冒号分隔的目录列表
$PPID               # 父进程 PID
$PS1                # 主提示符串，[\u@\h \W]\$ 其中的 \$ 对 root 用户是 #、对普通用户是 $
$PS2                # 次提示符串，默认值是 >，当输入的命令分为好几行时，新行前出现的字符串即为 PS2 的值，例如当在 SHELL 中运行 for 循环时
$PS3                # 与 select 命令一起使用的选择提示符串，默认是 #?
$PS4                # 当使用 bash -x 追踪时使用的调试提示符串，默认是 +
$LOGNAME            # 登录者帐号
$UID                # 当前用户的 ID
$USER               # 当前用户名
$PWD                # 当前工作目录，由 cd 设置
$RANDOM             # 随机数变量
$REPLY              # 当未给 read 提供参数时的默认接收变量，是记录多个 BASH 进程实例嵌套深度的累加器
$MACHTYPE           # 处理器架构及发行版信息，例如 x86_64-redhat-linux-gnu
$OSTYPE             # 操作系统类型，例如 linux-gnu
$SSH_CLIENT         # SSH 会话的客户端来源 IP
$SSH_CONNECTION     # SSH 的连接信息
$SSH_TTY            # 设备
$PROMPT_COMMAND     # 在显示 PS1 提示之前，该变量的文本内容作为常规的 BASH 命令执行，它是位于 /etc/bashrc 的特殊变量
$LS_OPTIONS         # 决定如何显示命令 ls 的结果，例如 export LS_OPTIONS=--color=auto（经测试不可行）
$TMOUT              # 表示 SHELL 自动终止命令之间的秒数
$EPOCHSECONDS       # 返回 1970 年以来的秒数
$PIPESTATUS         # 提供管道中每个命令的状态
$FUNCNAME           # 当前正在运行的函数名称，可以函数内使用 "echo $FUNCNAME" 来引用
$LINENO             # 返回脚本中当前的行号
$XDG_SESSION_ID
$HOSTNAME
$SELINUX_ROLE_REQUESTED
$SELINUX_USE_CURRENT_RANGE
$LD_LIBRARY_PATH
$TERM
$LS_COLORS
$rs
$MAIL
$SELINUX_LEVEL_REQUESTED
$LESSOPEN
$XDG_RUNTIME_DIR

# —————————————————————————————————————————————————————————————————————————————————————————— 特殊变量

$0                  # 当前脚本名称
$num                # num 为从 1 开始的数字，它是脚本参数的位置索引
$#                  # 传入脚本的参数的总数量
$*                  # 所有位置参数，作为单个字符串
$@                  # 所有位置参数，每个都作为独立的字符串，形成参数组成的数组
${#@}               # 脚本的命令行参数个数
$?                  # 当前进程中上个命令的返回值，常配合 if 或 &&
$$                  # 脚本自身 PID
$!                  # 后台运行的最后一个 PID
$-                  # 输出 SHELL 使用的当前选项
$_                  # 之前命令的最后一个参数
${!#}               # 获取命令行最后面的参数，因为 $# 的值是当前进程的参数个数，因此使用 ! 引用该变量的值即为最后一个参数值

```

### set、env、export

```bash

# set [+-abCdefhHklmnpPtuvx]
#   -a      将后续定义的变量或函数自动导出
#   -e      如果命令以非零状态退出，则立即退出（停止后续的脚本逻辑）
#   -f      禁用文件名扩展
#   -x      执行指令后，先显示该指令及所下的参数
#   -n      只读取指令，而不实际执行（相当于全局级别的注释）
#   -b      使被中止的后台程序立刻回报执行状态
#   -k      将所有参数当作变量赋值
#   -m      使用监视模式（启用作业控制）
#   -p	    启动特权模式
#   -v	    打印 SHELL 输入行
#   -t      执行完随后的指令，即退出 SHELL
#   -u      当执行时使用到未定义过的变量，则显示错误信息
#   +<arg>  取消某个 set 曾启动的参数

# 设置位置参数的值，可以使用 set 命令为脚本或程序中的位置参数赋值
set Ubuntu Fedora LinuxMint Debian Kubuntu
echo $3 
# LinuxMint

set -- "arg 1" "arg 2" "arg 3"
echo "$1"   # arg 1
echo "$2"   # arg 2
echo "$3"   # arg 3

# 双破折号用于取消位置参数
set --

# 可以使用 + 选项来禁用 SHELL 选项，例如 set +o nounset
# 可以使用 - 选项来启用 SHELL 选项，例如 set -o nounset

# —————————————————————————————————————————————————————————————————————————————————————————— export

# 内建 builtin 命令 export 用于把当前 SHELL 的变量、函数导出到子 SHELL，这样在子 SHELL 中就可以使用父 SHELL 中定义的变量、函数
export              # 显示当前导出的变量
export -p           # 显示当前导出的变量
export name         # 导出变量 name
export name=word    # 导出变量 name 并赋值为 word
export -n name      # 取消导出的变量 name
export -f name      # 导出函数 name
export -f           # 显示当前导出的函数
export -fn name     # 取消导出的函数 name

```

### BASH 中的常见特殊符号

```bash

# 通配扩展及代码块: {}
mkdir /home/{userA,userB,userC}/{source,data}
{command...;} && { command1;command2;....;} &> /dev/null
echo {01..10}
echo ${string##endword}

# 通配扩展及判断: []
ls /[eh][to][cm]*
if [ "$?" != 0 ] # 等价于 if test "$?" != 0

# 执行替换: `...` 与 $( ... ) 及求值运算: $(( expr... ))
result=`command...`
result=$(command...)
result=$(( 10**10 ))

# 强/弱引用
a=1
echo '$a'    # 输出 $a
echo "$a"    # 输出 1

# and/or 即 &&、||
[ condition ] && command FOR true || command FOR false
if ! revcondition; then ....... ; fi

# 特殊字符冒号作为内建空指令其返回值永远为 0，while : 与 while true 及 while 1 均可实现无限循环

```

### Tips
```bash

if [ CONDITION ]                    # 测试结构
if [[ CONDITION ]]                  # 扩展的测试结构，支持更多操作符，如 &&、||、=~、<=、>=、......
{ command1; command2; ... ; }       # 代码块
{1..100}                            # 花括号展开
>(COMMAND)                          # 进程替换 ?
<(COMMAND)                          # 进程替换 ?
( command... )                      # 在子 SHELL 中执行命令
result=$( command... )              # 在子 SHELL 中执行命令并将结果赋给变量
Array=(element1 element2 element3)  # 初始化下标类型的数组
(( var = 78 ))                      # 进行整型运算
var=$(( 20 + 5 ))                   # 进行整型运算并将结果赋值给变量
C=$A$B                              # 变量值的字串拼接，不需要空格
cd -                                # 切换到之前的目录，上一个目录
cd ~                                # 用户的主目录
command &                           # 将当前任务放到后台进行

# 当使用 let 进行时在变量名前面不能添加 $
var1=2
var2=3
let result=var1+var2
echo $result            # 输出 5
let var++               # 自加
let var--               # 自减
let var+=6              # 等同于 let var=var+6
let var-=6              # 等同于 let var=var-6

# 操作符 []、(()) 的使用和 let 类似，当使用它们时在变量名前面的 $ 可加可不加，但操作符前必须加 $
var1=2
var2=3
result=$[var1+var2]     # 也可以用 result=$[$var1+$var2]、result=$((var1+var2))
echo $result            # 输出 5

# expr 同样可用于基本算术操作，但需要注意在进行乘法运算时需要在 * 前面加上转义符 \ 否则会报错
# 除此之外表达式中的每个部分都要用空格分开

```