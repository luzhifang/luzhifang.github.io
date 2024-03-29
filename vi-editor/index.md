# vi编辑器


## 1. vi 和 vim 的区别

1. vim 是进阶版的 vi
2. vim 不但可以用不同颜色显示文字内容
3. vim 能够进行如 shell script, C program 等程序编辑功能， 你可以将 vim 作为一种程序编辑器

## 2. vi 编辑器的三种模式

### 2.1. 一般指令模式

1. 使用 `vi[m] filename` 进入一般指令模式
2. 在这个模式中， 你可以使用“上下左右”按键来移动光标，你可以使用“删除字符”或“删除整列”来处理文件内容， 也可以使用“复制、贴上”来处理你的文件数据。
3. 快捷键
   1. h(←) j(↓) k(↑) l(→)，n+[hjkl↑↓←→]，n+Enter 向下移动 n 行
   2. Ctrl+f 向下一页，Ctrl+b 向上一页，Ctrl+d 向下半页，Ctrl+u 向上半页
   3. n+space 光标处向后移动 n 个字符
   4. 0/Home 键移至行首，$/End 键移至行尾
   5. H 移至屏幕第一行第一个字符，M 移至屏幕中央，L 移至屏幕最后一行
   6. G 移至文件最后一行，nG 移至文件第 n 行，gg 移至文件第一行，相当于 1G
   7. n+backspace 键退格几个字符
   8. n+dd 删除 n 行，d1G 删除光标至第一行，dG 删除光标至最后一行，d$删除光标至行尾，d0 删除光标至行首
   9. n+yy 向下复制 n 行，y1G 复制光标至第一行，yG 复制光标至最后一行，y$复制光标至行尾，y0 复制光标至行首
   10. p 光标处下一行或下一个字符粘贴，P 光标处上一行或上一个字符粘贴
   11. J 将光标所在列与下一列的数据结合成同一列
   12. u 撤销，Ctrl+r 恢复，.(小数点)重复前一个动作

### 2.2 编辑模式

1. i 光标处插入，I 该行第一个非空白字符处插入
2. a 光标所在的下一个字符处插入，A 该行最后一个字符处插入
3. o 光标处向下插入空行，O 光标处向上插入空行
4. r 只会取代光标处的那一个字符，R 会一直取代光标所在的文字，直到按下 ESC 为止
5. ESC 退出编辑模式

### 2.3. 命令行命令模式

1. 在一般模式中，输入[: / ? ]三个中的任何一个按钮，就可以将光标移动到最下面那一列
2. 提供查找、存盘、替换、离开 vi 、显示行号等等的
3. 快捷键
   1. /word 光标之下查找，?word 光标之上查找。n 重复前一个查找，N 与 n 相反
   2. :n1,n2s/word1/word2/g 替换 n1 与 n2 行之间的 word1 为 word2
   3. :1,$s/word1/word2/g 替换所有行
   4. :1,$s/word1/word2/gc 二次确认替换
   5. :w 写入，:w!强制写入
   6. :q 退出，:q!强制退出
   7. :wq 保存后离开，:wq!强制保存后离开
   8. ZZ 文件无修改不保存退出，文件有修改保存后退出
   9. :w filename 另存为，:r filename 读入文件数据
   10. :n1,n2 w filename 将 n1 到 n2 行另存为
   11. :! [shell command] 暂时离开，可接命令
   12. :set nu 显示行号，:set nonu 取消行号

## 3. 区块选择

1. v 字符选择，V 行选择
2. Ctrl+v 方块选择
3. y 将选中地方复制
4. d 将选中地方删除
5. p 将选中地方在光标处粘贴

## 4. 多窗口编辑

1. :sp [filename] 打开一个新窗口
2. Ctrl+w+[j/↓]移动到下一个窗口
3. Ctrl+w+[k/↑]移动到下一个窗口
4. Ctrl+w+q 关闭当前窗口，同:q

## 5. 多文件编辑

1. vi[m] filename1 filename2… 编辑多个文件
2. :n 编辑下一个文件，:N 编辑上一个文件
3. :files 列出所有编辑的文件

