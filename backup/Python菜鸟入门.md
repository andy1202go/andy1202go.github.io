## [[Python3 教程 | 菜鸟教程](https://www.runoob.com/python/python-tutorial.html)](https://www.runoob.com/python/python-tutorial.html)

**[[Python Tutor: Visualize code](https://pythontutor.com/visualize.html#mode=edit)](https://pythontutor.com/visualize.html#mode=edit)**

### Python 发展历史

Python 是由 Guido van Rossum 在八十年代末和九十年代初，在荷兰国家数学和计算机科学研究所设计出来的。

Python 本身也是由诸多其他语言发展而来的,这包括 ABC、Modula-3、C、C++、Algol-68、SmallTalk、Unix shell 和其他的脚本语言等等。

像 Perl 语言一样，Python 源代码同样遵循 GPL(GNU General Public License)协议。

现在 Python 是由一个核心开发团队在维护，Guido van Rossum 仍然占据着至关重要的作用，指导其进展。

Python 2.7 被确定为最后一个 Python 2.x 版本，它除了支持 Python 2.x 语法外，还支持部分 Python 3.1 语法。

### 输出中文

头加上**# coding=utf-8**

**注意：**Python3.X 源码文件默认使用utf-8编码，所以可以正常解析中文，无需指定 UTF-8 编码。

**注意：**如果你使用编辑器，同时需要设置 py 文件存储的格式为 UTF-8

### Python 标识符

- 字母、数字、下划线组成
- 区分大小写
- 以下划线开头的有特殊含义：
  - 单下划线开头的，_foo表示不能直接访问的类属性，不能通过from xxx import * 导入
  - 双下划线开头的，__foo代表类的私有成员
  - 双下划线开头和结尾的，`__foo__`代表python中的特殊用法

### python保留字符

- 全是小写
- 常用的一些英语词汇

### 行和缩进

- 保持一致，强制要求！！比如一个tab

### 关于引号

- 单引号，双引号都行，但前后保持一致

- 三引号比较特殊，可以多行文本使用，或者多行注释使用

  ```python
  paragraph='''段落第一行段落第二行'''"""这里是多行注释"""
  ```

### 关于空行

- 非强制要求
- 函数间最好加一个

### 关于不换行输出

```python
print("a","b")
```

### 标准数据类型

五个：数字，字符串，列表，元组（Tuple），字典

### 数字类型

- 是不可变类型的变量

- 有int，long，float，complex四种，python3是没有long了，统一int支持

- long的举例：`123123L`

- float：`2.32e10`

- complex：`3+2j`

- 类型转换：```int(x),float(x),str(x)```

- 使用math、cmath模块做数学运算

  ```python
  import math
  import cmath
  
  cmath.sin(1.5)
  
  
  ```

  

### 字符串类型

- 可以用`str[i]`取字符串中的字符

- 字符串的拼接`+`，重复``

- 字符串的切割遵循**[头下标:尾下标]**

- 不支持单字符类型，单字符在python中也是字符串类型

- 转义：反斜杠```\```

- 字符串有运算符，牛逼，+，*（重复输出），[]，in，not in

- 字符串格式化：s%，和C类似的操作

  ```python
  print("shit %s" % ("happens"))
  ```

- 三引号：Python 三引号允许一个字符串跨多行，字符串中可以包含换行符、制表符以及其他特殊字符。

  > 三引号让程序员从引号和特殊字符串的泥潭里面解脱出来，自始至终保持一小块字符串的格式是所谓的WYSIWYG（所见即所得）格式的。
  >
  > 一个典型的用例是，当你需要一块HTML或者SQL时，这时当用三引号标记，使用传统的转义字符体系将十分费神。

  ```python
  text_block = '''
  这是一个使用三引号的示例。
  这里有多行文字，可以自由插入。
  结束。
  '''
  print(text_block)
  ```

- unicode:先记录下

  ```python
  print(u"\u0020")
  ```

- 字符串内建函数：比如string.split()

### 列表类型

- 标识

- 可以包含数字、字符串甚至是列表类型的元素

- 举例：`list=[123,'abc','s']`

- 使用最频繁

- 函数和方法：

  ```python
  list1=[123,"asdf"]
  list2=["asdf",123]
  list3=["f","d"]
  cmp(list1,list2)
  len(list1)
  max(list3)
  min(list3)
  # 把元组转换为list
  list(seq) 
  
  list.append()
  list.insert(index,obj)
  list.count(obj)
  list.extend(seq)
  list.index(obj)
  list.pop([index])
  list.remove(obj)
  list.reverse()
  list.sort()
  ```

### 元组类型

- ()标识
- 可以包含数字、字符串甚至是列表类型的元素
- 举例：`tu=(123,'abc','s')`
- 不可修改，约等于只读列表

### 字典类型

- {}标识

- key-value形式

- key可以是数字或字符串；实际上更准确的描述是，key必须要是不可变的类型，所以key可以是数字、字符串、元组，不能是列表

- 可以通过.keys() .values()函数获取key列表，value列表

- key不能重复，重复的话，会覆盖掉

- 函数和方法

  ```python
  cmp(dict1,dict2)
  len(dict)
  str(dict)#3.1版本好像不能用来着
  type(dict)
  
  dict.clear()
  dict.copy()#浅复制
  dict.items() #相当于entrySet,以列表返回可遍历的(键, 值) 元组数组 dict_items([(1,123),(2,2324)])
  dict.keys()#以列表返回一个字典所有的键 dict_keys([1,2])
  dict.values()#以列表返回一个字典所有的值 dict_values([123,2324])
  ```

### 数据类型转换

各种函数，待使用中补充

### 运算符

- 特殊的运算符：成员运算符和身份运算符
- 成员：in，举例：`print(’a’ in list)`
- 身份：is，实际就是比对是否是同一个对象这样子，python也是面向对象的

### 条件语句

```python
num = 10
if num <= 90
	print("shit, too cheap")
elif num == 1000 and num = 9999
	print("haha")
else:
	print("I will take it anyway")
```

### while语句

```python
a=1
while (a<9 and a>3):
	print('ok')
	a = a+1
```

特殊的用法是**循环使用 else 语句**

```python
count = 0
while count < 5:
   print count, " is  less than 5"
   count = count + 1
else:
   print count, " is not less than 5"
```

### for循环语句

python的for循环语句，可以遍历任何序列的项目，比如列表或字符串

```python
long_str = 'adflj123'
for str in long_str:
	print(str)
```

可以通过其他参数，按序列号遍历

```python
long_str = 'adflj123'
for index in range(len(long_str)):
	print(long_str[index])
```

也可以像while一样用else循环使用

- 在 python 中，for … else 表示这样的意思，for 中的语句和普通的没有区别，else 中的语句会在循环正常执行完（即 for 不是通过 break 跳出而中断的）的情况下执行，while … else 也是一样

```python
for num in range(10,20):  # 迭代 10 到 20 之间的数字
   for i in range(2,num): # 根据因子迭代
      if num%i == 0:      # 确定第一个因子
         j=num/i          # 计算第二个因子
         print ('%d 等于 %d * %d' % (num,i,j))
         break            # 跳出当前循环
   else:                  # 循环的 else 部分
      print ('%d 是一个质数' % num)
```

### 日期和时间

- 主要使用time,calendar,datetime,pytz,deteutil模块

- 很多Python函数用一个元组装起来的9组数字处理时间

- calendar主要是日历，先不管，主要看time

  ```python
  import calendar
  import time
  
  calendar.month(2023,1)
  time.time()#当前时间戳
  time.localtime()#time.struct_time(tm_year=2016, tm_mon=4, tm_mday=7, tm_hour=10, tm_min=3, tm_sec=27, tm_wday=3, tm_yday=98, tm_isdst=0) 元组保存时间
  time.asctime()#获取格式化时间 Thu Apr  7 10:05:21 2016
  time.strftime(format[,time])#格式化日期 比如 %Y-%m-%d %H:%M:%S
  ```

  > python中时间日期格式化符号：
  >
  > - %y 两位数的年份表示（00-99）
  > - %Y 四位数的年份表示（000-9999）
  > - %m 月份（01-12）
  > - %d 月内中的一天（0-31）
  > - %H 24小时制小时数（0-23）
  > - %I 12小时制小时数（01-12）
  > - %M 分钟数（00-59）
  > - %S 秒（00-59）
  > - %a 本地简化星期名称
  > - %A 本地完整星期名称
  > - %b 本地简化的月份名称
  > - %B 本地完整的月份名称
  > - %c 本地相应的日期表示和时间表示
  > - %j 年内的一天（001-366）
  > - %p 本地A.M.或P.M.的等价符
  > - %U 一年中的星期数（00-53）星期天为星期的开始
  > - %w 星期（0-6），星期天为星期的开始
  > - %W 一年中的星期数（00-53）星期一为星期的开始
  > - %x 本地相应的日期表示
  > - %X 本地相应的时间表示
  > - %Z 当前时区的名称
  > - %% %号本身

### 函数

- 基本语法

  ```python
  def sumAll(*var_args):
      sum = 0
      for var in var_args:
              sum = sum + var
      return sum
  print(sumAll(1,2,3))
  ```

- 关于参数传递，需要注意传入的是什么类型，总体来分有两种大类型

  - 不可变类型对象：数字，字符串，元组
  - 可变类型对象：列表，字典
  - 可变类型对象，在函数中的修改，会影响入参本身

- 可变参数：如上面示例的参数就是

- lambda表达式创建函数

  > - lambda的主体是一个表达式，而不是一个代码块。仅仅能在lambda表达式中封装有限的逻辑进去。

  ```python
  # 可写函数说明
  sum = lambda arg1, arg2: arg1 + arg2
   
  # 调用sum函数
  print "相加后的值为 : ", sum( 10, 20 )
  print "相加后的值为 : ", sum( 20, 20 )
  ```

- return:

  - 可以没有return，这样会认为返回的是none

### 模块

- 模块是一个python文件，以.py结尾，包含了python对象定义和python语句

- 引入模块

  ```python
  import math
  from math import fibonacci #从math模块中，只引入fibonacci部分
  from math import *
  ```

- dir函数：可以看某个模块有哪些变量、函数

- 包：其实就是文件夹，但该文件夹下必须有特定文件，文件内容不管，但必须要有

  ```python
  __init__.py
  
  test.py
  package_runoob
  |-- __init__.py
  |-- runoob1.py
  |-- runoob2.py
  ```

  使用包的代码

  ```python
  from package.runoob1 import runoob1
  ```

### 文件I/O

- open()
- file函数
- 注意关闭文件流，释放资源

### 异常处理

- 基本语法：

  ```python
  try:
      # some code that might raise an exception
  except SomeException:
      # exception handling code
  else:
      # code to execute if no exception was raised
  finally:
      # code to execute regardless of whether an exception was raised
  ```

- except可以不加异常类型，catch各种类型

- 自行发起异常

  ```python
  raise [Exception [, args [, traceback]]]
  raise Exception("Invalid param")
  ```

- 自定义异常：通过创建一个新的异常类，程序可以命名它们自己的异常。异常应该是典型的继承自Exception类，通过直接或间接的方式。

  ```python
  class Networkerror(RuntimeError):
      def __init__(self, arg):
          self.args = arg
  ```