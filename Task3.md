# Task 3 Neo4j图数据库导入数据 学习笔记

> 文章编写人：Chris Van<br/>

## 目录

- [目录](#目录)
- [一、引言](#一引言)
- [二、Neo4j简介](#二neo4j简介)
  - [2.1 基本概念](#21-基本概念)
  - [2.2 索引](#22-索引)
  - [2.3 Neo4j的优势](#23-neo4j的优势)
- [2.4 环境部署](#24-环境部署)
  - [2.4.1 运行环境](#241-运行环境)
  - [2.4.2 neo4j安装及使用](#242-neo4j安装及使用)
- [三、Neo4j 数据导入](#三neo4j-数据导入)
  - [3.1 数据集简介](#31-数据集简介)
  - [3.2 主体类 MedicalGraph 介绍](#34-主体类-medicalgraph-介绍)
  - [3.3 主体类 MedicalGraph 中关键代码讲解](#35-主体类-medicalgraph-中关键代码讲解)
 - [四、多线程导入](#四多线程导入)
- [参考资料](#参考资料)

## 一、引言

- 图形数据结构：用于描述数据之间的复杂关系，表达为G(V,E)，可以在节点和边上增加键值对属性。
- 图数据库：应用图形数据结构的特点（节点、属性和边）存储数据实体和关系，Neo4J是当前较为主流的原生图数据库之一，提供原生的图数据存储、检索和处理。

## 二、Neo4j简介

### 2.1 基本概念

- 节点：节点表示实体，每个节点有唯一的ID，节点可以带有属性；
- 边：边表示关系，连接两个节点，分为有向和无向，边可以带有属性；
- 属性：键值对，存在于节点和关系中。

![图片.png](https://i.loli.net/2020/11/28/OGTbAcSw7UMdzHl.png)

### 2.2 索引

1. 是什么

索引是一种数据结构，可以提高数据库数据检索的速度。索引是通过属性创建的，Neo4J支持创建节点或关系属性上的索引。节点或关系上的属性越多，查找节点或关系的速度越快。每个索引通过key/value/object三个参数来工作，object是node或者relation。

2. 为什么

Neo4J使用遍历操作进行查询，为了加速查询，Neo4J会建立索引，并根据索引进行遍历，快速查找节点或者关系。

3. 怎么做

- 给节点的属性创建/删除索引

```s
    # 创建索引 CREATE INDEX ON :<label_name>(<property>)
    CREATE INDEX ON :Person(name)  # 给Label为Person的节点的name属性上创建索引
    
    # 删除索引 DROP INDEX ON :<label_name>(<property>)
    DROP INDEX ON :Person(name)
```

- 给关系的属性创建/删除索引
    - Neo4j索引对象可分为：基于relationship的索引和基于node的索引。neo4j本身即是关于relationship的索引实现，所以不用对关系的属性名创建索引。 所以一般说创建索引，都是说的针对节点的属性创建索引。

- 使用索引

```s
    # 索引创建完成后，执行一些查询语句时会自动使用索引
    MATCH (person:Person { name: 'Keanu Reeves' }) RETURN person
```

- 查看已经创建的索引

```s
    :schema
```

4. 什么时候使用索引

当 Neo4j 创建索引时，它会在数据库中创建冗余的副本，因此使用索引会占用更多的硬盘空间并减慢写入速度。一般来说，当某些节点数量很多或者你发现查询时间太长时，可以尝试通过添加索引来解决。

### 2.3 Neo4j的优势

- **查询的高性能**

使用图结构的自然伸展特性来设计**免索引邻近节点遍历的查询算法**，即图的遍历算法设计。图的遍历是图数据结构所具有的独特算法，即从一个节点开始，根据其连接的关系，可以快速和方便地找出它的邻近节点。这种查找数据的方法并不受数据量的大小所影响，因为邻近查询查找的始终是有限的局部数据，而不会对整个数据库进行搜索。

- **设计的灵活性**

图数据结构的自然伸展特性及其非结构化的数据格式，让 Neo4j的数据库设计可以具有很大的伸缩性和灵活性。随着需求的变化而增加的节点、关系及其属性并不会影响到原来数据的正常使用，所以使用Neo4j来设计数据库，可以更接近业务需求的变化。

- **开发的敏捷性**

图数据库设计中的数据模型，从需求的讨论开始，到程序开发和实现，以及最终保存在数据库中的样子，几乎没有什么变化，拉近了业务需求与系统设计之间的距离。

## 2.4 环境部署

### 2.4.1 运行环境

- python3.0及以上
- neo4j 3.5.0及以上（笔者安装的是3.5.26）
- jdk 1.8.0

### 2.4.2 neo4j安装及使用

1. 安装jdk 1.8.0

- 在官网下载 https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html
- 配置环境变量
- 命令行窗口查看是否安装成功 **java -version**

2. 安装Neo4J

- 在官网下载community版本 https://neo4j.com/
- 配置环境变量
- 命令行窗口查看是否安装成功**neo4j.bat console**
- 登录网址修改密码 http://localhost:7474/browser/

## 三、Neo4j 数据导入

### 3.1 数据集简介

- 数据源：39健康网。包括15项信息，其中7类实体，约3.7万实体，21万实体关系。

- 本次组队学习搭建的系统的知识图谱结构如下：

![知识图谱结构](https://i.loli.net/2020/11/28/TacFAJUnCWfuZXr.png)

- 知识图谱实体类型

![知识图谱实体类型](https://i.loli.net/2020/11/28/7zlp2Y9du3Mon1U.png)
	
- 知识图谱实体关系类型

![知识图谱实体关系类型](https://i.loli.net/2020/11/28/QI2tCpk3LuTfoca.png)

- 知识图谱疾病属性

![知识图谱疾病属性](https://i.loli.net/2020/11/28/y4repEsqd9bPkSl.png)

- 基于特征词分类的方法来识别用户查询意图

![基于特征词分类的方法来识别用户查询意图](https://i.loli.net/2020/11/28/r1ugnS3CPBWfmGk.png)

### 3.2 主体类MedicalGraph介绍

```s
class MedicalGraph:
    # 初始化
    def __init__(self):
        pass
    
    # 读取文件，获得实体，实体关系
    def read_file(self):
        psss
	
    # 创建不带有属性的节点
    def create_node(self, label, nodes):
        pass
	
    # 创建带有属性的节点
    def create_diseases_nodes(self, disease_info):
        pass
	
    # 创建知识图谱实体
    def create_graphNodes(self):
        pass
	
    # 创建实体关系边
    def create_graphRels(self):
        pass
	
    # 创建实体关系边
    def create_relationship(self, start_node, end_node, edges, rel_type, rel_name):
        pass
```
### 3.3 主体类MedicalGraph中关键代码讲解

- 初始化

```s
    def __init__(self):
        cur_dir = '/'.join(os.path.abspath(__file__).split('/')[:-1])  # 获取脚本完整路径，返回项目路径
        self.data_path = os.path.join(cur_dir, 'data/disease.csv')  # 数据存储在data文件中
        self.graph = Graph("http://localhost:7474", username="neo4j", password="******")  # 连接数据库
```

- 读取文件，获得实体，实体关系

```s
    def read_file(self):

        # cols = ["name", "alias", "part", "age", "infection", "insurance", "department", "checklist", "symptom",
        #         "complication", "treatment", "drug", "period", "rate", "money"]  # 数据文件字段
        
	# 实体
        diseases = []  # 疾病
        aliases = []  # 别名
        symptoms = []  # 症状
        parts = []  # 部位
        departments = []  # 科室
        complications = []  # 并发症
        drugs = []  # 药品

        # 疾病的属性：age, infection, insurance, checklist, treatment, period, rate, money
        diseases_infos = []
	
        # 关系
        disease_to_symptom = []  # 疾病与症状关系
        disease_to_alias = []  # 疾病与别名关系
        diseases_to_part = []  # 疾病与部位关系
        disease_to_department = []  # 疾病与科室关系
        disease_to_complication = []  # 疾病与并发症关系
        disease_to_drug = []  # 疾病与药品关系

        all_data = pd.read_csv(self.data_path, encoding='gb18030').loc[:, :].values  # 返回各个字段的值
	
        for data in all_data:
	    # 存储疾病的属性
            disease_dict = {}  
        
	    # 疾病
            disease = str(data[0]).replace("...", " ").strip()
            disease_dict["name"] = disease
            
	    # 别名
            line = re.sub("[，、；,.;]", " ", str(data[1])) if str(data[1]) else "未知"
            for alias in line.strip().split():
                aliases.append(alias)
                disease_to_alias.append([disease, alias])
            
	    # 部位
            part_list = str(data[2]).strip().split() if str(data[2]) else "未知"
            for part in part_list:
                parts.append(part)
                diseases_to_part.append([disease, part])
		
            # 年龄
            age = str(data[3]).strip()
            disease_dict["age"] = age
            
	    # 传染性
            infect = str(data[4]).strip()
            disease_dict["infection"] = infect
            
	    # 医保
            insurance = str(data[5]).strip()
            disease_dict["insurance"] = insurance
            
	    # 科室
            department_list = str(data[6]).strip().split()
            for department in department_list:
                departments.append(department)
                disease_to_department.append([disease, department])
            
	    # 检查项
            check = str(data[7]).strip()
            disease_dict["checklist"] = check
            
	    # 症状
            symptom_list = str(data[8]).replace("...", " ").strip().split()[:-1]
            for symptom in symptom_list:
                symptoms.append(symptom)
                disease_to_symptom.append([disease, symptom])
            
	    # 并发症
            complication_list = str(data[9]).strip().split()[:-1] if str(data[9]) else "未知"
            for complication in complication_list:
                complications.append(complication)
                disease_to_complication.append([disease, complication])
            
	    # 治疗方法
            treat = str(data[10]).strip()[:-4]
            disease_dict["treatment"] = treat
            
	    # 药品
            drug_string = str(data[11]).replace("...", " ").strip()
            for drug in drug_string.split()[:-1]:
                drugs.append(drug)
                disease_to_drug.append([disease, drug])
            
	    # 治愈周期
            period = str(data[12]).strip()
            disease_dict["period"] = period
            
	    # 治愈率
            rate = str(data[13]).strip()
            disease_dict["rate"] = rate
            
	    # 费用
            money = str(data[14]).strip() if str(data[14]) else "未知"
            disease_dict["money"] = money

            diseases_infos.append(disease_dict)

        return set(diseases), set(symptoms), set(aliases), set(parts), set(departments), set(complications), \
                set(drugs), disease_to_alias, disease_to_symptom, diseases_to_part, disease_to_department, \
                disease_to_complication, disease_to_drug, diseases_infos
```

- 创建不带有属性的节点

```s
    def create_node(self, label, nodes):
        count = 0
        for node_name in nodes:
            node = Node(label, name=node_name)  # 节点的type，节点的名称
            self.graph.create(node)  # 创建节点
            count += 1
            print(count, len(nodes))  # 记录节点的创建情况
        return
```

- 创建带有属性的节点

```s
    def create_diseases_nodes(self, disease_info):
        count = 0
        for disease_dict in disease_info:
            node = Node("Disease", name=disease_dict['name'], age=disease_dict['age'],
                        infection=disease_dict['infection'], insurance=disease_dict['insurance'],
                        treatment=disease_dict['treatment'], checklist=disease_dict['checklist'],
                        period=disease_dict['period'], rate=disease_dict['rate'],
                        money=disease_dict['money'])  # 节点的type，节点的名称，节点的属性
            self.graph.create(node)  # 创建节点
            count += 1
            print(count)  # 记录节点的创建情况
        return
```

- 创建边

```s
    def create_relationship(self, start_node, end_node, edges, rel_type, rel_name):
        count = 0
	
        # 去重
        set_edges = []
        for edge in edges:
            set_edges.append('###'.join(edge))   # 用###连接edge中的两个元素
        all = len(set(set_edges))  # 去重后的length
        
	for edge in set(set_edges):
            edge = edge.split('###')  # 用###分隔出两个元素
            p = edge[0]  # start_node
            q = edge[1]  # end_node
            query = "match(p:%s),(q:%s) where p.name='%s'and q.name='%s' create (p)-[rel:%s{name:'%s'}]->(q)" % (
                start_node, end_node, p, q, rel_type, rel_name)
            try:
                self.graph.run(query)  # 创建边
                count += 1
                print(rel_type, count, all)  # 记录边的创建情况
            except Exception as e:
                print(e)
        return
```

- 创建知识图谱实体

```s
    def create_graphNodes(self):
        # 创建节点
        disease, symptom, alias, part, department, complication, drug, rel_alias, rel_symptom, rel_part, \
        rel_department, rel_complication, rel_drug, rel_infos = self.read_file()
        self.create_diseases_nodes(rel_infos)
        self.create_node("Symptom", symptom)
        self.create_node("Alias", alias)
        self.create_node("Part", part)
        self.create_node("Department", department)
        self.create_node("Complication", complication)
        self.create_node("Drug", drug)
        return

    def create_graphRels(self):
        # 创建边
        disease, symptom, alias, part, department, complication, drug, rel_alias, rel_symptom, rel_part, \
        rel_department, rel_complication, rel_drug, rel_infos = self.read_file()
        self.create_relationship("Disease", "Alias", rel_alias, "ALIAS_IS", "别名")
        self.create_relationship("Disease", "Symptom", rel_symptom, "HAS_SYMPTOM", "症状")
        self.create_relationship("Disease", "Part", rel_part, "PART_IS", "发病部位")
        self.create_relationship("Disease", "Department", rel_department, "DEPARTMENT_IS", "所属科室")
        self.create_relationship("Disease", "Complication", rel_complication, "HAS_COMPLICATION", "并发症")
        self.create_relationship("Disease", "Drug", rel_drug, "HAS_DRUG", "药品")
```

## 四、多线程导入

### 4.1 CPU的内存结构和多线程

1. CPU和各级缓存(cache)、内存/主存(memory)、硬盘(disk)之间的关系

[!image.jpg](https://pic1.zhimg.com/v2-f7df2460ef1d2af17bbf1b2a9d6bb550_r.jpg)

- 为什么会出现多级缓存？
    - CPU的频率太快了，直接读取内存中的数据又太慢了，我们不想让 CPU 停下来等待，所以加入了一层读取速度大于内存但小于 CPU 的东西，这就是缓存。
    - CPU 需要数据就问缓存要，缓存没有就从主存中读取，并保留一份在缓存中。下次读取就从缓存中读取，加快速度。（数据的保存和读取都要在主存上进行）
    - CPU 运行速度快（电），缓存次之（保存内部存储数据不需要刷新电路），而内存慢一点（保存内部存储数据需要刷新电路），硬盘最慢（磁头移动+电信号）。

2. 多线程

[!image.jpg](https://pic2.zhimg.com/v2-c8c982aa5384854a804ab3a5a57488f5_r.jpg)

- 面临的问题
    - 每个线程都有自己的缓存，假如线程 1 从主存中读取到 x，并对其加 1 ，此时还没有写回主存，线程 2 也从主存中读取 x ，并加 1 ，它们是不知道对方的，也不可以读取对方的缓存，这时都将 x 写回主存，那此时 x 的值就少了 1 。

- 解决方法
    - 指定协议，规定不同的线程在读写主存数据时遵守某种规则，以保证不会出现数据不一致的问题。
    - 例如MESI协议，定义了 cache line 的四种状态，而线程对 cache line 的四种操作可能会产生不一致的状态，因此缓存控制器监听到本地操作和远程操作的时候，需要对地址一致的 cache line 状态进行一致性修改，从而保证数据在多个缓存之间保持一致性。(M: modified E: Exclusive S: shared I: invalid)。

### 4.2 多进程导入数据

```s
    from multiprocessing import Process
    def create_graphNodes(self):
        disease, symptom, alias, part, department, complication, drug, rel_alias, rel_symptom, rel_part, \
	rel_department, rel_complication, rel_drug, rel_infos = self.read_file()

	p1 = Process(target=self.create_node,args=("Symptom", symptom))
	p2 = Process(target=self.create_node,args=("Alias", alias))
	p3 = Process(target=self.create_node,args=("Part", part))
	p4 = Process(target=self.create_node,args=("Complication", complication))
	p5 = Process(target=self.create_node,args=("Drug", drug))
	p6 = Process(target=self.create_diseases_nodes,args=([rel_infos]))

	p1.start()
	p2.start()
	p3.start()
	p4.start()
	p5.start()
	p6.start()
	p1.join()
	p2.join()
	p3.join()
	p4.join()
	p5.join()
	p6.join()
```

## 参考资料 

1. [QASystemOnMedicalGraph](https://github.com/zhihao-chen/QASystemOnMedicalGraph)
2. [韩浩明.图数据库系统研究综述[J].计算机光盘软件与应用,2014,17(23):14-15.](http://gb.oversea.cnki.net/KCMS/detail/detail.aspx?filename=GPRJ201423018&dbcode=CJFD&dbname=CJFD2014) 
3. [neo4j各种索引](https://www.jianshu.com/p/91f3e214b47e)
