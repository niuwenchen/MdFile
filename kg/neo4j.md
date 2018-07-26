## neo4j
**Neo4j的特点**
* SQL就像简单的查询语言Neo4j CQL
* 遵循属性图数据模型
* 通过使用Apache Lucence支持索引
* 支持UNIQUE约束
* 包含一个用于执行CQL命令的UI: Neo4j数据浏览器
* 支持完整的ACID
* 采用原生图形库与本地GPE
* 支持查询的数据导出到JSON和XLS格式
* 提供了可以通过任何UI MVC框架访问的Java脚本

**优点**
* 很容易表示连接的数据
* 检索、遍历、导航更多的连接数据非常容易和快速的
* 非常容易的表示半结构化数据
* CQL查询语言命令是人性化的可读格式
* 简单而强大的数据模型
* 不需要复杂的连接来检索连接的/相关的数据

**缺点**
* AS的neo4j 2.1.3 最新版本，具有支持节点数目，关系和属性的限制
* 不支持Sharding

### 数据模型
图模型

属性图模型规则

* 表示节点，关系和属性中的数据
* 节点和关系都包含属性
* 关系连接节点
* 属性是键值对
* 节点用圆圈表示，关系用方向键表示。
* 关系具有方向：单向和双向。
* 每个关系包含“开始节点”或“从节点”和“到节点”或“结束节点”


### Neo4j构建模块
**节点**
节点是图标的基本单位。包含具有键值对的属性，如下
Node Name =“Employee”

![](https://7n.w3cschool.cn/attachments/day_161226/201612260847226397.png)
**属性**
用于描述图节点和关系的键值对

**Key=值**
key是一个字符串

**关系**
关系是图形数据库的另一个主要构建块，连接两个节点
![](https://7n.w3cschool.cn/attachments/day_161226/201612260904329065.png)

这里Emp和Dept是两个不同的节点。 “WORKS_FOR”是Emp和Dept节点之间的关系。
因为它表示从Emp到Dept的箭头标记，那么这种关系描述的一样
Emp WORKS_FOR Dept

![](https://7n.w3cschool.cn/attachments/day_161226/201612260907367369.png)

**标签**
Label将一个公共名称与一组节点或关系相连接。节点或关系可以包含一个或多个标签。 我们可以为现有节点或关系创建新标签。 

## CQL
CQL代表Cypher查询语言。

**Neo4j CQL**
* 是Neo4j图像数据库的查询语言
* 是一种声明性模式匹配 语言
* 遵循SQL语法
* 语法非常简单且人性化、可读的格式

**命令**
```
create  创建节点、关系和属性
match	匹配，检索有关节点，关系和属性数据
return	返回，返回查询结果
where	哪里，提供条件过滤检索数据
delete	删除，删除节点和关系
remove	移除，删除节点和关系的属性
order by	以....排序，排序检索数据
set 组	添加或更新标签


## CQL函数
string	字符串  用于使用string字面量
aggregation	聚合	用于对CQL查询结果执行聚合操作
relationship	用于获取关系的细节

## 数据类型
boolean		用于表示布尔文字
byte		用于表示8位整数
short		用于表示16位整数
int			用于表示32位整数
long		用于表示64位整数
float		用于表示32位浮点数
double		用于表示64位浮点数
char		用于表示16位字符串
string		用于表示字符串
```

**create命令**
* 创建没有属性的节点
* 使用属性创建节点
* 在没有属性的节点之间创建关系
* 使用属性创建节点之间的关系
* 为节点或关系创建单个或多个标签

```
create (<node-name>:<label-name>)
node-name: 节点名称
label-name: 节点标签名称

Neo4j数据库服务器使用此<node-name>将此节点详细信息存储在DataBase.AS中作为Neo4j DBA或者Developer，不能使用它来访问节点详细信息
Neo4j数据库服务器创建一个<label-name>作为内部节点名称的别名，使用此标签名称来访问节点详细信息


# 创建具有属性的节点
CRETAE (
	<node-name>:<label-name>
    {
    	<property1-name>:<Property1-value>
        ...
        ...
    
    }
)

create (emp:Employee{id:123,name:"Lokesh",sal:53000,depth:10})

说法: 创建一个具有4个属性的节点，并分配了一个标签Employee

```


**Match命令**
Match命令用于

* 从数据库获取有关节点和属性的数据
* 从数据库获取有关节点，关系和属性的数据

```
Match (<node-name>:<label_name>)
Neo.ClientError.Statement.SyntaxError: Query cannot conclude with MATCH (must be RETURN or an update clause) (line 1, column 1 (offset: 0))
"MATCH   (dept:Dept)"
 ^
```

**Return子句**
* 检索节点的某些属性
* 检索节点的所有属性
* 检索节点和关联关系的某些属性
* 检索节点和关联关系的所有属性

```
return <node-name>.<property1-name>
....

Neo.ClientError.Statement.SyntaxError: Query cannot conclude with MATCH (must be RETURN or an update clause) (line 1, column 1 (offset: 0))
"MATCH   (dept:Dept)"
```


**MATCH & RETURN匹配和返回**
```
Match command
return command

#返回该表现下的所有数据
MATCH (n: Employee)
RETURN n

```


**关系基础**
构建属性图模型，关系应该是定向的。否则，Neo4j将抛出一个错误消息
基于方向性，Neo4j关系被分为两种主要类型

	单向关系
    双向关系

创建如下关系

![](https://www.tutorialspoint.com/neo4j/images/create_relationship_example1.png)

**使用现有节点创建没有属性的关系**
使用两个现有节点CreditCard 和 Customer创建没有属性的关系。这意味着，我们的Neo4j数据库应该有两个节点

```
MATCH (<node1-label-name>:<node1-name>),(<node2-label-name>:<node2-name>)
CREATE  
	(<node1-label-name>)-[<relationship-label-name>:<relationship-name>]->(<node2-label-name>)
RETURN <relationship-label-name>

如下
MATCH (e:Customer),(cc:CreditCard) 
CREATE (e)-[r:DO_SHOPPING_WITH ]->(cc) 
创建relationship为 DO_SHOPPING_WITH 的关系


按照上面创建过程，查询关系
MATCH (e)-[r:DO_SHOPPING_WITH ]->(cc) 
RETURN r

Match (e) -[r:relationship_name]->(cc)
return r
主要是里面的relationship_name 的匹配
但是执行失败，改用以下查询

MATCH p=()-[r:DO_SHOPPING_WITH ]->() RETURN p


```
![Markdown](http://i1.bvimg.com/654958/7d741c4ad0764f5c.png)
![Markdown](http://i1.bvimg.com/654958/88b917a2b72b7119.png)

**Neo4j-与现有节点的属性的关系**
```
MATCH (cust:Customer),(cc:CreditCard) 
CREATE (cust)-[r:DO_SHOPPING_WITH{shopdate:"12/12/2014",price:55000}]->(cc) 
RETURN r

创建关系，然后在关系后面加{} json 表示属性内容
match p=()-[c:DO_SHOPPING_WITH]->() return p
可以返回所有查询到的关系

```

**使用新节点创建没有属性的关系**
```
CREATE  
   (<node1-label-name>:<node1-name>)-
   [<relationship-label-name>:<relationship-name>]->
   (<node1-label-name>:<node1-name>)
RETURN <relationship-label-name>


```


### Create 创建标签
CQL创建节点标签

Label是Neo4j数据库中的节点或关系的名称或标识符,可以将此标签名称称为关系类型。可以使用CQL create命令为节点或关系创建单个标签，并为节点创建多个标签。意味着Neo4j仅支持两个节点之间的单个关系类型

```
# 创建多个标签
CREATE (m:Movie:Cinema:Film:Picture)

# 创建单个标签关系
CREATE (p1:Profile1)-[r1:LIKES]->(p2:Profile2)

```

### WHERE子句
```
WHERE <condition>

WHERE <condition> <boolean-operator> <condition>

MATCH (emp:Employee) WHERE emp.name = 'Lokesh' RETURN emp

MATCH (cust:Customer),(cc:CreditCard) 
WHERE cust.id = "1001" AND cc.id= "5001" 
CREATE (cust)-[r:DO_SHOPPING_WITH{shopdate:"12/12/2014",price:55000}]->(cc) 
RETURN r

```
### 删除
DELETE <node-name-list>

MATCH (e: Employee) DELETE e

删除关系
MATCH (cc: CreditCard)-[rel]-(c:Customer) DELETE cc,c,rel
找出关系  ()-[]-() 这是关系写法

### 删除属性
时基于我们的客户端要求，我们需要向现有节点或关系添加或删除属性。
我们使用Neo4j CQL SET子句向现有节点或关系添加新属性。
我们使用Neo4j CQL REMOVE子句来删除节点或关系的现有属性。

```
CREATE (book:Book {id:122,title:"Neo4j Tutorial",pages:340,price:250}) 


```
![Markdown](http://i2.bvimg.com/654958/6182e357840de8cf.png)



### SET子句
有时，根据我们的客户端要求，我们需要向现有节点或关系添加新属性。
要做到这一点，Neo4j CQL提供了一个SET子句。
Neo4j CQL已提供SET子句来执行以下操作。
向现有节点或关系添加新属性
添加或更新属性值

MATCH (dc:DebitCard) SET dc.atm_pin = 3456 RETURN dc

### Sorting
match(n:UserD) return n.name,n.age order by n.age


### UNION
MATCH (cc:CreditCard)
RETURN cc.id as id,cc.number as number,cc.name as name,
   cc.valid_from as valid_from,cc.valid_to as valid_to
UNION ALL
MATCH (dc:DebitCard)
RETURN dc.id as id,dc.number as number,dc.name as name,
   dc.valid_from as valid_from,dc.valid_to as valid_to








## Admin 管理员






