# datetime

```python
from datetime import datetime
# two type
# datetime.datetime 下文中我们用D代表
# string 下文中我们用S代表
```
## 常用函数

- `strftime()` str format 将D转化成S `D.strftime("%y%m%d")`
- `strptime()` str point time 将S转化成D `datetime.strptime(S, "%y%m%d")`
- `timedelta(days=0, seconds=0, micseconds=0, ...)` 两个D类型相减就可以产生timedelta `timedelta(1) #常用加减一天`
- 