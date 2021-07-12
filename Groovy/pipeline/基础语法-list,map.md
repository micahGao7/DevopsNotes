## 基础语法-list,map

### 列表元素增删

```groovy
// 定义列表
devopsTools = ["jenkins", "gitlab", "nexus", "sonarqube", "kubernetes", "cicd"]
// list元素增删
println(devopsTools + "helm")
println(devopsTools - "cicd")
println(devopsTools << "argocd")	// + - << 生成新列表	
devopsTools.add("jira")				// add修改原列表
println(devopsTools)
```

### 列表操作

```groovy
// 判断元素是否为空
println(devopsTools.isEmpty())

// 列表去重
println(devopsTools.unique())	`

// 列表反转
println(devopsTools.reverse())

// 列表排序
println(devopsTools.sort())

// 判断列表是否包含元素
println(devopsTools.contains("devops"))

// 列表长度
println(devopsTools.size())

// 扩展列表定义方式
String[] stus = ["zhangsan", "lisi", "wangwu"]  //string类型列表
def numList = [1, 2, 3, 4, 4, 4] as int[]		//int类型列表

// 通过索引获取列表元素
println(numList[2])

// 计算列表中元素出现的次数
println(numList.count(4))
```

### map

```groovy
// 定义map
def mytools = [ "mvn": "/usr/local/maven",
				"gradle": "/usr/local/gradle" ]
				
// 根据key获取value
println(mytools["mvn"])
println(mytools.get("gradle"))

// 根据key重新赋值
mytools["mvn"] = "/opt/local/maven"
mytools.gradle = "/opt/local/gradle"

// 判断map是否包含某个key或者value
println(mytools.containsKey("mvn"))
println(mytools.containsValue("/usr/local/maven"))

// 返回map的key列表
println(mytools.keySet())

// 根据key删除元素
mytools.remove("mvn")
println(mytools)
```

