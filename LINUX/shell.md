# shell 脚本

## skill 

- `#!`决定脚本采用的解释器 `#! /bin/bash` 
- 定义变量以及使用变量
    + `your_name="hello"` 等号左右两侧不能存在空格
    + `echo $your_name` 使用变量必须在其前面加上$符号
- 添加单行注释 `# xxx`
- 添加多行注释 
```shell
:<<'
注释内容xxx
注释内容xxx
注释内容xxx'
```

- 脚本的执行参数自动存储到0, 1, 2, 3, ...,n变量下, 访问的话可以用$0, $1 `echo "第一个参数: $0"; echo "第二个参数: $1";`

### 数据类型(字符串)

- 单引号中间无法使用变量, 而双引号可以
- 获取字符串长度 `len=${#your_name}`
- 子字符串的获取 `substr=${your_name:0:2}` 从第0个字符开始截取2个字符

### 数据类型(字符串)

- 仅仅支持一维数组,元素之间仅仅有一个空格作为分隔符 `array=("english" "math" 2)`
- 数组的访问通过下标来实现 `array[0]="chinese"`
- 数组的打印 `echo ${array[2]}`
- 获取数组的长度 `echo ${#array[*]}` 或者 `echo ${#array[@]}`

### 数据类型()


### shell中的括号

- 小括号 TODO: 中午补充完整



### 引号

- 双引号可以转义`$` 
- 单引号可以转义`(反引号)`

## shell 常用变量 

- `$#` 添加到shell的参数个数
- `$1` - `$n` 所有参数
- `$0` shell文件名

## 比较

- -eq equation
- -lt less than 
- -le less or equation
- -gt greater than
- -ge greater or equation
- -f 被检测文件是一个regular文件
- -e 文件存在

## 产生数组

- `{start..end..incr}` eg. `{1..10..2}` == 1 3 5 7 9

## 常用命令

- awk shell文本分析语言
    + awk -F'\t' '{print $5}' test.txt # 打印所有行第5列
    + awk -F'\t' 'END {print}' test.txt # 打印最后一行
    + awk -F'\t' 'NR == 1 {print}' test.txt # 打印第一行
    + awk -F'\t' 'sum[$5]++END{for(i in sum) print i "\t" sum[i]}' test.txt # 统计第5列中所有不同元素出现的次数
    + `~ ~!` 匹配正则表达式和不匹配正则表达式
    + awk 分为三个部分 BEGIN{(只在开头执行一次)}, {(循环每行主体)}, END{(只在最后执行一次)}
- sort 同c++函数sort
- uniq 去除相同元素, 仅仅保留独立元素, 但是仅仅只对相邻元素进行相同元素的判定


## Ref
[shell脚本教程](http://www.runoob.com/linux/linux-shell.html)
[expr命令使用技巧](https://blog.csdn.net/adcxf/article/details/3001275)
[shell中的括号](https://blog.csdn.net/tttyd/artice/details/11742241)
[awk命令](https://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html)
[awk运算符](https://www.cnblogs.com/chengmo/archive/2010/10/11/1847515.html)
