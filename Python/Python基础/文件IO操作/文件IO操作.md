### 文件IO操作

| 函数      | 说明     |
| --------- | -------- |
| open      | 打开     |
| read      | 读取     |
| write     | 写入     |
| close     | 关闭     |
| readline  | 行读取   |
| readlines | 多行读取 |

#### open参数

主模式：不能共存

**mode=r**

readonly：只读，不可写，从头读起。文件不存在，FileNotFoundError

**mode=w**

writable ：只写，不可读，从头开始写。文件存在则清空；文件不存在，创建新文件

**mode=a**

append：尾部追加内容，只写不可读。文件存在，从末尾追加写入；文件不存在，创建新文件

**mode=x**

exist：只写，不可读。文件存在，FileExistsError；文件不存在，创建新文件

附加模式：和主模式配置使用，不能影响主模式

**mode=t**

text：文本的。默认模式，按字符读取

**mode=b**

binary：二进制，bytes

**mode=+**



**文本的世界是字符串的世界，和编码有关；二进制的世界是字节的世界，和编码没有任何关系**

#### read

read(size=-1)

* size表示读取的多少个字符或字节；负数或者None表示读取到EOF

建议使用文件对象时，一定要指定编码，而不是使用默认编码

#### write

* write(s)：文本模式时，从当前指针处把字符串s写入到文件中并返回写入字符的个数；二进制时将bytes写入文件并返回写入字节数
* writelines(lines)：将字符串列表写入文件

#### close

flush并关闭文件对象。文件已经关闭，再次关闭没有任何效果。可以查看文件对象的closed属性，判断是否关闭

### 上下文管理

#### with

```python
with 文件对象 as f:
	pass
```

with离开的时候，文件对象会调用close方法，不管with中是否正常执行了

上下文管理：

1. 使用with关键字，上下文管理针对的是with后的对象
2. 使用with...as关键字
3. 上下文管理的语句块并不会开启新的作用域

文件对象上下文管理

1. 进入with时，with后的文件对象是被管理对象
2. as子句后的标识符，指向with后的文件对象
3. with语句块执行完的时候，会自动关闭文件对象

#### 文件的遍历

```python
with open(filename, encoding="x") as f:
    for line in f:
        print(line, end='') # 带换行符
```

### 路径操作

#### os.path模块

from os import path

#### Path类

**from pathlib import Path**

#### 拼接

**操作符/**

* Path对象/Path对象
* Path对象/字符串
* 字符串/Path对象

* joinpath(*other)：在当前Path路径上连接多个字符串返回新路径对象

#### 分解

parts属性，会返回目录各部分元祖

.parts

#### 父目录

.parent

#### 目录组成部分

name、stem、suffix、suffixes、with_suffix(suffix)、with_name(name)

* name：目录的最后一个部分

* suffix：目录中最后一个部分的扩展名
* stem：目录最后一个部分，没雨后缀
* name = stem + suffix
* suffixes：返回多个扩展名列表
* with_suffix(suffix)：有扩展名则替换，无则补充扩展名
* with_name(name)：替换目录最后一个部分并返回一个新的路径

#### 全局方法

home()：返回当前家目录

cwd()：返回当前工作目录

#### 判断方法

* exists()：目录或者文件是否存在
* is_dir()：是否是目录，目录存在返回True
* is_file()：是否是普通文件，文件存在返回True
* is_symlink()：是否是软连接
* is_adsolute()：是否是绝对路径

#### 绝对路径

* resolve()：非windows，返回一个新的路径，这个新路径就是当前Path对象的绝对路径，如果是软连接则直接被解析
* absolute()：获取绝对路径。如果是软连接，不会被解析