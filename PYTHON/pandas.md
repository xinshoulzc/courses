# pandas

> pandas是python科学计算中一个十分有用的包, 在此做一个基本介绍以及常见函数的用法

## Data type in pandas
pandas的采用的主要数据结构为DataFrame, 与[R dataframe](https://github.com/xinshoulzc/courses/blob/master/R/R language.md)相同

## basics
- 创建 dataframe
```python
df = pd.DataFrame('A': [1, 2, 3, 4], 'B': [3, 4, 5, 6], 'C': [5, 6, 7, 8])
'''
  A B C
0 1 3 5
1 2 4 6
2 3 5 7
3 4 6 8
'''
```

- if-then `df.loc[df.A >= 3, 'B'] = -1` 将'A'大于等于3的行中'B'赋值为-1
    + `df.loc[df.A >= 3, ['B', 'C']] = -1` 赋值两列
    + `df['C'] = np.where(df['A'] > 3, 'high', 'low')` if > 3, then 'high', else 'low'
- 通过创建dataframe的mask矩阵进行if-then操作
```python
df_mask = pd.DataFrame('A': [True] * 4, 'B': [False] * 4, 'C': [True False])
# 长度不够的话倍长补齐
df.where(df_mask, -100)
```
- splitting 与python中切片保持一致
- 

## Ref
[10 Minutes to pandas](http://pandas.pydata.org/pandas-docs/stable/10min.html)
[pandas doc](http://pandas.pydata.org/pandas-docs/stable/10min.html)


pd.reset_index()
# 用于将group之后的dataframe重新分配index
https://blog.csdn.net/jingyi130705008/article/details/78162758

https://segmentfault.com/a/1190000002965620

mock

https://blog.csdn.net/zutsoft/article/details/51498026
pd.merge