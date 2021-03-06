Solr是Apache Lucene项目的开源企业搜索平台。其主要功能包括全文检索、命中标示、分面搜索、动态聚类、数据库集成，以及富文本（如Word、PDF）的处理。Solr是高度可扩展的，并提供了分布式搜索和索引复制。Solr是最流行的企业级搜索引擎，Solr 4还增加了NoSQL支持。

[官网地址](http://lucene.apache.org/solr/)
[文档下载](http://apache.mirror.vexxhost.com/lucene/solr/ref-guide/apache-solr-ref-guide-6.3.pdf)

通过该篇笔记，可以使用到solr的以下几个功能点：

* 从Mysql中获取到数据并创建索引
* 进行简单的多表查询、以及从不同表的不同字段进行全文检索（该篇笔记以一个常见的场景进行了演示，即：帖子标题表以及帖子回复表，根据关键字进行标题以及帖子回复的内容进行全文检索）
* 检索结果的分页、以及高亮显示
* 集成中文分词插件ik-analyzer以及从各大词库制作自定义词库（演示了从搜狗输入法词库导入）
* 增量索引，包括数据的增加、修改、删除
* 自动进行增量索引


### 开始使用

准备环境：JDK1.8(最低版本)

本文演示solr版本：6.3.0

建议尽量在linux或者类unix环境中运行使用solr。这里，我搭建了一个ubuntu 16的虚拟机用来演示运行。

![](http://p1.bqimg.com/567571/7129966a40aeefa1.jpg)

进入[下载页面](http://www.apache.org/dyn/closer.lua/lucene/solr/6.3.0)下载后解压到桌面（建议翻墙下载，不翻墙容易中途断掉）

![](http://p1.bqimg.com/567571/be07530f727cfba9.jpg)

##### 启动

1. linux下进入目录后执行：`bin/solr start`
2. windows下进入目录后执行:`bin/solr.cmd start`

linux下面启动成功如图：
![](http://p1.bqimg.com/567571/d649e654b34f9f18.jpg)
进入浏览器访问如下地址：http://localhost:8983/solr 看到如下界面说明solr启动成功
![](http://p1.bqimg.com/567571/45ea18ca85c2c664.jpg)

可以通过执行命令`bin/solr -help`查看帮助。

**下面是一些常用的启动参数：**

* 默认情况下，solr是在后台启动的，如果你不用后台启动的方式可以在启动的时候加一个`-f`参数，`bin/solr start -f`
* solr默认启动的端口是8983，你可以在启动的时候使用`-p`参数指定端口启动，`bin/solr start -p 8090`使用8090端口启动
* solr在一开始就已经提供了一些用于学习的例子，也可以在启动的时候直接启动这些实例用于学习。例如：`bin/solr start -e techproducts`,提供了四个例子：
	* cloud ： This example starts a 1-4 node SolrCloud cluster on a single machine. When chosen, an interactive session will start to guide you through options to select the initial configset to use, the number of nodes for your example cluster, the ports to use, and name of the collection to be created. When using this example, you can choose from any of the available configsets found in $SOLR_HOME/server/solr/configsets.
	* techproducts： This example starts Solr in standalone mode with a schema designed for the sample documents included in the $SOLR_HOME/example/exampledocs directory. The configset used can be found in $SOLR_HOME/server/solr/configsets/sample_techproducts_configs.
	* dih： This example starts Solr in standalone mode with the DataImportHandler (DIH) enabled and several example dataconfig.xml files pre-configured for different types of data supported with DIH (such as, database contents, email, RSS feeds, etc.). The configset used is customized for DIH, and is found in $SOLR_HOME/example/example-DIH/solr/conf. For more information about DIH, see the section Uploading Structured Data Store Data with the Data Import Handler.
	* schemaless： This example starts Solr in standalone mode using a managed schema, as described in the section Schema Factory Definition in SolrConfig, and provides a very minimal pre-defined schema. Solr will run in Schemaless Mode with this configuration, where Solr will create fields in the schema on the fly and will guess field types used in incoming documents. The configset used can be found in $SOLR_HOME/server/solr/configsets/data_driven_schema_configs.

### 创建一个Core

如果我们在启动solr的时候没有启动任何示例配置的情况下，我们需要创建一个core并创建索引。

如果使用上面的命令`bin/solr start`启动了solr并且没有关闭的情况下，我们可以用命令`bin/solr create -c start`创建一个core(创建core需要保证solr并没有停止) 

在管理页面刷新可以看见多了一个start

![](http://p1.bpimg.com/567571/f7ac5059e984549f.jpg)

### 增加一个文件夹的索引

创建文件的索引需要用到post这个工具，这里我们以创建一个文件夹(当前目录下的docs文件夹)的索引为例：

执行命令`bin/post -c start docs/`稍等一下等命令执行完毕，`start`是我们上面刚刚创建的core，docs是我们当前目录的文件夹。如此创建完毕后就可以开始使用搜索功能了。

### 体验简单的搜索

 这里为了方便我就不在虚拟机里面进行搜索了，记住了虚拟机的IP地址就可以在主机访问。

![](http://p1.bpimg.com/567571/24ab541a2373c135.jpg)

点击进入后选择刚刚创建的`start`

![](http://p1.bpimg.com/567571/0af529009d1bc2c5.jpg)

选择Query，并填入index，点击执行查询得到如下结果：

![](http://p1.bpimg.com/567571/0cfd0c83a81069b3.jpg)

可以查看右边的数据，QTime是耗时（应该是毫秒单位）。response下面的numFound意思就是一共发现了413条记录。。。

从速度可以看出，检索查询的速度快快哒。


### 从mysql导入数据到solr

https://wiki.apache.org/solr/DataImportHandler#A_shorter_data-config

**首先准备Mysql的环境：**

这里我创建了一个名为solr的数据库，字符集为utf-8。里面创建了两张表，创建的代码分别如下：

question表
```
DROP TABLE IF EXISTS `question`;
CREATE TABLE `question` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL,
  `create_time` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

replay表（现在创建备用）
```
DROP TABLE IF EXISTS `replay`;
CREATE TABLE `replay` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `q_id` int(11) NOT NULL,
  `content` varchar(255) NOT NULL,
  `create_time` datetime NOT NULL,
  PRIMARY KEY (`id`,`q_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

至于数据，就随机导入了。我不提供初始数据了。

**接下来是solr的部分：**

首先下载[mysql-connector-java](http://dev.mysql.com/downloads/connector/j/)

解压其中的mysql-connector-java-xx.xx.jar到`server/solr-webapp/webapp/WEB-INF/lib`,如下图：

![](http://p1.bqimg.com/567571/0ab07862e7dbc5a1.jpg)

还需要导入solr-dataimporthandler-6.3.0.jar,到`server/solr-webapp/webapp/WEB-INF/lib`。这个jar在dist目录可以找到。

创建一个core名字为for_mysql.命令是`bin/solr create -c for_mysql`。`server/solr/`目录下面就是存放的core的配置以及索引等信息。

例如刚刚创建的`for_mysql`的目录就是`server/solr/for_mysql`。

进入`server/solr/for_mysql/conf`目录，打开`managed-schema`这个文件，加入以下代码：

```
<field name="id" type="string" multiValued="false" indexed="true" required="true" stored="true"/>
<field name="title" type="string" indexed="true" stored="false" required="true" multiValued="false"/>
<field name="create_time" type="date" indexed="true" stored="true" required="true" multiValued="false"/>
```
一些情况下：在我们添加完成上面的代码重启solr时会出现一下错误：

![](http://p1.bpimg.com/567571/2f8c2223e0ee2624.jpg)

那就是managed-schema里面已经有了id这个字段的配置，我们把上面的关于id那一行删除即可。。


**todo：解释各个字段**


在`server/solr/for_mysql/conf`目录下面创建一个xml文件名字为：`data-config.xml`填入以下内容：

```
<?xml version="1.0" encoding="UTF-8" ?>  
<dataConfig>   
<dataSource name="fromMysql"
          type="JdbcDataSource"   
          driver="com.mysql.jdbc.Driver"   
          url="jdbc:mysql://192.168.1.179:3306/solr"   
          user="root"   
          password="password"/>   
<document>   
    <entity dateSource="fromMysql" pk="id" name="question" query="SELECT * FROM question" >
         <field column="id" name="id"/> 
         <field column="title" name="title"/> 
         <field column="create_time" name="create_time"/>
    </entity>   
</document>   

</dataConfig>
```

上面的代码和我们配置jdbc数据源类似，dataSource的name是随意取的，但是下面的节点需要和dataSource保持一致。下面的entity节点和hibernate差不多，column和数据库的字段保持一致，name和前面`managed-schema`文件配置的保持一致。

然后在同目录下的`solrconfig.xml`添加一下节点：

```
<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
    <lst name="defaults">
      <str name="config">data-config.xml</str>
     </lst>
  </requestHandler>
```

节点添加在<requestHandler name="/select"这个节点前面，防止添加节点出错。这个节点的添加是为了引用我们前面创建的data-config.xml的配置到配置文件中。

![](http://i1.piimg.com/567571/b3901307556a8b4b.jpg)

保存后重启solr.

打开浏览器可以看见多出了一个配置项：

![](http://i1.piimg.com/567571/ae863d53d44e6050.jpg)

然后如下图所示：

![](http://p1.bpimg.com/567571/86cab04870f78007.jpg)

按照图中选择完毕后点击execute按钮。Clean选项表示删除以前的索引，full-import表示引入完整的数据，如果执行成功会如下图所示：

![](http://p1.bpimg.com/567571/8c43de10daba17d1.jpg)

表示导入数据到solr成功。

至此，从mysql导入数据到solr完成，接下来就是如何进行简单的全文检索了~~~

### 同一张表中不同字段的检索

进入`server/solr/for_mysql/conf`目录，打开`managed-schema`这个文件，加入以下代码：

```
<field name="text" type="text_general" indexed="true" stored="false" multiValued="true" />
<copyField source="title" dest="text"/>
<copyField source="id" dest="text"/>
```

大致解释一下代码的意思，第一行的意思就是建立一个新的字段名为text,下面的copyField就是把前面配置的title和id的数据复制到text这个节点中去。配置完毕后重启solr。重启以后记得重新导入一下mysql的数据，打开query界面，点击execute query按钮如下图：
![](http://p1.bqimg.com/567571/92f8c215043b1b4e.jpg)

如图会获取到全部的数据。上面的q表示搜索的内容，`*:*`表示从所有字段中检索所有内容。

接下来，我们要实现同一张表中的不同字段的检索只需要输入：`text:xxx` xxx是你想搜索的内容。

比如我们这里输入： 人 就可以看见能搜索出title带有 人 的内容。
![](http://p1.bqimg.com/567571/259a4aa71a6a98e3.jpg)


再输入：3

![](http://p1.bqimg.com/567571/15155c19e763e15d.jpg)

可以看到能搜索出id为3的内容。

下面还有一种更简便的方法，我们在`server/solr/for_mysql/conf/managed-schema`这个文件添加如下节点：

```
<defaultSearchField>text</defaultSearchField>
```

意思是让text这个字段作为我们默认的搜索字段。

重新换一种查询方式：

![](http://p1.bqimg.com/567571/853a4bf7e6e16f00.jpg)

在df填入上面配置的text后直接搜索3就能得到和上面一样的结果。。。是不是更加方便了呢~~

### 多表查询

先看一个非常常用的需求，如同我们上面建立的两张表。如果我们做搜索功能，肯定是从标题和回复的内容中做检索。这久涉及到了从不同的两张表查询的需求。

因为solr并不支持一对多或者多对一的查询，所以我们需要导入数据到solr的过程中就做成一张表的形式（solr也可以进行子查询的方式，但是对于我们的需求来说，子查询的方式并不适合）。

由于是两张表的查询，为了避免混淆我们重新编辑`managed-schema`如下：

```
<field name="id" type="string" multiValued="false" indexed="true" required="true" stored="true"/>
<field name="title" type="text_general" multiValued="false" indexed="true" required="true" stored="true" />
<field name="create_time" type="date" indexed="true" stored="true" required="true" multiValued="false"/>


<field name="r_id" type="string" multiValued="false" indexed="true" required="false" stored="true"/>
<field name="q_id" type="string" indexed="true" stored="true" required="false" multiValued="false"/>
<field name="content" type="text_general" multiValued="true" indexed="true" required="false" stored="true" />
<field name="r_create_time" type="date" indexed="true" stored="true" required="false" multiValued="false"/>

<field name="text" type="text_general" indexed="true" stored="false" multiValued="true" />
<copyField source="title" dest="text"/>
<copyField source="id" dest="text"/>
<copyField source="content" dest="text"/>
<defaultSearchField>text</defaultSearchField>
```

`data-config.xml`文件变更如下：

```
<?xml version="1.0" encoding="UTF-8" ?>  
<dataConfig>   
<dataSource name="fromMysql"
          type="JdbcDataSource"   
          driver="com.mysql.jdbc.Driver"   
          url="jdbc:mysql://192.168.1.179:3306/solr"   
          user="root"   
          password="bpaTH330"/>   
<document>   
    <entity dateSource="fromMysql" pk="id"  name="question" query="SELECT r.id,q.id as q_id,q.title,r.id AS r_id,r.content,r.create_time  FROM replay AS r,question as q WHERE r.q_id = q.id" >
         <field column="id" name="id"/>
	 	 <field column="title" name="title"/>
         <field column="q_id" name="q_id"/> 
         <field column="content" name="content"/>
	     <field column="create_time" name="create_time"/>
    </entity>   
</document>   

</dataConfig>

```

上述更改了`query`的语句。需要注意的是此时的id已经不是`question`表的id了，而是`replay`表的id。如果继续使用`question`表的id作为主键的话，查询时solr会过滤掉id重复的实体，返回的实体会不全。

上述更改完成后重启solr。并且重新导入数据，导入数据后查询大概会显示如下的结果：

![](http://i1.piimg.com/567571/632f26840f6f9604.jpg)

验证一下是否能正常的查询：分别以content和title里面的个别字进行搜索

![](http://p1.bqimg.com/567571/7c3fdd8d2bea62e2.jpg)

![](http://p1.bqimg.com/567571/3880b3ed456979d7.jpg)

应该是能分别检索出我们想要的结果。但是会有一个问题，下节会说到。


### 简单的去除重复和高亮检索中的文本

**去重**
从上面的查询不难看出有一个非常严重的问题，当我们在以标题搜索的时候。会导致把标题和搜索内容高度重合的所有回答都检索出来，很明显。这样不是我们想要的结果，应该尽量避免这种情况出现。我试着用知乎的搜索功能找到一个标题搜索了一下，不会重复出现。但是会出现一个尽量和搜索内容高度吻合的答案，这里我们使用solr的分组（group）的功能也能达到需要的效果：

在查询的时候添加这两个参数即可`group=true&group.field=q_id`，第一个是开启`group`功能，第二个是用来分组的字段，我们为了搜索标题的时候不会出现多篇标题一样的所以填的是`q_id`，`q_id`代表的是`question`表的id。

![](http://p1.bqimg.com/567571/8131b88f14a70341.jpg)

输入标题检索：

![](http://p1.bpimg.com/567571/3be2ab6b7fc7be42.jpg)

可以看见不会出现之前的全是同一篇文章的情况了。

**搜索到的内容高亮：**

搜索时加入`hl`以及`hl.fl`即可，`hl.fl`是搜索匹配时高亮的字段，多个字段用英文逗号分隔即可。如果所有内容都想要高亮，直接填`*`就好。
![](http://p1.bpimg.com/567571/adae632369251e47.jpg)
高亮的内容在json数据的底部：
![](http://p1.bqimg.com/567571/2917ecb40f7255e8.jpg)

**去重但是保留**

这个小标题实在不知道该怎么写，大概的要求就是，，上面我们已经实现了去重只保留了一条数据。。。但是我们还可以设置一个显示的最大值，允许重复2条或者3条。只需要再加一个参数即可`group.limit=3`比如这样允许重复了最大是三条。如图：

![](http://p1.bqimg.com/567571/1bb01c678633ccaa.jpg)


### 分页

solr中分页和mysql中类似，主要有两个参数。一个`start`表示开始的位置（从0开始），`rows`表示每页显示的条数（如果你填0的话会不查不到数据）。

![](http://i1.piimg.com/567571/997096baeee2a334.jpg)

如图就是第二页start=(1*2),rows=2的数据。

### 加入中文分词

solr默认对中文的分词支持不是很好，如下图
![](http://p1.bqimg.com/567571/278f5b067a6121be.jpg)
solr对中文是一个字一个字分隔开的。

我选用的分词插件是：ik-analyzer 对高版本的solr支持比较好。

[google-code官网链接](https://code.google.com/archive/p/ik-analyzer/ "google-code官网链接") 

[github仓库链接](https://github.com/wks/ik-analyzer)

但是solr6并不能从上面的编译拿来使用，solr6有对应的jar包，需要对应版本才行。这里给出solr6.3可用的版本
[ik-analyzer-solr6](https://github.com/cj96248/ik-analyzer-solr6)
需要自己下载源代码并进行编译，直接作为maven项目导入以后执行命令`mvn clean install`成功后在target目录找到jar包即可。为了方便我编译了一份上传[百度网盘](http://pan.baidu.com/s/1hs7ckIw)

下载jar并拷贝到`server/solr-webapp/webapp/WEB-INF/lib`(这个目录里面的jar包在solr-webapp启动的时候都会被加载)

然后打开`server/solr/for_mysql/conf/managed-schema`，添加如下代码：

```
<fieldType name="text_ik" class="solr.TextField">   
  <analyzer type="index">
    <tokenizer class="org.wltea.analyzer.lucene.IKTokenizerFactory" useSmart="false" />
  </analyzer>
  <analyzer type="query">
    <tokenizer class="org.wltea.analyzer.lucene.IKTokenizerFactory" useSmart="true" />
  </analyzer>
</fieldType>
```

或者（两者选一）

```
<fieldType name="text_ik" class="solr.TextField">   
  <analyzer type="index" useSmart="false" class="org.wltea.analyzer.lucene.IKAnalyzer"/>   
  <analyzer type="query" useSmart="true" class="org.wltea.analyzer.lucene.IKAnalyzer"/>   
</fieldType>
```

重启solr，进入分词页面进行分词验证。

![](http://p1.bqimg.com/567571/2b6a8cdc25f58c1d.jpg)

可以看见上面的一段已经被分成了几个部分。

**增加自定义词库**

为了提高准确率，我们还应该加入一些流行的词库、地名、院校名称、网络用语等。使得搜索更加准确。

比如一些院校名称，分词的时候仍然有缺点（例如：`贵州财经学院` 是一个高校名称不应该分开）：

![](http://p1.bqimg.com/567571/29da906eca5e64a0.png)

首先是网上的参考文章，当然 本篇的也可以。[加入自定义词库](http://www.coin163.com/java/docs/201309/d_2835029802.html) ，[分词用到的工具](https://github.com/studyzy/imewlconverter)

首先去[搜狗字典](http://pinyin.sogou.com/dict/)下载细胞词库，什么词库都可以，只要工具能转换即可。

下载好词库后用工具打开并转换：

![](http://p1.bqimg.com/567571/a072fec7f22e7472.jpg)

把导出来的txt并重新命名为`ext.dic`。

在`WEB-INF`目录下创建`classes`文件夹。

在刚刚下载的solr6的那个target目录，找到`IKAnalyzer.cfg.xml`以及`stopword.dic`文件。连同上面生成的`ext.dic`文件,一起拷贝到刚刚创建的`classes`文件夹下。如图：

![](http://i1.piimg.com/567571/78829040537f1a77.jpg)

重启solr，再次进入分词的页面，输入 `贵州财经学院` 我们会得到如图的结果：

![](http://i1.piimg.com/567571/ccf5a14c2cb8768e.jpg) 

说明词库导入成功了~~~
 
下面解释一下`IKAnalyzer.cfg.xml`里面的配置，看懂的就略过(⊙o⊙)

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">  
<properties>  
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 
	<entry key="ext_dict">ext.dic;</entry> 
	-->
	<entry key="ext_dict">ext.dic;</entry> 
	<!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords">stopword.dic;</entry> 
	
</properties>
```

只要把文件`IKAnalyzer.cfg.xml`打开看过的应该都一目了然了。。。

**最后，需要把上面检索的字段改成`text_ik`类型**。

![](http://p1.bpimg.com/567571/5ff20fa79a3fb312.jpg)


### 从数据库中更新数据

[自动从数据库中更新数据并建立索引](https://wiki.apache.org/solr/DataImportHandler#Using_delta-import_command)

首先为了能自动从数据库中拉取数据，不得不再次修改一下数据库。在`question`表和`replay`表中分别添加字段`last_update_time`。字段的意思就是对于本条数据最后更新的时间。**每当该条数据更新时需要把这个时间重置为当前时间**

首先更改`bin/solr.in.sh`文件里面的`SOLR_TIMEZONE`的值为`GMT+8`。需要把这个更改为中国的时区，不然等下solr更新的时间会和中国有时间差。

我们进行一次full-import以后打开和`data-config.xml`同一级别目录的`dataimport.properties`文件，进去可以看见有last_index_time等属性，这个就是最后一次建立索引的时间。

solr是如何进行增量建立索引的呢？就是通过建立最后一次索引的时间去和数据库中最后更新时间进行对比，当数据库最后修改时间在最后一次建立索引的时间之后时，就说明数据库中的数据已经更新了。需要我们重新增量把更新后的数据更新到solr的索引中。而最后一次建立索引的时间就记录在`dataimport.properties`文件中。

为了简便我们从数据库中更新数据，需要把`data-config.xml`该成下面的代码（等下解释节点属性的涵义）：

```
<document>   
    <entity dateSource="fromMysql" pk="id"  name="replay" query="SELECT * FROM replay" deltaQuery="SELECT id FROM replay WHERE last_update_time > '${dataimporter.last_index_time}'" deltaImportQuery="SELECT * FROM replay WHERE id='${dataimporter.delta.id}'" >
         <field column="id" name="id"/>
         <field column="content" name="content"/>
	     <field column="create_time" name="create_time"/>

        <entity pk="q_id" name="question" query="SELECT title FROM question WHERE id=${replay.q_id}"  deltaQuery="SELECT id as q_id  FROM question WHERE  last_update_time > '${dataimporter.last_index_time}'" parentDeltaQuery="SELECT id FROM replay WHERE q_id=${question.q_id}"  >
            <field column="title" name="title" />
                <field column="q_id" name="q_id"/> 
        </entity>
    </entity>   
</document>   
```

其它文件不需要改动，一样可以做到检索title字段和content字段。

首先query属性不用说，这是在进行full-import时用到的查询语句。

被包含的`entity`节点是查询出某篇回复所属的标题的内容，需要用`q_id`查询。

重点说一下`deltaQuery`和`deltaImportQuery`以及`parentDeltaQuery`。

> * The query gives the data needed to populate fields of the Solr document in full-import
> * The deltaImportQuery gives the data needed to populate fields when running a delta-import
> * The deltaQuery gives the primary keys of the current entity which have changes since the last index time
> * The parentDeltaQuery uses the changed rows of the current table (fetched with deltaQuery) to give the changed rows in the parent table. This is necessary because whenever a row in the child table changes, we need to re-generate the document which has that field.

先给出文档上面的解释。然后说一下我的理解，如果有偏差的地方还请指出：

* `deltaQuery`是查询出哪些数据需要更新的SQL语句，例如：`SELECT id FROM replay WHERE last_update_time > '${dataimporter.last_index_time}'` 就是把数据库中更新时间在建立索引时间之后的数据的主键查询出来，`'${dataimporter.last_index_time}'`就是`dataimport.properties`文件中的时间。
* `deltaImportQuery`是确定需要把数据查询并重新建立索引，其中的ID数据来自`deltaQuery`的查询。`'${dataimporter.delta.id}'`就是`deltaQuery`产生的id。查询的时候字段应和query查询的字段保持一致。
* `parentDeltaQuery`是用于子表的，例如上述 对于每个回复  这篇帖子的标题 对于每条回复来说就是子表，因为每条回复信息里面都包含了标题的内容。对于这种情况，例如很多时候，回复并没有更新。但是标题却更新了，更新的时候就需要用到这个属性。并且`parentDeltaQuery="SELECT id FROM replay WHERE q_id=${question.q_id}"`注意查询的表并不是`question`表。而且`${question.q_id}`是来自`deltaQuery`查询得来得，和前面根节点的不一样。

到现在为止，已经能正确的从数据库中更新数据了。包括question表更新时也能正确更新到索引中。

但是还有一种情况没有处理，那就是我们只处理了更新的情况。但是还有一种情况，就是我们如果是在数据库中删除了索引，也要能正确的反应到solr的索引中。

为了能把删除反应到solr中，我们需要在mysql中建立一个用于识别‘删除’的字段，这里的删除并不是真的从mysql中删除，而是用一个字段表示这条数据已经是废弃的。这里我们分别在`question`和`replay`表中新建一个int类型的`is_delete`字段。我们规定当这个字段值为1时该条数据默认被删除。

在`name="replay"`的`entity`节点上新加一个属性：`deletedPkQuery="SELECT id FROM replay AS r WHERE r.q_id in ( SELECT id FROM question WHERE is_delete =1) OR r.is_delete = 1"`

完成后代码如下：

```
<document>   
    <entity dateSource="fromMysql" pk="id"  name="replay" query="SELECT * FROM replay" deltaQuery="SELECT id FROM replay WHERE last_update_time > '${dataimporter.last_index_time}'" deltaImportQuery="SELECT * FROM replay WHERE id='${dataimporter.delta.id}'" deletedPkQuery="SELECT id FROM replay AS r WHERE r.q_id in ( SELECT id FROM question WHERE is_delete =1) OR r.is_delete = 1">
         <field column="id" name="id"/>
         <field column="content" name="content"/>
	     <field column="create_time" name="create_time"/>

        <entity pk="q_id" name="question" query="SELECT title FROM question WHERE id=${replay.q_id}"  deltaQuery="SELECT id as q_id  FROM question WHERE  last_update_time > '${dataimporter.last_index_time}'" parentDeltaQuery="SELECT id FROM replay WHERE q_id=${question.q_id}"  >
            <field column="title" name="title" />
                <field column="q_id" name="q_id"/> 
        </entity>
    </entity>   
</document>   
```

达到的效果是：如果我们标识了`replay`表的某条数据为is_delete时，进行增量索引时会把这条数据从索引中删除。当我们把`question`表的某条数据的`is_delete`为1时，进行增量索引时会把所有q_id和标识相等的所有数据从索引中删除。效果类似与mysql中的关联删除。当我们把某条question删除时，与之关联的replay的数据也要删除。

到目前为止，已经能按照我们的需求进行增量索引了。

### 自动进行增量索引

自动进行增量索引，solr给出了一个类似于插件的类库。当我们启动jetty服务的时候启动一个定时服务，原理就是每隔一段时间进行一次增量索引的请求。这样就达到了自动进行增量索引的效果。

原作者github连接[solr-data-import-scheduler](https://github.com/mbonaci/solr-data-import-scheduler)

但是原来的基础上有bug，于是又有作者在此基础上改进并增加了功能。[文章链接](http://www.sxt.cn/u/756/blog/4231)

但是作者的在solr6.3版本中呢，又有一点小问题。所以把源码下载下来改了一下编译放到github，由于没有找到[改进版本](http://www.sxt.cn/u/756/blog/4231)的github地址，所以我直接从原作者那里fork了一份过来。地址为：[链接](https://github.com/callme001/solr-data-import-scheduler)


1. 首先，下载`apache-solr-dataimportscheduler.jar`和solr自带的 `apache-solr-dataimporthandler-6.3.0.jar`, `apache-solr-dataimporthandler-extras-6.3.0.jar`后面两个jar在dist目录可以找到，如果是按照前面的步骤来的话，只用`apache-solr-dataimporthandler-extras-6.3.0.jar`以及`apache-solr-dataimportscheduler.jar`。把这两个jar复制到`server/solr-webapp/webapp/WEB-INF/lib`目录。


2. 并在`web.xml`的servlet节点前加入一个节点，代码如下

```
<listener>
  <listener-class>org.apache.solr.handler.dataimport.scheduler.ApplicationListener</listener-class>
</listener>
```

3. 在`server/solr`下创建一个文件夹`conf`.创建一个名为`dataimport.properties`文件，文件内容为：

```
#################################################
#                                               
#
#      dataimport scheduler properties         
#
#                                              
#
#################################################

 

#to sync or not to sync

# 1 - active; anything else - inactive

syncEnabled=1

 

# which cores to schedule

# in a multi-core environment you can decide which cores you want syncronized

# leave empty or comment it out if using single-core deployment

syncCores=for_mysql

 

# solr server name or IP address

# [defaults to localhost if empty]

server=localhost

 

# solr server port

# [defaults to 80 if empty]

port=8983

 

# application name/context

# [defaults to current ServletContextListener's context (app) name]

webapp=solr

 

# URL params [mandatory]

# remainder of URL

params=/dataimport?command=delta-import&clean=false&commit=true

 

# schedule interval

# number of minutes between two runs

# [defaults to 30 if empty]

interval=1

 

# 重做索引的时间间隔，单位分钟，默认7200，即5天; 

# 为空,为0,或者注释掉:表示永不重做索引

reBuildIndexInterval=7200

 

# 重做索引的参数

reBuildIndexParams=/dataimport?command=full-import&clean=true&commit=true


# 重做索引时间间隔的计时开始时间，第一次真正执行的时间=reBuildIndexBeginTime+reBuildIndexInterval*60*1000；

# 两种格式：2012-04-11 03:10:00 或者  03:10:00，后一种会自动补全日期部分为服务启动时的日期

reBuildIndexBeginTime=03:10:00

```

4. 完毕后，我们重新启动solr。我们尝试修改一条数据并更新数据的最后修改时间。大约过一分钟我们直接查询就能看到更新后的数据了