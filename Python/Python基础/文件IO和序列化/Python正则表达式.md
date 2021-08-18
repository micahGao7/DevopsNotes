### Python的正则表达式

#### 常量

| 常量 | 说明 |
| ---- | ---- |
| re.M |      |
|      |      |
|      |      |
|      |      |

#### re模块使用

* match：默认必须从0处匹配上，或指定位置开始匹配，搜不到返回None
* search：在字符串中搜索，从0或者指定位置开始向后搜索，搜不到返回None
* fullmatch：要求全长匹配（如果指定子串，子串要全匹配），一个不落
* findall：在文本中，全文搜索，匹配多次。返回列表，元素是字符串
* finditer：全文搜索，但是返回迭代器，元素是match对象
* sub
* subn

```python
regex = re.compile(pattern, Flag)
m = regex.search(string, pos, endpos)

m = re.search(pattern, string, Flag)
```

