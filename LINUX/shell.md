# shell

## shell 脚本技巧

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



## Ref
[shell脚本教程](http://www.runoob.com/linux/linux-shell.html)
[expr命令使用技巧](https://blog.csdn.net/adcxf/article/details/3001275)
