# 命令行操作方式

## 常用命令
- `hadoop fs -ls /` 查看hdfs的根目录下的内容
- `hadoop fs -lsr /` 递归查看hdfs的根目录下的内容
- `hadoop fs -mkdir /d1` 在hdfs上创建文件夹d1
- `hadoop fs -put <linux source> <hdfs destination>` 将数据从linux上传到hdfs的特定路径中
- `hadoop fs -getmerge <hdfs destination> <linux source>` 将数据从hdfs合并下载到linux的特定路径中
- `hadoop fs -text <hdfs文件路径>` 查看hdfs中的文件
- `hadoop fs -rm <hdfs文件路径>` 删除hdfs中文件
- `hadoop fs -rmr <hdfs文件夹路径>` 递归删除hdfs中的文件夹

## python 接口

> 使用python接口调用shell执行hadoop命令

```python
os.system(cmd) # 返回值 0(成功), 1, 2
os.popen(cmd) # cmd的返回值为popen返回值
```

## Ref
[hadoop框架之HDFS的shell操作](www.cnblogs.com/cl1234/p/3566923.html)