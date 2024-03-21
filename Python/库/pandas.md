## 安装使用

```shell
$ pip install pandas
```

```python
import pandas as pd
```

## `DataFrame`

> `DataFrame `是一个表格型的数据结构，它含有一组有序的列，每列可以是不同的值类型（数值、字符串、布尔型值）。`DataFrame `既有行索引也有列索引，它可以被看做由 Series 组成的字典（共同用一个索引）

```python
pandas.DataFrame( data, index, columns, dtype, copy)
```

**参数说明**：

- **data**：一组数据(`ndarray`、`series`, `map`, `lists`, `dict `等类型)。
- **index**：索引值，或者可以称为行标签。
- **columns**：列标签，默认为 `RangeIndex `(0, 1, 2, …, n) 。
- **dtype**：数据类型。
- **copy**：拷贝数据，默认为 False。



Pandas `DataFrame `是一个二维的数组结构，类似二维数组。

```python
import pandas as pd

data = [['Google',10],['Runoob',12],['Wiki',13]]

df = pd.DataFrame(data,columns=['Site','Age'],dtype=float)

print(df)
```

1.使用 `ndarrays `创建:

```python
import pandas as pd

data = {'Site':['Google', 'Runoob', 'Wiki'], 'Age':[10, 12, 13]}

df = pd.DataFrame(data)

print (df)
```

2.使用字典创建

```python
import pandas as pd

data = [{'a': 1, 'b': 2},{'a': 5, 'b': 10, 'c': 20}]

df = pd.DataFrame(data)

print (df)
```

## `loc` 返回指定行,列

> 格式：`df.loc[行标签,列标签]`
>
> 1.`df.loc['a':'b']` 选取ab两行数据
>
> 2.`df.loc[:,'one']` 选取one列的数据

> Pandas 可以使用 **`loc`** 属性返回指定行的数据，如果没有设置索引，第一行索引为 **0**，第二行索引为 **1**，以此类推：

```python
import pandas as pd

data = {
  "calories": [420, 380, 390],
  "duration": [50, 40, 45]
}

# 数据载入到 DataFrame 对象
df = pd.DataFrame(data)

# 返回第一行
print(df.loc[0])
# 返回第二行
print(df.loc[1])

```

**注意：**返回结果其实就是一个 Pandas Series 数据。



> 返回多行数据，使用 **[[ ... ]]** 格式，**...** 为各行的索引，以逗号隔开：

```python
import pandas as pd

data = {
  "calories": [420, 380, 390],
  "duration": [50, 40, 45]
}

# 数据载入到 DataFrame 对象
df = pd.DataFrame(data)

# df = pd.DataFrame(data, index = ["day1", "day2", "day3"])  # 可以指定索引值

# 返回第一行和第二行
print(df.loc[[0, 1]])
```

**注意：**返回结果其实就是一个 Pandas DataFrame 数据。



> 使用 **`loc`** 属性返回指定**索引** 或 **条件** 对应的某一行：

```python
import pandas as pd

data = {
  "calories": [420, 380, 390],
  "duration": [50, 40, 45]
}

df = pd.DataFrame(data, index = ["day1", "day2", "day3"])

# 指定索引
print(df.loc["day2"])

# 指定条件
df.loc[df['某个列名'] > '值']
```

### 替换

```python
df.loc[ (df.Cabin.notnull()), 'Cabin' ] = "Yes"  # 选取Cabin列中不为空的位置替换为“Yes”
```





## CSV 文件

### **to_csv()**

> 使用 **to_csv()** 方法将 `DataFrame `存储为 csv 文件

```python
import pandas as pd
   
# 三个字段 name, site, age
nme = ["Google", "Runoob", "Taobao", "Wiki"]
st = ["www.google.com", "www.runoob.com", "www.taobao.com", "www.wikipedia.org"]
ag = [90, 40, 80, 98]
   
# 字典
dict = {'name': nme, 'site': st, 'age': ag}
     
df = pd.DataFrame(dict)
 
# 保存 dataframe
df.to_csv('site.csv')
```

### 读取

#### read_csv()

```python
import pandas as pd

df = pd.read_csv('nba.csv')

# print(df.to_string())
```

#### head()

> **head(n)** 方法用于读取前面的 n 行，如果不填参数 n ，默认返回 5 行。

```python
import pandas as pd

df = pd.read_csv('nba.csv')

print(df.head(10))
```

#### tail()

> **tail( n )** 方法用于读取尾部的 n 行，如果不填参数 n ，默认返回 5 行，空行各个字段的值返回 **NaN**。

```python
import pandas as pd

df = pd.read_csv('nba.csv')

print(df.tail(10))
```

### 删除

```python
import pandas as pd
import csv
 
data = pd.read_csv("filename.csv")
 
# 删除某些固定行
data_new=data.drop([row1, row2,...])
 
# 删除某些列
data_new=data.drop(["column1"，'column2','column3']，axis=1) 
 
# 生成新的文件
data_new.to_csv("filename.csv",index=0)
```

### 替换

```python
import pandas as pd
 
# 读取文件
data = pd.read_csv("filename.csv")
 
# 原来的值：新的值
# 使用map属性 
change= {'old_value1':'new_value1', 'old_value2':num2}
data['column_name'] = data['column_name'].map(change)
 
#生成新的文件
data.to_csv("filename_new.csv")

```





## JSON数据

```json
// sites.json
[
   {
   "id": "A001",
   "name": "菜鸟教程",
   "url": "www.runoob.com",
   "likes": 61
   },
   {
   "id": "A002",
   "name": "Google",
   "url": "www.google.com",
   "likes": 124
   },
   {
   "id": "A003",
   "name": "淘宝",
   "url": "www.taobao.com",
   "likes": 45
   }
]
```

```python
import pandas as pd

df = pd.read_json('sites.json')
   
print(df.to_string())
```

### 内嵌的 JSON 数据

```json
{
    "school_name": "ABC primary school",
    "class": "Year 1",
    "students": [
    {
        "id": "A001",
        "name": "Tom",
        "math": 60,
        "physics": 66,
        "chemistry": 61
    },
    {
        "id": "A002",
        "name": "James",
        "math": 89,
        "physics": 76,
        "chemistry": 51
    },
    {
        "id": "A003",
        "name": "Jenny",
        "math": 79,
        "physics": 90,
        "chemistry": 78
    }]
}
```

需要使用到 **json_normalize()** 方法将内嵌的数据完整的解析出来

```python
import pandas as pd
import json

# 使用 Python JSON 模块载入数据
with open('nested_list.json','r') as f:
    data = json.loads(f.read())

# 展平数据
df_nested_list = pd.json_normalize(data, record_path =['students'])  # 设置为 ['students'] 用于展开内嵌的 JSON 数据 students
print(df_nested_list)
```

显示结果还没有包含 school_name 和 class 元素，如果需要展示出来可以使用 meta 参数来显示这些元数据：

```python
import pandas as pd
import json

# 使用 Python JSON 模块载入数据
with open('nested_list.json','r') as f:
    data = json.loads(f.read())

# 展平数据
df_nested_list = pd.json_normalize(
    data,
    record_path =['students'],
    meta=['school_name', 'class']
)
print(df_nested_list)
```

