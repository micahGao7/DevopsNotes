## Iptables

### 1 Iptables防火墙实践

#### 1.1 表的功能

| 表名   | 作用                          | 包含的链                                        |
| ------ | ----------------------------- | ----------------------------------------------- |
| filter | 过滤功能                      | INPUT、OUTPUT、FORWARD                          |
| nat    | 网络地址转换功能              | PREROUTING、INPUT、OUTPUTPOSTROUTING            |
| mangle | 修改数据包内容                | INPUT、OUTPUT、FORWARD、POSTROUTING、PRETOUTING |
| raw    | 关闭nat表上启用的连接追踪机制 | PREROUTING、OUTPUT                              |

 #### 1.2 