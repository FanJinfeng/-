# 基于医疗知识图谱的问答系统操作介绍 学习笔记

> 文章编写人：Chris Van<br/>

## 目录

- [目录](#目录)
- [一、引言](#一引言)
- [二、运行环境](#二运行环境)
- [三、搭建知识图谱](#三搭建知识图谱)
- [四、启动问答测试](#四启动问答测试)
- [参考资料](#参考资料)

## 一、引言

- 该项目主要分为两部分：
  - 第一部分：搭建知识图谱，将在Task 3中进行学习。
  - 第二部分：启动问答测试，将在Task 4和Task 5中进行学习。
  
- 本节的核心目标：着眼全局，跑通项目。

## 二、运行环境

- python3.0及以上
- neo4j 3.5.0及以上（笔者装的是3.5.26）
- jdk 1.8.0

## 三、搭建知识图谱

1. 将disease.csv文件存放至项目路径下

2. 运行build_graph.py文件

- 将数据存放路径修改为自己的路径
- 修改数据库账号、密码
- 2000 years later打开Neo4J Web界面，可以看到导入的数据

3. 知识图谱结构介绍

- 7类实体：Disease, Alias, Symptom, Part, Department, Conplication, Drug
- 6类实体关系：ALIAS_IS, HAS_SYMPTOM, PART_IS, DEPARTMENT_IS, HAS_COMPLICATION, HAS_DRUG
- Disease的属性：age, insurance, infection, checklist, treatment, period, rate, money

## 四、启动问答测试

1. 运行kbqa_test.py文件

- 修改entity_extractor.py文件
  - from sklearn.externals import joblib修改为import joblib
  - 将数据存放路径修改为自己的路径（词向量文件需要自己下载哦！地址为：https://github.com/Embedding/Chinese-Word-Vectors）
  - 使用joblib重新dump两个模型生成.joblib文件（来自队友@Xiesy的解决方案！！！）

2. Joblib简介

```s
  joblib.dump(model, path)  # 将模型保存到本地
  jolib.load(path)  # 将模型从本地调回
```

## 参考资料 

1. [QASystemOnMedicalGraph](https://github.com/zhihao-chen/QASystemOnMedicalGraph)
