---
layout: post
title: Python高级变量——容器
subtitle: false
tags: [Python]
---

<!-- # Python高级变量——容器 -->

## 列表

### 1.1 列表定义

- `list`（列表）是Python中使用最频繁的类型，在其他语言中通常叫做**数组**
- 专门用于存储一串信息
- 列表用`[]`定义，数据之间使用`,`分割
- 列表的索引从`0`开始
  - 索引就是数据在列表中的位置编号，索引又可被称为下标

> 注意：从列表取值时，如果**超出索引范围**，程序会报错

```python
name_list = ["zhangsan", "lisi", "wangwu"]
```

### 1.2 列表常用操作

在`ipython3`中定义一个列表，如上述列表，按下`TAB`键，可以提示列表所能使用的**方法**如下：

```python
name_list.append  name_list.count   name_list.insert  name_list.reverse
name_list.clear   name_list.extend  name_list.pop     name_list.sort
name_list.copy    name_list.index   name_list.remove
```

| 分类 | 关键字/函数/方法     | 说明                     |
| :--: | -------------------- | ------------------------ |
| 增加 | insert（索引，数据） | 在指定位置插入数据       |
|      | append（数据）       | 在末尾追加数据           |
|      | extend（列表2）      | 将列表2的数据追加到列表  |
| 修改 | 列表[索引] = 数据    | 修改指定所以的数据       |
| 删除 | del  列表            | 删除指定索引的数据       |
|      | remove[数据]         | 删除第一个出现的指定数据 |
|      | pop                  | 删除末尾数据             |
|      | pop（索引）          | 删除指定所以的数据       |
|      | clear                | 清空列表                 |
| 统计 | len（列表）          | 列表长度                 |
|      | count（数据）        | 数据在列表中出现的次数   |
| 排序 | sort()               | 升序排序                 |
|      | sort(reverse=True)   | 降序排序                 |
|      | reverse()            | 逆序，反转               |

> `del` 关键字本质上是将一个变量从内存中删除

### 1.3 循环遍历

遍历就是从头到尾依次从列表中获取数据，在循环体内部针对每一个元素，执行相同的操作，在Python中为了提高列表的遍历效率，专门提供的**迭代iteration**遍历，使用for就可实现迭代遍历

```python
# for 循环内部使用的变量 in 列表
for name in name_list:
	循环内部针对列表元素进行操作
	print(name)
```

尽管Python 的列表可以存储不同类新的数据，但是更多的应用场景是：

1. 列表存储相同类型的数据
2. 通过迭代遍历，在循环体内部，针对列表中的每一项元素，执 行相同的操作



## 元组

### 2.1元组的定义

- Tuple（元组）与列表类似，不同之处在于**元组的元素不能修改**
  - 元组 表示多个元素组成的序列
  - 元组在python开发中，有特定的应用场景

- 用于存储一串信息，数据之间用`,`分割
- 元组用`()`定义
- 元组的索引从`0`开始，索引就是数据在元组中的位置编号

```python
info_tuple = ("zhangsan", 18, 1.75)
```

创建空元组

```python
info_tuple = ()
```

元祖中只包含一个元素时，需要在元素后面添加逗号

```python
info_tuple = (50, )	# tuple

# 反例
singel_tuple = (50)
print(single_tuple)  # int
```

### 2.2 元组常用操作

在`ipython3`中定义一个元组，如上述列表，按下`TAB`键，可以提示列表所能使用的**方法**如下：

```python
info.count  info.index
```

### 2.3 循环遍历

取值是从元组汇总获取存储的指定位置的数据，遍历是从头到尾依次从元组中获取数据

```python
# for 循环内部使用的变量 in 元组
for name in info:
	循环内部针对列表元素进行操作
	print(name)
```

> 在Python中，可以使用for循环遍历所有非数字型类型的变量：列表，元组，字典和字符串
>
> 提示：在实际开发中，除非能够确认元组中的数据类型，否则针对元组的循环遍历需求并不是很多

### 2.4应用场景

- 函数的 **参数** 和**返回值**，一个函数可以接受**任意多个参数**，或者**一次返回多个数据**
- **格式化字符串**，格式化字符串 `%` 后面的`()`本质上就是一个元组
- **让列表不可以被修改，**以保护数据安全

```python
 info_tuple = {"小明", 18, 1.85}
 print("%s 年龄是 %d, 身高 %.2f m" % (info_tuple))
 
 info_str = "%s 年龄是 %d, 身高 %.2f m" % (info_tuple)
 print(info_str)
```

元组和列表之间的转换

- 使用list函数可以把元组转换成列表

```
list(元组)
```

- 使用tuple 函数可以把列表转换成元组

```
tuple(列表)
```



## 字典

### 3.1字典的定义

- dictionary （字典）是除列表外Python中最灵活的数据类型，通常用来描述一个物体的相关信息

- 和列表的区别：
  - 列表是有序的对象集合
  - 字典是无序的对象集合

- 字典用`{}`定义
- 字典使用键值对存储数据，键值对之间使用`,`分隔
  - 键`key`是索引
  - 值`value`是数据
  - 键 和 值 之间使用`:`分隔
  - **键 必须是唯一的**
  - **值** 可以取任何数据类型，但 **键** 只能使用**字符串、数字或元组**

```python
xiaoming = {
    "name": "小明",
    "age": 18,
    "gender": True,
    "height": 1.75}
```

### 3.2 字典常用操作

在`ipython3`中定义一个字典，如上述列表，按下`TAB`键，可以提示列表所能使用的**方法**如下：

```python
dict.clear	dict.items	dict.setdefault
dict.copy	dict.keys	dict.update
dict.fromkeys	dict.pop	dict.values
dict.get	dict.popitem
```

### 3.3 循环遍历

遍历就是依次从字典中获取所有键值对

```python
# for 循环内部使用的'key'的变量 in 字典
for k in dict:
	print("%s, %s" % (k, dict[k]))
```

### 3.4 应用场景

- 使用 多个键值对，存储描述一个物体的相关信息——描述更复杂的数据信息
- 将多个字典放在一个列表中，再进行遍历，在循环体内部针对每一个字典进行相同的处理

```python
card_list = [
    {"name": "zhangsan",			
     "qq": "12345",
     "phone": "110"}.

    {"name": "lisi",
     "qq": "54321",
     "phone": "10086"}
]
```



## 字符串

### 4.1 字符串的定义

- 是编程语言表示文本的数据类型
- 在Python中可以使用一对双引号`""`或者一对单引号`''`定义一个字符串

  - 虽然可以使用`\" \'`做字符串的转义，但是在实际开发中：如果字符串内部需要使用 `“`，可以使用 `'`义字符串，如果字符串内部需要使用  `'`，可以使用  `“`定义字符串
- 可以使用索引获取一个字符串中指定位置的字符，索引计数从0开始
- 可以使用`for`循环遍历字符串中每一个字符

```python
string = "Hello Python"

for c in string:
	print(c)
```

### 4.2 字符串常用操作

在`ipython3`中定义一个字符串，如上述列表，按下`TAB`键，可以提示列表所能使用的**方法**如下：

```python
str.capitalize   str.isalnum      str.join         str.rsplit
str.casefold     str.isalpha      str.ljust        str.rstrip
str.center       str.isdecimal    str.lower        str.split
str.count        str.isdigit      str.lstrip       str.splitlines
str.encode       str.isidentifier str.maketrans    str.startswith
str.endswith     str.islower      str.partition    str.strip
str.expandtabs   str.isnumeric    str.replace      str.swapcase
str.find         str.isprintable  str.rfind        str.title
str.format       str.isspace      str.rindex       str.translate
str.format_map   str.istitle      str.rjust        str.upper
str.index        str.isupper      str.rpartition   str.zfill
```

常用方法举例：

1）字符串查找和替换

|                       方法                        | 说明                                                         |
| :-----------------------------------------------: | ------------------------------------------------------------ |
|              str.startswith(string)               | 检查字符串是否以string开头，是则返回True                     |
|               str.endswith(string)                | 检查字符串是否以string结尾，是则返回True                     |
|     str.find(string, strart=0, end=len(str))      | 检测string是否包含在str中，如果start和end指定范围，则检查是否包含在指定范围内，如果是则返回开始的索引值，否则返回-1 |
|     str.index(string, start=0, end= len(str))     | 跟find()方法类似，只不过如果string不在str中会报错            |
| str.replace(old_str, new_str, num=str.count(old)) | 把str中的old_str替换成new_str，如果num指定，则替换不超过num次 |

2）文本对齐

|       方法        | 说明                                                        |
| :---------------: | ----------------------------------------------------------- |
| str.center(width) | 返回一个原字符串居中，并使用空格填充至长度width的新字符串   |
| str.ljust(width)  | 返回一个原字符串左对齐，并使用空格填充至长度width的新字符串 |
| str.rjust(width)  | 返回一个原字符串右对齐，并使用空格填充至长度width的新字符串 |

3）去除空白字符

|     方法     | 说明                          |
| :----------: | ----------------------------- |
| str.lstrip() | 截掉str左边（开始）的空白字符 |
| str.rstrip() | 截掉str右边（末尾）的空白字符 |
| str.strip()  | 截掉str左右两边的空白字符     |

4）拆分和连接

|             方法              | 说明                                                         |
| :---------------------------: | ------------------------------------------------------------ |
|     str.partition(string)     | 把字符串string分成一个3元素的元组（string前面，string，string后面） |
|    str.rpartition(string)     | 类似于partition方法，不过是从右边开始查找                    |
| **str.split(string="", num)** | 以string为分隔符拆分str，如果num有指定值，则仅分隔num+1个子字符串，str默认包括'\r'，'\t'，'\n'和空格 |
|       str.splitlines()        | 按照行（'\r'，'\t'，'\r\n'）分隔，返回一个包含各行作为元素的列表 |
|       **str.join(seq)**       | 以str作为分隔符，将seq中所有的元素（的字符串表示）合并为一个新的字符串 |



## 公共方法

### 5.1 python内置函数

|       函数        | 描述                                 | 备注                    |
| :---------------: | ------------------------------------ | ----------------------- |
|     len(item)     | 计算容器中元素个数                   |                         |
|     del(item)     | 删除变量                             | del有两种方式           |
|     max(item)     | 返回容器中元素最大值                 | 如果是字典，只对key比较 |
|     min(item)     | 返回容器中元素最小值                 | 如果是字典，只对key比较 |
| cmp(item1, item2) | 比较两个值，-1小于 / 0 相等 / 1 大于 | Python3.X取消了cmp函数  |

### 5.2 切片

| 描述 | Python表达式       | 结果    | 支持的数据类型     |
| ---- | ------------------ | ------- | ------------------ |
| 切片 | "0123456789"[::-2] | "97531" | 字符串，列表，元组 |

- 切片使用索引值来限定范围，从一个大的字符串中切出一个小的字符串
- 列表和元组都是有序的集合，都能够通过索引值获取到对应的数据
- 字典是一个无序的集合，是使用键值对保存的数据

### 5.3 运算符

|    运算符    | Python表达式          | 结果                     | 描述           | 支持的数据类型           |
| :----------: | --------------------- | ------------------------ | -------------- | ------------------------ |
|      +       | [1, 2] + [3, 4]       | [1, 2, 3, 4]             | 合并           | 字符串、列表、元组       |
|      *       | ["Hi"] * 4            | ["Hi", "Hi", "Hi", "Hi"] | 重复           | 字符串、列表、元组       |
|      in      | 3 in (1, 2, 3)        | True                     | 元素是否存在   | 字符串、列表、元组、字典 |
|    not in    | 4 not in (1, 2, 3)    | True                     | 元素是否不存在 | 字符串、列表、元组、字典 |
| > >= == < <= | (1, 2, 3) < (2, 2, 3) | True                     | 元素比较       | 字符串、列表、元组       |

注意：

- `in` 在对**字典**操作时，针对的是**字典的键**
- `in` 和 `not in` 被称为 **成员运算符**

**成员运算符：**

| 运算符  | 描述                                                | 实例                         |
| ------- | --------------------------------------------------- | ---------------------------- |
| in      | 如果在指定的序列中找到值返回True，否则返回False     | 3 in (1, 2, 3) 返回True      |
| not  in | 如果在指定的序列中灭有找到值返回True，否则返回False | 3 not in (1, 2, 3) 返回False |



### 5.4 完整的for 循环语法

在Python中完整的for循环的语法如下：

```python
for 变量 in 集合：
	循环体代码
else:
	没有通过 break 退出循环，循环结束后，会执行的代码
```

只有保证for循环的代码全部顺利执行后，才会接着执行else的代码；一旦for循环内部遇到break，被打断，else中的代码将不会被执行

**应用场景**：

- 在**迭代遍历**嵌套的数据类型时，例如，**一个列表包含了很多个字典**
- 需求：要判断某一个字典中 是否存在指定的值
  - 如果 **存在**，提示并且推出循环
  - 如果 **不存在**，在**循环整体结束后**，希望**得到一个统一的提示**

```python
students = [
    {"name": "小明",
     "age": 20,
     "gender": True,
     "heigh": 1.7,
     "weight": 75.0},
    {"name": "小美",
     "age": 19,
     "gender": False,
     "heigh": 1.6,
     "weight": 45.0},
]

# 在学院列表中搜索指定姓名
find_name = "尼古拉奇"

for stu_dict in students:
    print(stu_dict)
    if stu_dict["name"] == find_name:
        print("找到了 %s" % find_name)

        # 如果已经找到，应该直接退出循环，而不再遍历后面的元素
        break
    """
    下面两行代码不合理，对于每次（stu_dict["name"] == find_name）不相等的情况，都会被print
    """
    # else:
    #     print("抱歉，没有找到 %s" % find_name)
else:
    # 如果希望在搜索列表时，所有的字典检查都没有发现需要搜索的目标，还需要得到一个统一的提示
    print("抱歉，没有找到 %s" % find_name)
print("循环结束")
```











