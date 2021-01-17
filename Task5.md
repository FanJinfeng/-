# Task 5 Neo4j 图数据库查询

> 文章编写人：Chris Van<br/>

## 目录

- [目录](#目录)
- [一、 Neo4J 介绍](#一-neo4j-介绍)
- [1.1 Neo4J 介绍](#11-neo4j-介绍)
- [1.2 Cypher 介绍](#31-cypher-介绍)
- [1.3 Neo4J 图数据库 查询](#32-neo4j-图数据库-查询)
- [二、 主体类 AnswerSearching 代码注解](#四-主体类-answersearching-代码注解)
- [参考资料](#参考资料)

## 一、 Neo4介绍

### 1.1 Neo4介绍

在Neo4j中，节点和边都可以包含属性；可以为节点设置零或多个标签（例如Author或Book）；每个关系都对应一种类型（例如WROTE或FRIEND_OF）；关系总是从一个节点指向另一个节点，但可以在不考虑指向性的情况下进行查询。

### 1.2 Cypher 介绍

作为Neo4j的查询语言，“Cypher”是一个描述性的图形查询语言，一个人类查询语言，一个申明式的语言。

### 1.3 Neo4j 图数据库 查询

查询乙肝的症状
```s
    MATCH (d:Disease)-[:HAS_SYMPTOM]->(s) WHERE d.name='乙肝' RETURN d.name,s.name
```

## 二、 主体类 AnswerSearching 代码注解

1. 主体类框架
```s
class AnswerSearching:

    # 初始化
    def __init__(self):
        pass
	
    # 主要是根据不同的实体和意图构造cypher查询语句
    def question_parser(self, data):
        pass
	
    # 将问题转变为cypher查询语句
    def transfor_to_sql(self, label, entities, intent):
        pass
	
    # 执行cypher查询，返回结果
    def searching(self, sqls):
        pass
	
    # 根据不同意图，返回不同模板的答案
    def answer_template(self, intent, answers):
        pass
```

2. 代码注解

```s
    # 根据不同的实体和意图构造cypher查询语句
    def question_parser(data):
        """
        :param data: {"Disease":[], "Alias":[], "Symptom":[], "Complication":[]}
        :return:
        """
        sqls = []  # 存储所有的查询语句
        if data:
            for intent in data["intentions"]:
                sql_ = {}
                sql_["intention"] = intent
                sql = []
                if data.get("Disease"):
                   sql = transfor_to_sql("Disease", data["Disease"], intent)  # 实体标签，实体列表，查询意图
                elif data.get("Alias"):
                    sql = transfor_to_sql("Alias", data["Alias"], intent)
                elif data.get("Symptom"):
                    sql = transfor_to_sql("Symptom", data["Symptom"], intent)
                elif data.get("Complication"):
                    sql = transfor_to_sql("Complication", data["Complication"], intent)
		    
                if sql:
                    sql_['sql'] = sql
                    sqls.append(sql_)
        return sql
```

```s
    # 将问题转变为cypher查询语句
    def transfor_to_sql(label, entities, intent):
    
        if not entities:
            return []
	    
        sql = []
        # 查询症状，返回疾病和症状
        if intent == "query_symptom" and label == "Disease":
            sql = ["MATCH (d:Disease)-[:HAS_SYMPTOM]->(s) WHERE d.name='{0}' RETURN d.name,s.name".format(e)
                   for e in entities]
		   
        # 查询治疗方法，返回疾病和治疗方法
        if intent == "query_cureway" and label == "Disease":
            sql = ["MATCH (d:Disease)-[:HAS_DRUG]->(n) WHERE d.name='{0}' return d.name,d.treatment," \
                   "n.name".format(e) for e in entities]
		   
         # 查询治疗周期，返回疾病和治疗周期
        if intent == "query_period" and label == "Disease":
            sql = ["MATCH (d:Disease) WHERE d.name='{0}' return d.name,d.period".format(e) for e in entities
	  
        # 查询疾病（已知疾病别名或者症状）
        if intent == "query_disease" and label == "Alias":
            sql = ["MATCH (d:Disease)-[]->(s:Alias) WHERE s.name='{0}' return " \
                   "d.name".format(e) for e in entities]
        if intent == "query_disease" and label == "Symptom":
            sql = ["MATCH (d:Disease)-[]->(s:Symptom) WHERE s.name='{0}' return " \
                   "d.name".format(e) for e in entities]
        ...
```

```s
    # 执行cypher查询，返回结果
    def searching(sqls):

        final_answers = []
        for sql_ in sqls:
            intent = sql_['intention']
            queries = sql_['sql']
            answers = []
            for query in queries:
                ress = graph.run(query).data()  # 返回查询的结果
                answers += ress
            final_answer = answer_template(intent, answers)  # 格式化输出
	    
            if final_answer:
                final_answers.append(final_answer)
        return final_answers
```

```s
    # 根据不同意图，返回不同模板的答案
    def answer_template(intent, answers):

        final_answer = ""
        if not answers:
            return ""
	
        # 查询症状
        if intent == "query_symptom":
            disease_dic = {}
            for data in answers:
                d = data['d.name']  # 疾病
                s = data['s.name']  # 症状
                if d not in disease_dic:
                    disease_dic[d] = [s]
                else:
                    disease_dic[d].append(s)
            i = 0
            for k, v in disease_dic.items():
                if i >= 10:  # 仅显示前10个疾病
                    break
                final_answer += "疾病 {0} 的症状有：{1}\n".format(k, ','.join(list(set(v))))
                i += 1
		
        # 查询疾病
        if intent == "query_disease":
            disease_freq = {}
            for data in answers:
                d = data["d.name"]  # 疾病
                disease_freq[d] = disease_freq.get(d, 0) + 1  # 统计频数
            n = len(disease_freq.keys())
            freq = sorted(disease_freq.items(), key=lambda x: x[1], reverse=True)  # 根据频数倒序排列
            for d, v in freq[:10]:
                final_answer += "疾病为 {0} 的概率为：{1}\n".format(d, v/10)  # 除以10真的科学吗？？？
            ...
```
## 参考资料 
1. [ QASystemOnMedicalGraph](https://github.com/zhihao-chen/QASystemOnMedicalGraph)
