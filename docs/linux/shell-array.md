---
title: shell-array
layout: default
parent: Linux
has_children: false
---

### 数组的用法

```shell

# 数组是相同数据类型的元素按顺序排列的集合，把有限个类型相同的变量用一个名字命名后用编号区分
# 集合名称为数组名，编号称为下标，下标默认从 0 开始，组成数组的各个变量称为数组的元素
# 数组是在程序设计中为了处理方便，把相同类型的若干变量按有序的形式组织起来的形式

# 声明下标数组
declare -a array1
declare -a array2=( element1 element2 element3 ...)
array3=(
value1
value2
value3
...
)

# 声明关联数组
declare -A array
array[0]=one ; array[1]=two ; ...
array=(
[1]=one
[2]=two
[3]=three
...
)

# 使用命令的输出内容作为数组元素
array=($(ls -1 $HOME))

# —————————————————————————————————————————————————————————————————————————————————————————— 模拟 Maps
#!/bin/bash

declare -A maps=(
["a"]="
value1
value2
"
["b"]="
value3
value4
"
)

for key in ${!maps[@]}
{
    for value in ${maps[$key]}
    {
        echo K:$key V:$value
    }
}

# K:a V:value1
# K:a V:value2
# K:b V:value3
# K:b V:value4

# —————————————————————————————————————————————————————————————————————————————————————————— Array

${array[@]}                     # 数组元素展开
${array[*]}                     # 数组元素（字符串形式）
${!array[@]}                    # 显示所有的键（数组下标展开）
${#array[@]}                    # 数组元素个数
${#array[$index]}               # 特定元素长度
${array[@]:1}                   # 从第 1 个元素到最后
${array[@]:0:2}                 # 从第 0 个元素到第 2 个索引的元素 echo ${arry[@]:offset:number}
${array[@]#t*e}                 # 从左向右删除第一个匹配 t*e 的元素
${array[@]##t*e}                # 从左向右删除所有的匹配 t*e 的元素
${array[@]%t*e}                 # 从右向左删除第一个匹配 t*e 的元素
${array[@]%%t*e}                # 从右向左删除所有的匹配 t*e 的元素
${array[@]/x/y}                 # 替换第一个 x 字符为 y
${array[@]//x/y}                # 替换所有的 x 字符为 y
${array[@]//x/}                 # 当未指定替换的文本值时，其默认动作为删除
${array[@]/#x/y}                # 从左向右替换第一个 x 为 y
${array[@]/%x/y}                # 从右向左替换第一个 x 为 y
unset ${arry}                   # 删除数组
unset ${arry[$index]}           # 删除数组元素
arry[$index]=$((RANDOM%50+1))   # 为指定元素赋值指定范围的随机数
read -a arry                    # 交互式读入数组

# 复制数组
Unix=('Debian' 'Red hat' 'Ubuntu' 'Suse' 'Fedora' 'UTS' 'OpenLinux');
Linux=("${Unix[@]}")

# 关联数组引用变量值
declare -A maps 
value=123
ref=value
maps=([a]=1 [b]=$ref [c]=2) 
echo ${test[b]}     # value
echo ${!test[b]}    # 123
value=666
echo ${!test[b]}    # 666

# —————————————————————————————————————————————————————————————————————————————————————————— 数组元素比较

a=(1 2 3 4)
b=(4 5 6 7)
for((i=0;i<=${#a[@]};i++))
{
    n1='${a['$i']}'
    n2='${b['$i']}' 
    eval "cmp1=$n1 ; cmp2=$n2"
    (( cmp1 < cmp2 )) && echo "a[$i] < b[$i]"
}
# a[0] < b[0]
# a[1] < b[1]
# a[2] < b[2]
# a[3] < b[3]

# —————————————————————————————————————————————————————————————————————————————————————————— 遍历数组

for((i=0;i<${#array[@]};i++))
{
    echo ${array[i]}
}

# 或
for i in ${array[@]}
{
    echo $i
}

# 或
i=0
while [ $i -lt ${#array[@]} ]  
do
    echo ${array[$i]}  
    let i++  
done

```

### 求多维数组中的最大、最小值

```bash

cat <<EOF > numberfile
33   55   23   56   99
234  234  545  6546 34
11   43   534  33   75
43   34   76   756  33
343  890  77   667  55
EOF
# —————————————————————————————————————————————————————————————————————————————————————————— 
#!/bin/bash
max=0
min=999999
line=1
dnum=$(cat numberfile | wc -l)

while (($line<=$dnum))
do
    for i in $(cat numberfile | head -$line)
    do
        ((max<$i)) && max=$i
        ((min>$i)) && min=$i
    done
    let ++line
done

echo "max number: $max"
echo "min number: $min"

# 或
echo "MAX: $(cat numberfile | awk '{for(i=1;i<=NF;i++)if(max<$i){max=$i};print max}'| tail -1)"
echo "MIN: $(cat numberfile | awk '{min=999999;for(i=1;i<=NF;i++)if(min>$i){min=$i};print min }'| tail -1)
```

### 模拟字典结构

```bash

hput() {
    eval "hkey_$1"="$2"
}

hget() {
    eval echo '${'"hkey_$1"'}'
}

hput k1 aaa     # 设置字典的 KEY VALUE
hget k1         # 获取字典的 KEY 值
# aaa

```