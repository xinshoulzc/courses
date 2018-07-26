# shell 脚本

## skill 

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
- sort 同c++函数sort
- uniq 去除相同元素, 仅仅保留独立元素, 但是仅仅只对相邻元素进行相同元素的判定

## Ref

[shell中的括号](https://blog.csdn.net/tttyd/artice/details/11742241)
[awk命令](https://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html)
