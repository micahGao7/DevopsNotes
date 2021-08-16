### csv文件

#### CSV文件简介

* 一个被行分隔符、列分隔符划分成行和列的文本文件
* csv不指定字符编码
* 每一行称为一条记录record
* 表头可选，可字段列对齐就行了

分隔符

* 行分隔符\r\n，最后一行可以没有换行符
* 列分隔符常为逗号或者制表符

字段

* 字段可以使用双引号括起来，也可以不使用
* 如果字段中出现了双引号、逗号、换行符必须使用双引号括起来
* 如果字段的值是双引号，使用两个双引号表示一个转义

#### csv模块

```python
import csv
with open('t.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(rows[0])
    writer.writerow(rows[1:])
```

