# Linux 文本三剑客


## Linux 文本三剑客

1. awk、grep、sed 是 linux 操作文本的三大利器，合称文本三剑客，也是必须掌握的 linux 命令之一。
2. 三者的功能都是处理文本，但侧重点各不相同，其中属 awk 功能最强大，但也最复杂。
3. grep 更适合单纯的查找或匹配文本。
4. sed 更适合编辑匹配到的文本。
5. awk 更适合格式化文本，对文本进行较复杂格式处理。

## grep

### 介绍

1. grep 全称是 Global Regular Expression Print。grep 命令是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来（匹配到的标红）。
2. egrep = grep -E：扩展的正则表达式 （除了\< , \> , \b 使用其他正则都可以去掉\）
3. 命令格式：grep [option] pattern file...

### 命令参数

| 参数             | 释义                                                     |
| ---------------- | -------------------------------------------------------- |
| -A<显示行数>     | 除了显示符合范本样式的那一列之外，并显示该行之后的内容。 |
| -B<显示行数>     | 除了显示符合样式的那一行之外，并显示该行之前的内容。     |
| -C<显示行数>     | 除了显示符合样式的那一行之外，并显示该行之前后的内容。   |
| -c               | 统计匹配的行数                                           |
| -e               | 实现多个选项间的逻辑 or 关系                             |
| -E               | 扩展的正则表达式                                         |
| -f FILE          | 从 FILE 获取 PATTERN 匹配                                |
| -F               | 相当于 fgrep，搜索字符串而不会匹配正则                   |
| -i --ignore-case | 忽略字符大小写的差别。                                   |
| -n               | 显示匹配的行号                                           |
| -o               | 仅显示匹配到的字符串                                     |
| -q               | 静默模式，不输出任何信息                                 |
| -s               | 不显示错误信息。                                         |
| -v               | 显示不被 pattern 匹配到的行，相当于[^] 反向匹配          |
| -w               | 匹配 整个单词                                            |

### grep 实战演示

```Shell
[root@along ~]# cat test
aaa
bbbbb
AAAaaa
BBBBASDABBDA
[root@along ~]# grep -A2 b test
bbbbb
AAAaaa
BBBBASDABBDA
[root@along ~]# grep -B1 b test
aaa
bbbbb
[root@along ~]# grep -C1 b test
aaa
bbbbb
AAAaaa
[root@along ~]# grep -c b test
2
[root@along ~]# grep -e AAA -e bbb test
bbbbb
AAAaaa
[root@along ~]# grep -in b test
2:bbbbb
4:BBBBASDABBDA
[root@along ~]# grep -o ASDA test1
ASDA
[root@along ~]# grep -q aa test1
[root@along ~]# grep -v aaa test1
bbbbb
BBBBASDABBDA
[root@along ~]# grep -w aaa test1
aaa
[root@along ~]# cat grep.txt
aaa
[root@along ~]# grep -f grep.txt test
aaa
AAAaaa
```

## sed

### 介绍

1.  sed 英文全称是 stream editor，是一种流编辑器，它一次处理一行内容。
2.  处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（patternspace ），接着用 sed 命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。
3.  除非使用重定向存储输出或-i，文件内容才会改变，这个参数可以用来操作文件。
4.  命令格式：sed [options] '[地址定界] command' file(s)

### 常用选项 options

| 参数   | 释义                                                                            |
| ------ | ------------------------------------------------------------------------------- |
| -n     | 不输出模式空间内容到屏幕，即不自动打印，只打印匹配到的行                        |
| -e     | 多点编辑，对每行处理时，可以有多个 Script                                       |
| -f     | 把 Script 写到文件当中，在执行 sed 时-f 指定文件路径，如果是多个 Script，换行写 |
| -r     | 支持扩展的正则表达式                                                            |
| -i     | 直接将处理的结果写入文件                                                        |
| -i.bak | 在将处理的结果写入文件之前备份一份                                              |

### 常用选项 options 演示

```Shell
[root@along ~]# cat demo
aaa
bbbb
AABBCCDD
[root@along ~]# sed "/aaa/p" demo  #匹配到的行会打印一遍，不匹配的行也会打印
aaa
aaa
bbbb
AABBCCDD
[root@along ~]# sed -n "/aaa/p" demo  #-n不显示没匹配的行
aaa
[root@along ~]# sed -e "s/a/A/" -e "s/b/B/" demo  #-e多点编辑
Aaa
Bbbb
AABBCCDD
[root@along ~]# cat sedscript.txt
s/A/a/g
[root@along ~]# sed -f sedscript.txt demo  #-f使用文件处理
aaa
bbbb
aaBBCCDD
[root@along ~]# sed -i.bak "s/a/A/g" demo  #-i直接对文件进行处理
[root@along ~]# cat demo
AAA
bbbb
AABBCCDD
[root@along ~]# cat demo.bak
aaa
bbbb
AABBCCDD
```

### 地址定界

| 地址          | 释义                                                                         |
| ------------- | ---------------------------------------------------------------------------- |
| 不给地址      | 对全文处理                                                                   |
| #             | 指定的行                                                                     |
| /pattern/     | 被此处模式所能够匹配到的每一行                                               |
| #,#           | 从#行，到#行                                                                 |
| #,+n          | 从#行开始，一直到向下的 n 行                                                 |
| /pat1/,/pat2/ | 从第一次被 pat1 匹配到的行开始，到第一次被 pat2 匹配到的行结束，中间的所有行 |
| #,/pat1/      | 从#行开始，到第一次被 pat1 匹配到的行结束，中间的所有行                      |
| ~             | 步进，指定起始行及步长                                                       |
| 1~2           | 所有奇数行                                                                   |
| 2~2           | 所有偶数行                                                                   |

### 地址界定演示

```Shell
[root@along ~]# cat demo
aaa
bbbb
AABBCCDD
[root@along ~]# sed -n "p" demo  #不指定行，打印全文
aaa
bbbb
AABBCCDD
[root@along ~]# sed "2s/b/B/g" demo  #替换第2行的b->B
aaa
BBBB
AABBCCDD
[root@along ~]# sed -n "/aaa/p" demo
aaa
[root@along ~]# sed -n "1,2p" demo  #打印1-2行
aaa
bbbb
[root@along ~]# sed -n "/aaa/,/DD/p" demo
aaa
bbbb
AABBCCDD
[root@along ~]# sed -n "2,/DD/p" demo
bbbb
AABBCCDD
[root@along ~]# sed "1~2s/[aA]/E/g" demo  #将奇数行的a或A替换为E
EEE
bbbb
EEBBCCDD
```

### 编辑命令 command

| 参数 | 释义                                                                |
| ---- | ------------------------------------------------------------------- |
| d    | 删除模式空间匹配的行，并立即启用下一轮循环                          |
| p    | 打印当前模式空间内容，追加到默认输出之后                            |
| a    | 在指定行后面追加文本，支持使用\n 实现多行追加                       |
| i    | 在行前面插入文本，支持使用\n 实现多行追加                           |
| c    | 替换行为单行或多行文本，支持使用\n 实现多行追加                     |
| w    | 保存模式匹配的行至指定文件                                          |
| r    | 读取指定文件的文本至模式空间中匹配到的行后                          |
| =    | 为模式空间中的行打印行号                                            |
| !    | 模式空间中匹配行取反处理                                            |
| s/// | 查找替换，支持使用其它分隔符，如：s@@@，s###。加 g 表示行内全局替换 |

### 编辑演示

```Shell
[root@along ~]# cat demo
aaa
bbbb
AABBCCDD
[root@along ~]# sed "2d" demo  #删除第2行
aaa
AABBCCDD
[root@along ~]# sed -n "2p" demo  #打印第2行
bbbb
[root@along ~]# sed "2a123" demo  #在第2行后加123
aaa
bbbb
123
AABBCCDD
[root@along ~]# sed "1i123" demo  #在第1行前加123
123
aaa
bbbb
AABBCCDD
[root@along ~]# sed "3c123\n456" demo  #替换第3行内容
aaa
bbbb
123
456
[root@along ~]# sed -n "3w/root/demo3" demo  #保存第3行的内容到demo3文件中
[root@along ~]# cat demo3
AABBCCDD
[root@along ~]# sed "1r/root/demo3" demo  #读取demo3的内容到第1行后
aaa
AABBCCDD
bbbb
AABBCCDD
[root@along ~]# sed -n "=" demo  #=打印行号
1
2
3
[root@along ~]# sed -n '2!p' demo  #打印除了第2行的内容
aaa
AABBCCDD
[root@along ~]# sed 's@[a-z]@\u&@g' demo  #将全文的小写字母替换为大写字母
AAA
BBBB
AABBCCDD
```

## awk

### 介绍

1. awk 是 Aho Weinberger kernaighan 三个人的首字母缩写
2. awk 是一种编程语言，用于在 linux/unix 下对文本和数据进行处理。数据可以来自标准输入(stdin)、一个或多个文件，或其它命令的输出。
3. 支持用户自定义函数和动态正则表达式等先进功能，是 linux/unix 下的一个强大编程工具。
4. 可以在命令行中使用，但更多是作为脚本来使用
5. awk 有很多内建的功能，比如数组、函数等。

### 语法

```Shell
awk [options] 'program' var=value file…
awk [options] -f programfile var=value file…
awk [options] 'BEGIN{ action;… } pattern{ action;… } END{ action;… }' file ...
```

### 常用命令选项

| 选项         | 释义                                                  |
| ------------ | ----------------------------------------------------- |
| -F fs        | fs 指定输入分隔符，fs 可以是字符串或正则表达式，如-F: |
| -v var=value | 赋值一个用户定义变量，将外部变量传递给 awk            |
| -f scripfile | 从脚本文件中读取 awk 命令                             |

### 内置变量

| 内置变量 | 释义                                                                              |
| -------- | --------------------------------------------------------------------------------- |
| FS       | 输入字段分隔符，默认为空白字符                                                    |
| OFS      | 输出字段分隔符，默认为空白字符                                                    |
| RS       | 输入记录分隔符，指定输入时的换行符，原换行符仍有效                                |
| ORS      | 输出记录分隔符，输出时用指定符号代替换行符                                        |
| NF       | 字段数量，共有多少字段， $NF引用最后一列，$(NF-1)引用倒数第 2 列                  |
| NR       | 行号，后可跟多个文件，第二个文件行号继续从第一个文件最后行号开始                  |
| FNR      | 各文件分别计数, 行号，后跟一个文件和 NR 一样，跟多个文件，第二个文件行号从 1 开始 |
| FILENAME | 当前文件名                                                                        |
| ARGC     | 命令行参数的个数                                                                  |
| ARGV     | 数组，保存的是命令行所给定的各参数，查看参数                                      |

### 实战演示

```Shell
[root@along ~]# cat awkdemo
hello:world
linux:redhat:lalala:hahaha
along:love:youou
[root@along ~]# awk -v FS=':' '{print $1,$2}' awkdemo  #FS指定输入分隔符
hello world
linux redhat
along love
[root@along ~]# awk -v FS=':' -v OFS='---' '{print $1,$2}' awkdemo  #OFS指定输出分隔符
hello---world
linux---redhat
along---love
[root@along ~]# awk -v RS=':' '{print $1,$2}' awkdemo
hello
world linux
redhat
lalala
hahaha along
love
you
[root@along ~]# awk -v FS=':' -v ORS='---' '{print $1,$2}' awkdemo
hello world---linux redhat---along love---
[root@along ~]# awk -F: '{print NF}' awkdemo
2
4
3
[root@along ~]# awk -F: '{print $(NF-1)}' awkdemo  #显示倒数第2列
hello
lalala
love
[root@along ~]# awk '{print NR}' awkdemo awkdemo1
1
2
3
4
5
[root@along ~]# awk 'END {print NR}' awkdemo awkdemo1
5
[root@along ~]# awk '{print FNR}' awkdemo awkdemo1
1
2
3
1
2
[root@along ~]# awk '{print FILENAME}' awkdemo
awkdemo
awkdemo
awkdemo
[root@along ~]# awk 'BEGIN {print ARGC}' awkdemo awkdemo1
3
[root@along ~]# awk 'BEGIN {print ARGV[0]}' awkdemo awkdemo1
awk
[root@along ~]# awk 'BEGIN {print ARGV[1]}' awkdemo awkdemo1
awkdemo
[root@along ~]# awk 'BEGIN {print ARGV[2]}' awkdemo awkdemo1
awkdemo1
```

### 自定义变量

```Shell
[root@along ~]# awk -v name="along" -F: '{print name":"$0}' awkdemo
along:hello:world
along:linux:redhat:lalala:hahaha
along:along:love:you
```

### 统计分析

#### where 条件过滤

```Shell
# select * from table;
awk 1 table_file
# select * from table where cost > 100;
awk '$2>100' table_file
```

#### 对某个字段去重，或者按记录去重

```Shell
# select distinct(date) from table;
awk '!a[$3]++{print $3}' table_file
# select distinct(*) from table;
awk '!a[$0]++' table_file
```

#### 记录按序输出

```Shell
# select id from table order by id;
awk '{a[$1]}END{asorti(a);for(i=1;i<=length(a);i++){print a[i]}}' table_file
```

#### 取前多少条记录

```Shell
# select * from table limit 2;
awk 'NR<=2' table_file
awk 'NR>2{exit}1' table_file # performance is better
```

#### 分组求和统计，关键词：group by、having、sum、count

```Shell
# select id, count(1), sum(cost) from table group by id having count(1) > 2;
awk '{a[$1]=a[$1]==""?$2:a[$1]","$2}END{for(i in a){c=split(a[i],b,",");if(c>2){sum=0;for(j in b){sum+=b[j]};print i"\t"c"\t"sum}}}' table_file
```

#### 模糊查询

```Shell
# select name from table where name like 'wang%';
awk '$2 ~/^wang/{print $2}' table_file
# select addr from table where addr like '%bei';
awk '/.*bei$/{print $3}' table_file
# select addr from table where addr like '%bei%';
awk '$3 ~/bei/{print $3}' table_file
```

#### 多表 join 关联查询

```Shell
# select a._ , b._ from table a inner join table b on a.id = b.id and b.id = 2;
awk 'ARGIND==1{a[$1]=$0;next}{if(($1 in a)&&$1==2){print a[$1]"\t"$2"\t"$3}}' table_file table_file
```

#### 多表 union all

```Shell
# select a._ from table a union all select b._ from table b;
awk 1 table_file table_file
# select a._ from table a union select b._ from table b;
awk '!a[$0]++' table_file table_file
```

#### 随机抽样统计，关键词：order by rand()

```Shell
# select * FROM table ORDER BY RAND() LIMIT 2;
awk 'BEGIN{srand();while(i<2){k=int(rand()*10)+1;if(!(k in a)){a[k];i++}}}(NR in a)' table_file
```

- awk 还支持 if else while for do while 自定义函数.awk 脚本.awk

