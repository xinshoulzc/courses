# R language



## Basic

- *<-* 赋值 
- <div id="c()" />*c()* combine 相同类型的object组成vector, 如果类型不相同的话则会采用最低公共类型
- *#* 单行注释
- Inf/NaN Inf表示正无穷大(1/0), -Inf表示负无穷大, NaN表示未定义值(0/0)
- Na 表示缺失值, 注意与NaN进行区分 NaN belongs to NA
- **变量名可以包含.**
- **数组下标从1开始**

### data type 


- *atomic class* characters, numberic(与C中double相同), integer(通过整数之后添加L), complex, logical(FALSE, TRUE)
- *object* vector 所有元素的基本类型均相同
    + vector("numberic", 10) type, length不赋值采用默认值; [c()](#c())
- *list* 元素可能存在不同的类型 列表存在索引
- *factor* 因子变量 用于表示有序或者无序变量的索引
    + table("factor") 统计各个level出现的频率
    + unclass("factor") 去除各个level的标签直接用原始数据来表示
    + levels参数(factor(..., levels=...))表示对factor设置优先级
- *matrix* 矩阵, 所有元素必须具有相同的数据类型


### data frame 

> 表格类型, 注意与matrix的区别. 

- 每列具有相同的数据类型, 并且每行可以独立命名, row.name
- <div id="readcsv" />一般通过read.table() read.csv()创建, 当然也可以通过data.frame()来创建
    + data.frame(foo=1:4, bar=c(T,T,F,F))
- 通过data.matrix() 将data frame 转化成matrix
- ncol() #返回列数, nrow #返回行数

## read write

- [read.table() read.csv()](#readcsv) return data frame write.table()
- readLines() writeLines() 逐行读入/写入文本数据
- source() dump() 读取并运行指定位置的.R脚本/写入脚本到指定位置
- dget() dput() 读取/写入数据之后保留了元数据信息(应该会比read.table()快?)
    + dput() 通过自动生成部分代码重构某一R对象，保留元数据信息
    + dget() 通过dget()获取dput()存储的对象
```R
y <- data.frame(a = 1, b = "a")
dput(y, file="y.R")
# structure(list(a = 1, b = structure(1L, .Label = "a", class="factor")),
# .Names = c("a", "b"), row.names = c(NA, -1L), class = "data.frame")
# above code will be stored in y.R
```
- load() save()
- unserialize() serialize() 读取/写入二进制文件中的object
```R
read.table(file, header, sep, colClasses, nrows, comment.char, skip, stringsAsFactors)
# https://www.rdocumentation.org/packages/utils/versions/3.5.0/topics/read.table
# for large dataset details, please read above doc.
# file <- 文件路径(string)
# header <- 是否含有表头(T/F) 即是否含有col.name, 默认TRUE
# sep <- 数据之间的分隔符(string/characters) read.table默认空格, csv默认逗号
# colClasses <- 表示每列所属的数据类型(其长度与ncol()保持一致)
# nrows <- 数据行数(not required)
# comment.char <- 表示注释的字符(一般通常是#)以注释字符开始的行将被忽略
# skip <- 表示开头跳过多少行
# stringAsFactors <- 是否将string表示为factors类型变量(默认是T)
```
- file() gzfile() bzfile() url() 建立与各种文件以及网页之间的连接
```R
# file(description="filepath", open="w/r/a/rb/wb/ab", block=T, encoding=getOption("encoding"))
con <- gzfile("word.gz")
x <- readLines(con, 10)
close(con)
con <- file("foo.txt")
x <- read.table(con)
close(con)
```


## Skill

- **getwd()** 获取当前工作目录(仅有工作目录下的文件才能运行)
- *ls()* 获取当前加载入console的类
> print中的[1] 代表的含义就是输出的元素是一个向量并且输出值在向量中的位置为1
- *cbind() rbind()* 通过列绑定或者行绑定来构造矩阵
- *names()* 通过给vector/list的每个元素赋予name增加代码的可阅读性
- *dimnames()* 给matrix赋值
``` R
m <- matrix(1:4, nrow = 2, ncol = 2)
dimnames(m) <- list(c("a", "b"), c("c", "d"))
# m 
#   c d
# a 1 3 
# b 2 4
```
- 读入大数据集之前必须计算下内存是否会爆炸
- dput()与dump()区别: dump()可以存储多个对象; dput()只能存储一个对象

### Subset operation

- [, [[, $
    + [ 返回子集(ret数值是相同类型的) x[1] x[1:4] same class
    + [[ 返回某一行或者某一列或者某个元素
## Ref

- [R语言入门指导](https://wklchris.github.io/R-learning-basic.html)
- [R programming(coursera)](https://www.coursera.org/learn/r-programming/home/welcome)

## 中英文转化表

- coercion/coerce 强制类型转化
- csv comma seperate value 逗号分隔值