# Task 1 知识图谱介绍 学习笔记

> 文章编写人：Chris Van<br/>

## 目录

- [目录](#目录)
- [一、知识图谱简介](#一知识图谱简介)
  - [1.1 是什么？](#11-是什么？)
  - [1.2 为什么？](#12-为什么？)
  - [1.3 怎么做？](#13-怎么做？)
- [二、Neo4J 实战](#五neo4j-实战)
  - [5.1 引言](#51-引言)
  - [5.2 创建节点](#52-创建节点)
  - [5.3 创建关系](#53-创建关系)
  - [5.4 创建 出生地关系](#54-创建-出生地关系)
  - [5.5 图数据库查询](#55-图数据库查询)
  - [5.6 删除和修改](#56-删除和修改)
- [六、通过 Python 操作 Neo4j](#六通过-python-操作-neo4j)
  - [6.1 neo4j模块：执行CQL ( cypher ) 语句](#61-neo4j模块执行cql--cypher--语句)
  - [6.2 py2neo模块：通过操作python变量，达到操作neo4j的目的](#62-py2neo模块通过操作python变量达到操作neo4j的目的)
- [七、通过csv文件批量导入图数据](#七通过csv文件批量导入图数据)
- [参考资料](#参考资料)


## 一、知识图谱简介

### 1.1 是什么？

本质上是语义网络（Senmantic Network）的知识库，可以简单理解成多关系图（Multi-relational Graph）。

多关系图：包含多种类型的节点（Vertex）和多种类型的边（Edge）,节点代表现实世界中的事物实体，边则代表实体之间的某种联系。

模式（Schema）：限定了知识图谱的数据格式，包括实体对象（Thing）及其值的类型（DataType），实体对象是某个领域内有意义的概念集合（举例如下）。
![Schema定义.PNG](https://i.loli.net/2020/12/06/zxMLupBIewb2jFR.png)

### 1.2 为什么？

人工智能->知识工程->知识表示->知识图谱，知识图谱是人工智能很重要的一个细分领域。

### 1.3 怎么做？

涉及的NLP技术：
 
  1. 实体命名识别（Name Entity Recognition）：从文本中提取出实体并进行打标。
  2. 关系抽取（Relation Extraction）：从文本中提取出实体之间的关系。
  3. 实体统一（Entity Resolution）：将文本中表述不同的实体进行合并。
  4. 指代消解（Coreference Resolution）：确定文本中的代词指向的实体。
  5. ...

储存方式：

  1. RDF：学术界场景，形式上表示为SPO（Subject, Predicate, Object）三元组，推荐使用Jena。
![image.jpg](https://pic2.zhimg.com/v2-e3478e02c36ead3875e598b0668830fd_r.jpg)

    RDF由节点和边组成，节点表示实体、属性，边表示实体之间以及实体和属性之间的关系。

  2. 图数据库：工业界场景，形式上表示为属性图，推荐使用Neo4j。

    属性图由节点和边组成，节点和边可以带有属性。

## 二、Neo4J简介

### 2.1 是什么？

- Neo4J Web界面

  Neo4J提供了一个用户友好的 Web 界面，可以进行各项配置、写入、查询等操作，并且提供了可视化功能。

- Cypher查询语言

  Neo4J的声明式图形查询语言，类似于传统数据库的SQL，允许用户不必编写图形结构的遍历代码，对图形数据进行高效的查询。

### 2.2 Neo4J 实战

#### 案例介绍

节点包人物和城市，人物关系包括朋友、夫妻，人物的城市的关系包括出生地。

- Person-Friends-Person
- Person-Married-Person
- Person-Born_in-Location

#### Step 1 创建节点

1. 删除数据库中历史图

```s
  MATCH (n) DETACH DELETE n
```
MATCH表示匹配操作，()表示节点，n为节点的标识符，DETACH DELETE表示删除节点的同时并删除其所有关系。

2. 创建人物节点

```s
  CREATE (n:Person {name:'Sally'});
  CREATE (n:Person {name:'Steve'});
  CREATE (n:Person {name:'Mike'});
  CREATE (n:Person {name:'Liz'});
  CREATE (n:Person {name:'Shawn'});
  CREATE (n:Person {name:'John'});
```

CREATE表示创建操作，()表示节点，n为节点的标识符，
Person表示标签（代表节点属于哪一个集体，一个节点可以有0个或多个标签），
{}代表属性（描述节点或关系的属性），name为节点Person的属性，John为属性值，
RETURN表示返回操作。

3. 创建地区节点

```s
  CREATE (n:Location {city:'Miami', state:'FL'});
  CREATE (n:Location {city:'Boston', state:'MA'});
  CREATE (n:Location {city:'Lynn', state:'MA'});
  CREATE (n:Location {city:'Portland', state:'ME'});
  CREATE (n:Location {city:'San Francisco', state:'CA'});
```

节点Location有city和state属性。

#### Step 2 创建同类型节点关系

```s
  MATCH (a:Person {name:'Shawn'}), (b:Person {name:'Sally'}) MERGE (a)-[:FRIENDS {since:2001}]->(b);
  MATCH (a:Person {name:'Shawn'}), (b:Person {name:'John'}) MERGE (a)-[:FRIENDS {since:2012}]->(b);
  MATCH (a:Person {name:'Mike'}), (b:Person {name:'Shawn'}) MERGE (a)-[:FRIENDS {since:2006}]->(b);
  MATCH (a:Person {name:'Sally'}), (b:Person {name:'Steve'}) MERGE (a)-[:FRIENDS {since:2006}]->(b);
  MATCH (a:Person {name:'Liz'}), (b:Person {name:'John'}) MERGE (a)-[:MARRIED {since:1998}]->(b);
```

MATCH表示匹配操作，MERGE表示匹配已存在的节点（如果节点不存在就创建新的并绑定它，可以理解为match和create的组合），
[]表示关系，FRIENDS和MARRIED表示标签（代表关系的类型），-->表示有向图，--表示无向图，
{}代表属性，since为关系FRIENDS的属性，2012为属性值。

#### Step 3 创建不同类型节点关系

```s
  MATCH (a:Person {name:'John'}), (b:Location {city:'Boston'}) MERGE (a)-[:BORN_IN {year:1978}]->(b);
  MATCH (a:Person {name:'Liz'}), (b:Location {city:'Boston'}) MERGE (a)-[:BORN_IN {year:1981}]->(b);
  MATCH (a:Person {name:'Mike'}), (b:Location {city:'San Francisco'}) MERGE (a)-[:BORN_IN {year:1960}]->(b);
  MATCH (a:Person {name:'Shawn'}), (b:Location {city:'Miami'}) MERGE (a)-[:BORN_IN {year:1960}]->(b);
  MATCH (a:Person {name:'Steve'}), (b:Location {city:'Lynn'}) MERGE (a)-[:BORN_IN {year:1970}]->(b);
```

  注：创建节点的同时建立关系
  ```s
  CREATE (a:Person {name:'Todd'})-[r:FRIENDS]->(b:Person {name:'Carlos'});
  ```
![image.png](https://github.com/FanJinfeng/KnowledgeGraph/blob/main/graph1.png)

> 节点和关系可视化展示

#### Step 4 图数据库查询、删除修改

1. 查询所有在Boston出生的人物

```s
  MATCH (a:Person)-[:BORN_IN]->(b:Location {city:'Boston'}) RETURN a,b
```

MATCH表示匹配操作，RETURN表示返回操作。

2. 查询所有有关系的节点

```s
  MATCH (a)--()  RETURN a
```

3. 查询所有对外有关系的节点，以及关系类型

```s
  MATCH (a)-[r]->() RETURN a.name，type(r)
```

r为关系的标识符。


4. 查询所有有结婚关系的节点

```s
  MATCH (n)-[:MARRIED]-() RETURN n
```

5. 查找某人的朋友的朋友

```s
  MATCH (a:Person {name:'Mike'})-[r1:FRIENDS]-()-[r2:FRIENDS]-(friend_of_a_friend) RETURN friend_of_a_friend.name AS fofName
```

AS指定返回查询结果的名称。

7. 增加/修改节点的属性

```s
  MATCH (a:Person {name:'Liz'}) SET a.age=34
  MATCH (a:Person {name:'Shawn'}) SET a.age=32
  MATCH (a:Person {name:'John'}) SET a.age=44
  MATCH (a:Person {name:'Mike'}) SET a.age=25
```

SET表示修改操作，a.age表示节点的age属性。

8. 删除节点的属性

```s
  MATCH (a:Person {name:'Mike'}) SET a.test='test'
  MATCH (a:Person {name:'Mike'}) REMOVE a.test
```

REMOVE表示删除属性操作，a.test表示节点的test属性。

9. 删除节点

```s
  MATCH (a:Location {city:'Portland'}) DELETE a
```

DELETE表示删除节点操作。

10. 删除有关系的节点

```s
  MATCH (a:Person {name:'Todd'})-[rel]-(b:Person) DELETE a,b,rel
```

DELETE表示删除节点和关系的操作。

## 三、通过Python操作Neo4J

### 3.1 neo4j-driver

Python版本的Neo4J的驱动程序，在Python中使用Cypher来操作图数据库。

```s
  # step 1：导入 Neo4j 驱动包
  from neo4j import GraphDatabase
  # step 2：连接 Neo4j 图数据库
  driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "password"))
  # 添加 关系 函数
  def add_friend(tx, name, friend_name):
      tx.run("MERGE (a:Person {name: $name}) "
            "MERGE (a)-[:KNOWS]->(friend:Person {name: $friend_name})",
            name=name, friend_name=friend_name)
  # 定义 关系函数
  def print_friends(tx, name):
      for record in tx.run("MATCH (a:Person)-[:KNOWS]->(friend) WHERE a.name = $name "
                          "RETURN friend.name ORDER BY friend.name", name=name):
          print(record["friend.name"])
  # step 3：运行
  with driver.session() as session:
      session.write_transaction(add_friend, "Arthur", "Guinevere")
      session.write_transaction(add_friend, "Arthur", "Lancelot")
      session.write_transaction(add_friend, "Arthur", "Merlin")
      session.read_transaction(print_friends, "Arthur")
```

上述程序的核心部分，抽象一下就是：

```s
  neo4j.GraphDatabase.driver(xxxx).session().write_transaction(函数(含tx.run(CQL语句)))
```
或者

```s
  neo4j.GraphDatabase.driver(xxxx).session().begin_transaction.run(CQL语句)
```
### 3.2 py2neo

Python版本的Neo4J的驱动程序，其可以直接使用类似Python语法操作图数据库。

```s 
  # step 1：导包
  from py2neo import Graph, Node, Relationship
  # step 2：构建图
  g = Graph()
  # step 3：创建节点
  tx = g.begin()
  a = Node("Person", name="Alice")
  tx.create(a)
  b = Node("Person", name="Bob")
  # step 4：创建边
  ab = Relationship(a, "KNOWS", b)
  # step 5：运行
  tx.create(ab)
  tx.commit()
```
py2neo模块符合python的习惯，写着感觉顺畅，其实可以完全不会CQL也能写

## 四、通过csv文件批量导入图数据

前面学习的是单个创建节点，不适合大批量导入。这里我们介绍使用neo4j-admin import命令导入，适合部署在docker环境下的neo4j。
其他导入方法也可以参考[Neo4j之导入数据](https://zhuanlan.zhihu.com/p/93746655)

csv分为两个nodes.csv和relations.csv，注意关系里的起始节点必须是在nodes.csv里能找到的：

```s
  # nodes.csv需要指定唯一ID和nam,
  headers = [
  'unique_id:ID', # 图数据库中节点存储的唯一标识
  'name', # 节点展示的名称
  'node_type:LABEL', # 节点的类型，比如Person和Location
  'property' # 节点的其他属性
  ]
```

```s
  # relations.csv
  headers = [
  'unique_id', # 图数据库中关系存储的唯一标识
  'begin_node_id:START_ID', # begin_node和end_node的值来自于nodes.csv中节点
  'end_node_id:END_ID',
  'begin_node_name',
  'end_node_name',
  'begin_node_type',
  'end_node_type',
  'relation_type:TYPE', # 关系的类型，比如Friends和Married
  'property' # 关系的其他属性
  ]
```
制作出两个csv后，通过以下步骤导入neo4j:

1. 两个文件nodes.csv ，relas.csv放在

```s
  neo4j安装绝对路径/import
```
2. 导入到图数据库mygraph.db

```s
  neo4j bin/neo4j-admin import --nodes=/var/lib/neo4j/import/nodes.csv --relationships=/var/lib/neo4j/import/relas.csv   --delimiter=^ --database=xinfang*.db
```
delimiter=^ 指的是csv的分隔符

3. 指定neo4j使用哪个数据库

```s
  修改 /root/neo4j/conf/neo4j.conf 文件中的 dbms.default_database=mygraph.db
```

4. 重启neo4j就可以看到数据已经导入成功了

## 参考资料

1. [Datawhale 知识图谱组队学习 之 Task 1 知识图谱介绍](https://github.com/datawhalechina/team-learning-nlp/blob/master/KnowledgeGraph_Basic/task01.md)
