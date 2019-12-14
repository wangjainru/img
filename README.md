# 爱宠*你的选择



## 项目介绍：

本项目是第一组的研究成果。
该课题是第一组以实现萌宠了解为目标，探索生活中的乐趣的基础上基于碎片化宠物数据，构建面向爱宠人士的的知识图谱及其应用系统。

>
> 项目组：第一组
>
> 参与成员：
>
>| Title               | Name | 
>| ------------------- | ---- | 
>| Associate Professor | 刘斌龙  | 
>| Master              | 殷青栋  | 
>| Master              | 谷泽坤  | 
>| Master              | 崔登虎  | 
>| Master              | 李新宇  | 
>| Master              | 王坚如  | 



## 目录结构：

```
.
├── MyCrawler      // scrapy爬虫项目路径(已爬好)可直接调用
│   └── MyCrawler
│       ├── data
│       └── spiders
├── data\ processing    // 数据清洗(已无用)
│   └── data
├── demo     // django项目路径
│   ├── Model  // 模型层，用于封装Item类，以及neo4j和csv的读取
│   ├── demo   // 用于写页面的逻辑(View)
│   ├── label_data    // 标注训练集页面的保存路径
│   │   └── handwork
│   ├── static    // 静态资源
│   │   ├── css
│   │   ├── js
│   │   └── open-iconic
│   ├── templates   // html页面
│   └── toolkit   // 工具库，包括预加载，命名实体识别
│   └── KNN_predict   
├── KNN_predict    // KNN算法预测标签分类
└── baidudataSpider    //  爬取baidu中的关系
```



## 可复用资源
attributes.csv 萌宠实体及关系
data_relation2.csv 萌宠实体及关系
baidu_pedia.csv 萌宠信息
new_node.csv 萌宠实体



## 项目配置

**0.安装基本环境：**

确保安装好python3和Neo4j（任意版本）
 
安装一系列pip依赖： cd至项目根目录，运行 sudo pip3 install -r requirement.txt

**1.导入数据：**

将baidu_pedia.csv导入neo4j：开启neo4j，进入neo4j控制台。将baidu_pedia.csv放入neo4j安装目录下的/import目录。在控制台依次输入：

```
// 将baidu_pedia.csv 导入
LOAD CSV WITH HEADERS  FROM "file:///baidu_pedia.csv" AS line
CREATE (p:BaiDuItem{title:line.title,image:line.image,detail:line.detail,url:line.url,baseInfoKeyList:line.baseInfoKeyList,baseInfoValueList:line.baseInfoValueList})


// 创建索引
CREATE CONSTRAINT ON (c:BaiDuItem)
ASSERT c.title IS UNIQUE
```

以上两步的意思是，将baidu_pedia.csv导入neo4j作为结点，然后对titile属性添加UNIQUE（唯一约束/索引）

*（如果导入的时候出现neo4j jvm内存溢出，可以在导入前，先把neo4j下的conf/neo4j.conf中的dbms.memory.heap.initial_size 和dbms.memory.heap.max_size调大点。导入完成后再把值改回去）*



```
// 导入新的节点
LOAD CSV WITH HEADERS FROM "file:///new_node.csv" AS line
CREATE (:NewNode { title: line.title })

//添加索引
CREATE CONSTRAINT ON (c:NewNode)
ASSERT c.title IS UNIQUE

//导入data_relation2.csv和新加入节点之间的关系
LOAD CSV  WITH HEADERS FROM "file:///data_relation2.csv" AS line
MATCH (entity1:BaiDuItem{title:line.BaiDuItem}) , (entity2:NewNode{title:line.NewNode})
CREATE (entity1)-[:RELATION { type: line.relation }]->(entity2)


LOAD CSV  WITH HEADERS FROM "file:///data_relation.csv" AS line
MATCH (entity1:BaiDuItem{title:line.BaiDuItem1}) , (entity2:BaiDuItem{title:line.BaiDuItem2})
CREATE (entity1)-[:RELATION { type: line.relation }]->(entity2)

**导入实体属性(数据来源: 互动百科)**

将attributes.csv放到neo4j的import目录下，然后执行


LOAD CSV WITH HEADERS FROM "file:///attributes.csv" AS line
MATCH (entity1:BaiDuItem{title:line.Entity}), (entity2:BaiDuItem{title:line.Attribute})
CREATE (entity1)-[:RELATION { type: line.AttributeName }]->(entity2);


LOAD CSV WITH HEADERS FROM "file:///attributes.csv" AS line
MATCH (entity1:BaiDuItem{title:line.Entity}), (entity2:NewNode{title:line.Attribute})
CREATE (entity1)-[:RELATION { type: line.AttributeName }]->(entity2); 

//我们建索引的时候带了label，因此只有使用label时才会使用索引，这里我们的实体有两个label，所以一共做2*2=4次。当然，可以建立全局索引，即对于不同的label使用同一个索引
                                                            
          
                                                                                                                         
```




以上步骤是导入爬取到的关系


**2.下载词向量模型：（如果只是为了运行项目，步骤2可以不做，预测结果已经离线处理好了）**
 



**3.修改Neo4j用户**

进入demo/Model/neo_models.py,修改第9行的neo4j账号密码，改成你自己的

**4.启动服务**

进入demo目录，然后运行脚本：

```
sudo sh django_server_start.sh
```

这样就成功的启动了django。我们进入8000端口主页面，输入文本，即可看到以下命名实体和分词的结果（确保django和neo4j都处于开启状态）



###  (update 2019.12.13) 
- 修改部分配置信息
- 关系查询中，添加了2个实体间的最短路查询，从而挖掘出实体之间一些奇怪的隐含关系

### 萌宠实体识别+实体分类

点击实体的超链接，可以跳转到词条页面（词云采用了词向量技术）：
![](https://github.com/wangjainru/img/blob/master/%E5%9B%BE%E7%89%87/QQ%E5%9B%BE%E7%89%8720191214094456.png)


### 实体查询

实体查询部分，我们能够搜索出与某一实体相关的实体，以及它们之间的关系：
![](https://github.com/wangjainru/img/blob/master/%E5%9B%BE%E7%89%87/QQ%E5%9B%BE%E7%89%8720191213202911.png)


### 关系查询

关系查询即查询三元组关系entity1-[relation]->entity2 , 分为如下几种情况:

* 指定第一个实体entity1
* 指定第二个实体entity2
* 指定第一个实体entity1和关系relation
* 指定关系relation和第二个实体entity2
* 指定第一个实体entity1和第二个实体entity2
* 指定第一个实体entity1和第二个实体entity2以及关系relation

下图所示，是指定关系relation和第二个实体entity2的查询结果
![](https://github.com/wangjainru/img/blob/master/%E5%9B%BE%E7%89%87/QQ%E5%9B%BE%E7%89%8720191214094551.png)



### 知识的树形结构

萌宠知识概览部分，我们能够列出某一宠物分类下的词条列表，这些概念以树形结构组织在一起：




**使用方法**: 启动neo4j,mongodb之后，进入demo目录，启动django服务，进入127.0.0.1:8000/tagging即可使用


#### 分类器：KNN算法

- 无需表示成向量，比较相似度即可
- K值通过网格搜索得到

#### 定义两个页面的相似度sim(p1,p2)：

- 
  title之间的词向量的余弦相似度(利用fasttext计算的词向量能够避免out of vocabulary)
- 2组openType之间的词向量的余弦相似度的平均值
- 相同的baseInfoKey的IDF值之和（因为‘中文名’这种属性贡献应该比较小）
- 相同baseInfoKey下baseInfoValue相同的个数
- 预测一个页面时，由于KNN要将该页面和训练集中所有页面进行比较，因此每次预测的复杂度是O(n)，n为训练集规模。在这个过程中，我们可以统计各个分相似度的IDF值，均值，方差，标准差，然后对4个相似度进行标准化:**(x-均值)/方差**
- 上面四个部分的相似度的加权和为最终的两个页面的相似度，权值由向量weight控制，通过10折叠交叉验证+网格搜索得到


### Labels：（命名实体的分类）

| Label | NE Tags                                  | Example                                  |
| ----- | ---------------------------------------- | ---------------------------------------- |
| 0     | 狗                             |'柯基犬', '金毛犬', '法国斗牛犬'                    |
| 1     | 猫                             |'缅因猫', '布偶猫', '暹罗猫'                        |
| 2     | 老鼠                           |'仓鼠', '布丁仓鼠', '龙猫'                          |
| 3     | 兔子                           |'伊拉兔', '荷兰垂耳兔', '荷兰侏儒兔'                |
| 4     | 鱼                             |'小丑鱼', '龙鱼', '鹦鹉鱼', '锦鲤鱼'                |
| 5     | 鸟                             |'八哥鸟', '金丝雀', '百灵鸟'                        |
| 6     | 龟                             |'缅甸陆龟', '黄头侧颈龟', '大鳄龟'                  |
| 7     | 蛇                             |'赤练蛇', '喜玛拉雅白头蛇', '太攀蛇'                |
| 8     | 蜘蛛                           |'智利火玫瑰', '泰国金属蓝', '红玫瑰蜘蛛'            |
| 9     | 蜥蜴                           |'绿鬣蜥', '鬃狮蜥', '中华石龙子'                    |
| 10    | 鳄鱼                           |'泰鳄', '凯门鳄', '侏儒凯门鳄'                      |
| 11    | 猴                             |'松鼠猴', '狨猴', '指猴'                            |
| 12    | 猪                             |'小香猪', '越南大肚猪', '胡利亚尼猪'                |
| 13    | 狐狸                           |'蓝狐', '银狐', '大耳狐'                            |
| 14    | 貂                             |'貂', '蒙眼貂', '东方色貂'                          |
| 15    | 稀有                           |'欧洲红松鼠', '岩松鼠', '雪地松鼠'                  |



### 关系抽取

使用远程监督方法构建数据集，利用tensorflow训练PCNN模型













