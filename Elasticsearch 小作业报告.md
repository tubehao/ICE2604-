

# Elasticsearch 小作业报告

[TOC]









 ## part1

### Problem1

对于提供的词对：

- a. abandon/abandonment - 这两个词不应被合并。"abandon"是一个动词，表示放弃；而"abandonment"是一个名词，表示放弃的状态或结果。它们虽然词根相同，但意义和用途不同。
- b. absorbency/absorbent - 这对词不应合并。"absorbency"指的是一个物体吸收液体的能力，是名词；"absorbent"是形容词，用来描述具有吸收性的物体。尽管它们都和吸收有关，但一个是性质的描述，另一个是能力的描述。
- c. marketing/markets - 这对词不应该合并。"marketing"指市场营销活动，"markets"通常指市场本身。一个是活动过程，另一个是地点或概念。
- d. university/universe - 这对词不应该合并。"university"指的是高等教育机构，而"universe"指的是宇宙。它们的意义和用途完全不同。
- e. volume/volumes - 这对词通常可以合并，因为它们只是单数和复数形式的区别。但如果在特定上下文中（如物理学中表示体积的"volume"），这种合并可能会导致信息丢失。
- 可以发现，词对如果有相同的含义但不同的词性或者含义不同都是不可以合并的。

#### a. Term-Document Incidence Matrix

词项-文档出现矩阵如下所示：

| Term      | Doc 1 | Doc 2 | Doc 3 | Doc 4 |
| --------- | ----- | ----- | ----- | ----- |
| new       | 1     | 0     | 0     | 1     |
| home      | 1     | 1     | 1     | 1     |
| sales     | 1     | 1     | 1     | 1     |
| top       | 1     | 0     | 0     | 0     |
| forecasts | 1     | 0     | 0     | 0     |
| rise      | 0     | 1     | 0     | 1     |
| in        | 0     | 1     | 1     | 0     |
| july      | 0     | 1     | 1     | 1     |
| increase  | 0     | 0     | 1     | 0     |

#### b. Inverted Index Representation

倒排索引如下表所示：

| Term      | Document List              |
| --------- | -------------------------- |
| new       | Doc 1, Doc 4               |
| home      | Doc 1, Doc 2, Doc 3, Doc 4 |
| sales     | Doc 1, Doc 2, Doc 3, Doc 4 |
| top       | Doc 1                      |
| forecasts | Doc 1                      |
| rise      | Doc 2, Doc 4               |
| in        | Doc 2, Doc 3               |
| july      | Doc 2, Doc 3, Doc 4        |
| increase  | Doc 3                      |

#### c. Returned Results for Queries

##### 1) Query "july AND rise"

For this query, we want documents that contain both "july" and "rise".

| Term | Document List       |
| ---- | ------------------- |
| july | Doc 2, Doc 3, Doc 4 |
| rise | Doc 2, Doc 4        |

0111 AND 0101 = 0101

The query result is Doc 2 and Doc 4.

##### 2)  Query "(NOT increase) AND (home OR sale)"

我们需要查询不包含increase且包含home，sale中至少一个的文档

| Term     | Document List              |
| -------- | -------------------------- |
| increase | Doc 3                      |
| home     | Doc 1, Doc 2, Doc 3, Doc 4 |
| sales    | Doc 1, Doc 2, Doc 3, Doc 4 |

(NOT 0010) AND (1111 or 1111) = 1101

The query result is Doc 1, Doc 2, Doc 4.

### Problem3

#### For Doc1:

| Term      | TF   | IDF  | TF-IDF            |
| --------- | ---- | ---- | ----------------- |
| car       | 27   | 1.65 | 27 * 1.65 = 44.55 |
| auto      | 3    | 2.08 | 3 * 2.08 = 6.24   |
| insurance | 0    | 1.62 | 0 * 1.62 = 0      |
| best      | 14   | 1.5  | 14 * 1.5 = 21     |

#### For Doc2:

| Term      | TF   | IDF  | TF-IDF            |
| --------- | ---- | ---- | ----------------- |
| car       | 4    | 1.65 | 4 * 1.65 = 6.6    |
| auto      | 33   | 2.08 | 33 * 2.08 = 68.64 |
| insurance | 33   | 1.62 | 33 * 1.62 = 53.46 |
| best      | 0    | 1.5  | 0 * 1.5 = 0       |

#### For Doc3:

| Term      | TF   | IDF  | TF-IDF            |
| --------- | ---- | ---- | ----------------- |
| car       | 24   | 1.65 | 24 * 1.65 = 39.6  |
| auto      | 0    | 2.08 | 0 * 2.08 = 0      |
| insurance | 29   | 1.62 | 29 * 1.62 = 46.98 |
| best      | 17   | 1.5  | 17 * 1.5 = 25.5   |

## Part2

 ### 概述 

本次项目主要通过`init.py`和`query.py`两个python代码实现10000篇论文从`MySQL数据库`的导入和查询。

### 代码实现

通过init.py导入MySQL数据库中的数据后通过query.py进行查找。

#### init.py

在init.py中，主要分为两部分：构建`analyzer`和`mappings`，连接`MySQL数据库`进行数据导入

##### 1. 构建analyzer和mappings

- 构建名为 `gh_analyzer`的自定义类型的分词器，该分词器有以下优点：

  1. 使用`standard`分词器作为基础，按单词划分文本。
  2. `lowercase`过滤器将所有词项转换成小写，以确保搜索不受大小写影响。
  3. `english_possessive_stemmer`和`english_stemmer`提供了英语的所有格和一般形式的词干处理。
  4. `asciifolding`过滤器将所有的Unicode字符转换为它们的ASCII等效形式，使得搜索对于Unicode的不同表示更加健壮。
  5. `synonym_filter`定义了同义词，这对于理解一些英文缩写是十分重要的。
  6. `shingle_filter`创建了shingles（文本碎片），这有助于支持短语搜索和近似匹配，同时保留了相邻词项的一些上下文。

  在构建我自己的分词器后，与查询词段最符合的论文评分与默认分词器相比明显提高，其他论文虽有提高但没有特别明显的提高，整体来说文献之间的评分的差距会变大，说明分词器的选取对评分有较大的正面影响，构建好的分词器可以使论文的特定更突出，进而使搜索结果更匹配。

  我使用的分词器具体实现如下:

  ```python
  "settings": {
          "analysis": {
              "filter": {
                  "english_stop": {
                      "type": "stop",
                      "stopwords": "_english_"
                  },
                  "english_stemmer": {
                      "type": "stemmer",
                      "language": "english"
                  },
                  "english_possessive_stemmer": {
                      "type": "stemmer",
                      "language": "possessive_english"
                  },
                  "synonym_filter": {
                  "type": "synonym",
                      "synonyms": [
                          "nlp, natural language processing",
                          "ai, artificial intelligence",
                          "ml, machine learning"
                      ]
                  },
                  "shingle_filter": {
                      "type": "shingle",
                      "min_shingle_size": 2,
                      "max_shingle_size": 3
                  }
              },
              "analyzer": {
                  "gh_analyzer": {
                      "tokenizer": "standard",
                      "filter": [
                          "lowercase",
                          "english_possessive_stemmer",
                          "english_stop",
                          "english_stemmer",
                          "asciifolding",
                          "synonym_filter",
                          "shingle_filter"
                      ]
                  }
              }
          }
      },
  ```

  

- 构建`mappings`

  根据所需要的查询方式构建mappings：

  - year和paper_id字段需要精确查询，故分别设置为long和keyword数据类型
  - title，authors，keywords和venue字段需要进行模糊查询,设置为text类型。
  - MySQL数据库中authors和keywords的存储方式较为特殊，我选择在数据导入部分进行处理，详细操作见[这里](#jump1)
  
  映射具体实现如下:
  
  ```python
  "mappings": {
      "properties": {
          "paper_id": {
              "type": "keyword"
          },
          "title": {
              "type": "text",
              "analyzer": "gh_analyzer"
          },
          "authors": {
              "type": "text",
              "analyzer": "gh_analyzer"
          },
          "keywords": {
              "type": "text",
              "analyzer": "gh_analyzer"
          },
          "year": {
              "type": "long"
          },
          "venue": {
              "type": "text",
              "analyzer": "gh_analyzer"
          }
      }
  }
  ```
  
  

##### 2. 在Elasticsearch中创建索引并从MySQL中导入数据

1. **检查并创建索引**:

   - 首先检查指定的Elasticsearch索引是否存在。如果存在，则先删除该索引，然后重新创建它，如此可以避免数据的重复导入。
   - 创建索引时应用定义在`mappings_and_settings`变量中的映射和设置。

2. **从MySQL数据库提取数据**:

   - 使用SQL查询从MySQL表中提取论文的信息，包括论文ID、标题、年份、作者、关键词和发表的会议或期刊。

3. **处理数据**:

   - 将从数据库中提取的作者和关键词字段（可能存储为JSON字符串）转换为逗号分隔的字符串格式。
   - 为每一行记录创建一个Elasticsearch文档，该文档包含论文的所有相关信息。

4. **导入数据到Elasticsearch**:

   - 使用Elasticsearch客户端的`index`方法将每个文档插入到Elasticsearch索引中。
   - 在插入文档时，使用论文ID作为文档ID。

5. **异常处理和数据库连接关闭**:

   - 在插入数据时，如果遇到异常，则打印错误信息。

   - 数据处理完成后，关闭MySQL数据库的连接。

     

#### query.py

这段代码主要用于从Elasticsearch索引中查询学术论文的数据。它包括两个核心函数：`querypart` 和 `queryAll`，以及一个简单的命令行用户界面来接收输入并显示查询结果。以下是其功能分析：

##### 1.`querypart` 函数

目的：在指定索引中根据特定字段进行搜索。

- 参数：

  - `index`: Elasticsearch中的索引名。
  - `field`: 要查询的字段名。
  - `query_text`: 查询的关键字或文本。
  - `size`: 返回的最大结果数量，默认为10。

- 查询逻辑：

  - 对于数字类型或关键字类型的字段（如`year`或`paper_id`），使用`term`精确查询。
  - 对于文本类型字段，使用`match`查询，并支持自动模糊匹配。

- **结果处理**：提取并返回包括文档ID、得分、标题、年份、作者、关键词、发表地点等信息的结果列表。

##### 2. `queryAll` 函数

- 目的：在所有字段上执行全文搜索。

- 参数：

  - `query_text`: 查询的关键字或文本。
  - `size`: 返回的最大结果数量，默认为10。
  
- 查询逻辑：

  - 使用`multi_match`查询，在所有字段上执行查询，并支持自动模糊匹配。

- **结果处理**：类似于`querypart`，提取并返回详细的结果列表。

##### 3.命令行界面

- 用户可以选择查询特定字段或所有字段。
- 用户输入查询关键字和期望的结果数量。
- 根据用户输入的字段编号，调用相应的查询函数。
- 显示查询结果或“no result”提示。

### 结果展示

这部分展示了导入数据后Elastic search中数据的样式以及指定搜索和全文搜索的结果，并对返回结果评分与排序进行了评价。

####　１. 导入数据样式

在`Kibana`中运行下面代码进行查询

```kibana
GET /icees/_search
{
  "query": {
    "match_all": {}
  }
}
```

得到返回结果

```
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "icees",
        "_type" : "_doc",
        "_id" : "425",
        "_score" : 1.0,
        "_source" : {
          "paper_id" : 425,
          "title" : "Research Progress in Catalysts for Direct Oxidation of Methanol to Dimethoxymethane",
          "year" : 2013,
          "authors" : "Ning Chun-li",
          "keywords" : "Diesel fuel, Inorganic chemistry, Methanol, Organic chemistry, Dimethoxymethane, Chemistry, Catalysis",
          "venue" : "Chemistry World"
        }
      },
      
      --------篇幅原因只保留一个文档-------
      
    ]
  }
}
```

根据返回数据可以发现在`icees`这个索引中存储了10000篇论文，对于每一篇论文都存储了它的"paper_id", "title", "year", "authors", "keywords" , "venue"几个字段。

#### ２. 特定搜索

由于数据库中的数据主要有三个类型：year和paper_id字段分别为long和keyword数据类型，而title和venue字段为字符串，而authors，keywords两个字段存储类型为列表，需要处理后加入elastic search中，故对三个类型分别进行查询进行比较。除此以外，还进行了空字符串的查询查看返回结果。

##### 1. 查询paper_id

   运行代码后，在paper_id字段中查询paper_id为2的论文。

   - 输入

     > Type which field you want to query, 0 for all, 1 for paper_id, 2 for title, 3 for year, 4 for authors, 5 for keywords, 6 for venue
     >
     > > 1

     > Type the word or sentence you want to query
     >
     > > 2

     > Type the size of the result you want to show (default 10)
     >
     > > 10

   - 输出

     ```
     paper_id: 2 
      Score: 8.8049755
     ```
     

   可以看到由于paper_id的查询为特定查询，故查询结果只返回第二篇论文，评分为8.8049755。

##### 2. 查询标题

   运行代码后，在title字段中查询句段为“Transport of Glucose Polymer-Derived Glucose by Rabbit Jejunum”的论文（该句段paper_id为2的论文的标题）。

   - 输入

     > Type which field you want to query, 0 for all, 1 for paper_id, 2 for title, 3 for year, 4 for authors, 5 for keywords, 6 for venue
     >
     > >  2

     > Type the word or sentence you want to query
     >
     > > Transport of Glucose Polymer-Derived Glucose by Rabbit Jejunum

     > Type the size of the result you want to show (default 10)
     >
     > > 3

   - 输出

     ```
     paper_id: 2 
      Score: 99.174545
      title: Transport of Glucose Polymer-Derived Glucose by Rabbit Jejunum
     
     paper_id: 9681 
      Score: 26.666365
      title: Hydrogen Peroxide Mediates Brazilin-induced Glucose Transport in Adipocytes
     
     paper_id: 6064 
      Score: 26.051693
      title: The ultrastructural route of fluid transport in rabbit gall bladder.
     ```
     
     可以看到，我们的输出了三篇与查询词最匹配的文章并给出了每篇文章的得分。由于我们查询的字段为第二篇文章的标题，故第二篇文章有最高的得分99.174545分，其他文章也根据各个字段中匹配的程度给出了合理的得分，并按照得分进行了排序。

##### <span id="jump2">3. 查询作者</span>


运行代码后，在authors字段中查询"Leo A. Heitlinger, Howard R. Sloan, Donna R. DeVore, Ping-Cheung Lee, Emanuel Lebenthal, Michael E. Duffey"(paper_id为2的论文的作者)    
- 输入
  
> Type which field you want to query, 0 for all, 1 for paper_id, 2 for title, 3 for year, 4 for authors, 5 for keywords, 6 for venue
>
> > 4

> Type the word or sentence you want to query
>
> > Leo A. Heitlinger, Howard R. Sloan, Donna R. DeVore, Ping-Cheung Lee, Emanuel Lebenthal, Michael E. Duffey

> Type the size of the result you want to show (default 10)
>
> > 3

- 输出

```
paper_id: 2 
Score: 178.19482
authors: Leo A. Heitlinger, Howard R. Sloan, Donna R. DeVore, Ping-Cheung Lee, Emanuel Lebenthal, Michael E. Duffey

paper_id: 2985 
Score: 18.041279
authors: Howard R. Turtle, James Flood

paper_id: 8584 
Score: 17.726772
authors: Ashley V. Sloan, Joseph R. Martin, Shuo Li, Jiliang Li
```

可以看到，我们的输出了三篇与查询词最匹配的文章并给出了每篇文章的得分。由于我们查询的字段为paper_id为2的论文的作者，故其有最高的得分178.19482分，其他文章也根据各个字段中匹配的程度给出了合理的得分，并按照得分进行了排序。

##### 4. 查询会议

运行代码后，在venue字段中查询"Proceedings of the National Academy of Sciences of the United States of America"   
- 输入

> Type which field you want to query, 0 for all, 1 for paper_id, 2 for title, 3 for year, 4 for authors, 5 for keywords, 6 for venue
>
> > 6     

> Type the word or sentence you want to query
>
> > Proceedings of the National Academy of Sciences of the United States of America

> Type the size of the result you want to show (default 10)
>
> > 3
> >

- 输出

```
- paper_id: 3156 
Score: 86.885605
venue: Proceedings of the National Academy of Sciences of the United States of America

paper_id: 3286 
Score: 86.885605
venue: Proceedings of the National Academy of Sciences of the United States of America

paper_id: 3332 
Score: 86.885605
venue: Proceedings of the National Academy of Sciences of the United States of America
```

由于数据库中有多篇该会议的文章，故各篇文章评分相同。

##### 5. 查询关键词

运行代码后，在keywords字段中查询"Constant supervision, Insulin, Hypoglycemia, Coma, Hypoglycemic coma, Complication, Internal medicine, Diabetes mellitus, Medicine, Endocrinology"(paper_id为1的论文的关键词)

- 输入

> Type which field you want to query, 0 for all, 1 for paper_id, 2 for title, 3 for year, 4 for authors, 5 for keywords, 6 for venue
>
> > 5

> Type the word or sentence you want to query
>
> > Constant supervision, Insulin, Hypoglycemia, Coma, Hypoglycemic coma, Complication, Internal medicine, Diabetes mellitus, Medicine, Endocrinology

> Type the size of the result you want to show (default 10)
>
> > 3

- 输出

```
paper_id: 1 
Score: 128.78996
keywords: Constant supervision, Insulin, Hypoglycemia, Coma, Hypoglycemic coma, Complication, Internal medicine, Diabetes mellitus, Medicine, Endocrinology

paper_id: 6010 
Score: 44.81155
keywords: Insulin, Internal medicine, Medicine, Diabetes mellitus, Endocrinology, Hypoglycemia, Type 2 diabetes, Insulin glargine, Insulin detemir, Type 2 Diabetes Mellitus, NPH insulin

paper_id: 5197 
Score: 44.011177
keywords: Basal insulin, Hypoglycemia, Insulin, Heart rate, Type 1 diabetes, Internal medicine, Endocrinology, Diabetes mellitus, Aerobic exercise, Medicine
```

可以看到，我们的输出了三篇与查询词最匹配的文章并给出了每篇文章的得分。由于我们查询的字段为paper_id为1的论文的关键词，故其有最高的得分128.78996分，其他文章也根据各个字段中匹配的程度给出了合理的得分，并按照得分进行了排序。

##### 6. 查询空字符串

   - 输入

     > Type which field you want to query, 0 for all, 1 for paper_id, 2 for title, 3 for year, 4 for authors, 5 for keywords, 6 for venue
     >
     > > 0

     > Type the word or sentence you want to query
     >
     > >

     > Type the size of the result you want to show (default 10)
     >
     > >

   - 输出

     ```查询空字符串
     no result
     ```

     显示没有查询到结果。

#### 3. 全文搜索

运行代码后，在全文中查询关键词为“Transport of Glucose Polymer-Derived Glucose by Rabbit Jejunum”的论文（该句段paper_id为2的论文的标题）。

- 输入


> Type which field you want to query, 0 for all, 1 for paper_id, 2 for title, 3 for year, 4 for authors, 5 for keywords, 6 for venue
>
> > 0

> Type the word you want to query
>
> > Transport of Glucose Polymer-Derived Glucose by Rabbit Jejunum

> Type the size of the result you want to show (default 10)
>
> > 10

- 输出

```
paper_id: 2
 Score: 99.174545
 Title: Transport of Glucose Polymer-Derived Glucose by Rabbit Jejunum
 Year: 1992
 Authors: Leo A. Heitlinger, Howard R. Sloan, Donna R. DeVore, Ping-Cheung Lee, Emanuel Lebenthal, Michael E. Duffey
 keywords: Absorption (pharmacology), Glucose transporter, Digestion, Jejunum, Phlorizin, Internal medicine, Lagomorpha, Biology, Acarbose, Perfusion, Endocrinology
 Venue: Gastroenterology

paper_id: 8034
 Score: 28.145456
 Title: Mechanism of enhanced insulin sensitivity in athletes. Increased blood flow, muscle glucose transport protein (GLUT-4) concentration, and glycogen synthase activity.
 Year: 1993
 Authors: P. Ebeling, R. E. Bourey, L. Koranyi, J A Tuominen, Leif Groop, J. Henriksson, Mike Mueckler, A. Sovijärvi, Veikko A. Koivisto
 keywords: Insulin, Glucose transporter, Carbohydrate, Glycogen synthase, Glycogen, Hemodynamics, Glucose uptake, Diabetes mellitus, Biochemistry, Endocrinology, Medicine, Internal medicine
 Venue: Journal of Clinical Investigation

paper_id: 9681
 Score: 26.666365
 Title: Hydrogen Peroxide Mediates Brazilin-induced Glucose Transport in Adipocytes
 Year: 2004
 Authors: Lee-Yong Khil, Chang-Kiu Moon
 keywords: Catalase, Glucose transporter, Biochemistry, Biology, Structural change, Hydrogen peroxide, Protein phosphatase 1, Phosphatase, Carbohydrate metabolism, Brazilin
 Venue: Biomolecules & Therapeutics
```

可以看到，我们的输出了三篇与查询词最匹配的文章并给出了每篇文章的得分，由于我们查询的字段为第二篇文章的标题，故第二篇文章有最高的得分99.174545分，其他文章也根据各个字段中匹配的程度给出了合理的得分，并按照得分进行了排序。

此外，程序还输出了各篇文章的详细信息。

#### 4. 模糊查询

运行代码后，在title字段中查询句段为"Transpo of Glucoe Polymr-Derived Glucose by Rabbt Jejunu"的论文（该句段paper_id为2的论文的标题的错误拼写）。

- 输入

  > Type which field you want to query, 0 for all, 1 for paper_id, 2 for title, 3 for year, 4 for authors, 5 for keywords, 6 for venue
  > 
  > > 2

  > Type the word or sentence you want to query
  >
  > > Transpo of Glucoe Polymr-Derived Glucose by Rabbt Jejunu

  > Type the size of the result you want to show (default 10)
  >
  > > 3
  
- 输出

  ```
  paper_id: 2 
   Score: 30.514435
   title: Transport of Glucose Polymer-Derived Glucose by Rabbit Jejunum
  
  paper_id: 5510 
   Score: 10.24628
   title: Acetate, propionate, butyrate, glucose, and sucrose as carbon sources for denitrifying bacteria in soil
  
  paper_id: 4282 
   Score: 10.138487
   title: The clinical application value of glycosylated hemoglobin (HbA_1c) plus fasting plasma glucose in diagnosing diabetes
  ```
  
  可以看到，我们的输出了三篇与查询词最匹配的文章并给出了每篇文章的得分，由于我们查询的字段为第二篇文章的标题，故第二篇文章有最高的得分30.514435分，但与[精确查找](#jump2)的得分相比有较大的降低，其他文章也根据各个字段中匹配的程度给出了合理的得分，并按照得分进行了排序。

####   5. 对Elasticsearch中评分机制的理解和对返回结果的评分与排序的评价

在Elasticsearch中，评分（Score）是一个核心概念，用于量化搜索结果的相关性。Elasticsearch使用复杂的评分算法，基于多种因素计算每个文档的评分。

下面我会对Elasticsearch评分机制的分析，并对提供的搜索结果评分的合理性的解释：

##### Elasticsearch评分机制

1. **基于TF-IDF**: 传统上，Elasticsearch使用了“词频-逆文档频率”（TF-IDF）模型来计算评分。这意味着，一个词语在文档中出现的频率越高（TF），同时在所有文档中出现的频率越低（IDF），该文档就越相关。
2. **字段相关性**: 不同的字段（如标题、摘要、全文）可以被赋予不同的权重，以反映它们在确定文档相关性方面的重要性。
3. **其他因素**: Elasticsearch还可以考虑其他因素，如文档的新鲜度（更靠近当前日期的文档可能更相关）、用户定义的规则（例如，特定领域的文档可能更重要），以及查询时使用的特定搜索操作符和函数。

##### 分析对搜索结果的评分（针对全文搜索）

通过在kibana运行代码后可查询评分的具体解释

```
GET /icees/_search?explain=true
{
  "query": {
      "multi_match": {
          "query": "Transport of Glucose Polymer-Derived Glucose by Rabbit Jejunum",
          "fields": ["*"],
          "fuzziness": "AUTO"
        
      }
  },
  "size" : 1
}
```

通过查看评分的解释，我们可以发现下面几个评分的机制：

1. **关键词匹配**: 在输出中，第一个文档的标题直接匹配了搜索查询（"Transport of Glucose Polymer-Derived Glucose by Rabbit Jejunum"），导致这个文档的评分（99.174545）远高于其他文档。

2. **相关性但非直接匹配**: 第二个和第三个文档虽然与查询相关，但没有直接匹配查询中的所有关键词。这可能是因为它们包含了部分关键词（如“Glucose transporter”），但并未完全匹配整个查询短语，因此它们的评分（21.130503和20.405148）较低。

2. <span id=jump3>**分词器对评分的影响**</span>

   在构建我自己的分词器后，与查询词段最符合的论文评分与默认分词器相比明显提高，其他论文虽有提高但没有特别明显的提高，整体来说文献之间的评分的差距会变大，说明分词器的选取对评分有较大的正面影响，构建好的分词器可以使论文的特定更突出，进而使搜索结果更匹配。

**总体而言，我认为这次搜索实现了预期的功能，对结果进行了合理的评分和排序。**

### 遇到的问题与解决方式

#### 远程连接Elasticsearch

由于我在我们小组的服务器上进行代码的编写，而Elasticsearch服务启动在我自己的电脑上，所以我需要首先修改es的.yml文件开放端口，构建Elasticsearch节点使我自己电脑上的Elasticsearch可以在服务器上访问。

#### <span id="jump1">**mappings的构建**</span>

在构建mapping的过程中，由于MySQL数据库中authors和keywords的存储类似列表，我刚开始将authors和keywords存储为嵌套字段，但这样会使全文查找的部分较为复杂，所以在查阅资料后，我将二者存储为text类型，并在数据导入部分进行处理（见下）

#### 数据导入

从MySQL数据库中读取后，authors和keywords的存储为类似列表的 JSON 字符串，需要通过join的方式使其转变为合理的字符串，下面是我的处理代码：

```python
for row in cursor.fetchall():
    # 将作者和关键词的 JSON 字符串转换为逗号分隔的字符串
    authors_str = ', '.join(json.loads(row[3])) if row[3] else 'N/A'
    keywords_str = ', '.join(json.loads(row[4])) if row[4] else 'N/A'

    # 创建文档
    doc = {
        "paper_id": row[0],
        "title": row[1],
        "year": row[2],
        "authors": authors_str,
        "keywords": keywords_str,
        "venue": row[5]
    }
```

另外，在刚开始我更改mappings后导入数据前没有删除索引导致一个索引有同一篇论文多次出现，因此将代码更改为每次导入前删除索引再重新创建。

### 结论

本项目成功展示了如何使用Elasticsearch结合Python进行大规模学术论文数据的导入、索引和搜索。

通过`init.py`脚本，我们在Elasticsearch中创建了一个定制的分析器和映射，然后从MySQL数据库中导入了10000篇论文数据。这

随后，使用`query.py`脚本，我们对导入的数据进行了多样化的查询测试，包括按paper_id、title、authors等字段的特定搜索，以及全字段的全文搜索。这些示例充分利用了Elasticsearch在处理复杂和多样化的查询时强大的搜索能力和灵活性。

搜索结果的评分分析进一步证明了Elasticsearch评分机制的有效性，特别是在处理直接关键词匹配和相关性评估方面。

总体而言，通过这次项目我熟练掌握了包括配置映射和分析器，数据导入，全局搜索和特定搜索等Elasticsearch的基本用法，第一次搭建起自己的一个搜索引擎。我认为自己有很大收获。
