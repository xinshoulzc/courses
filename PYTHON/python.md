# advanced python

## python 元编程

- `@wraps(func)`可以用来保持原始函数的元数据

### 函数式编程

- 函数可以作为参数传递, 即函数和变量具有相同的地位 
> python中的lambda就是函数式编程的体现, 可以被任意赋值给其他变量

- python中的闭包设定. 内部函数可以引用外部函数的变量, 并且一旦外部函数的变量"freeze", 内部函数可以引用freeze之后的变量数值
```python
# python 闭包的一些坑
def count():
    fs = []
    for i in range(1, 4):
        def f():
            return i*i
        fs.append(f)
    return fs
f1, f2, f3 = count()
# f1() == f2() == 9 i为指针
def count():
    fs = []
    for i in range(1, 4):
        def f(j):
            def g():
                return j*j
            return g
        fs.append(f(i))
f1, f2, f3 = count()
# f1() = 1 f2() = 4 f3() = 9 
```

### partial function / 偏函数

- 固定函数的部分参数并赋予新的函数名称
```python
from functools
int("12345") # 将字符串转化成整数类型的数据

int2 = functools.partial(int, base=2)
# def int2(x, base=2)
#   return int(x, base)
```

### 给函数参数增加元信息

- 使用函数参数注解
```python
# 下面有一个被注解的函数 用于表示注释
def add(x:int, y:int) -> int:
    return x + y
```
```shell
>>> add.__annotations__
{'y': <class 'int'>, 'return': <class 'int'>, 'x': <class 'int'>}
```