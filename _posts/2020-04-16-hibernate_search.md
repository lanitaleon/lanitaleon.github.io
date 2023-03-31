---
layout: post
title: Hibernate-Search无法生成子类索引
---


官方文档：

https://docs.jboss.org/hibernate/search/6.0/reference/en-US/html_single/#gettingstarted-dependencies

索引查看工具：

https://code.google.com/archive/p/luke/



aggregable

norms

analyzer

bridge



### 1.是否支持继承结构

#### 1.1无法搜索父类字段

在实际操作中，建立如下结构，查询SubDocument，无法检索Document中的key字段。

```java
class Document {
	private Integer key;
}
class SubDocument extends Document {
	private Integer subInfo;
}
```

hibernate继承结构是通过如下注解实现的。

```java
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "doc_type")
@DiscriminatorValue("ARTICLE")
```

经过关键词检索，官方文档中唯一相关的是`multi_tenancy.strategy: discriminator`

但是它跟`@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`, 

`@DiscriminatorColumn(name = "xxx")`并没有关系，

指的是多个应用程序之间产生的租户概念。

此外，官方文档再无提及，在官方demo中也未发现类似实现。

经过检索发现如下描述称hibernate-search默认不支持继承结构。

https://stackoverflow.com/questions/12026813/hibernate-search-inheritance-models

经过测试发现，父类中的基本类型可正常查询，如下字段不能查询：

```java
@IndexingDependency(reindexOnUpdate = ReindexOnUpdate.NO)
@IndexedEmbedded
@ManyToOne
@JoinColumn(name = "tree_id")
private Tree tree;
```

修改为如下实现可正常查询：

```java
@GenericField(aggregable = Aggregable.NO, valueBridge = @ValueBridgeRef(type = DocumentBridge.class))
@ManyToOne
@JoinColumn(name = "tree_id")
private Tree tree;

public class DocumentBridge implements ValueBridge<Tree, String> {
    @Override
    public String toIndexedValue(Tree tree, ValueBridgeToIndexedValueContext valueBridgeToIndexedValueContext) {
        return tree == null ? null : tree.toString();
    }
}
```

是否`@IndexedEmbedded`对应的查询错误



#### 1.2未建立子类索引文件

通过如下方式生成索引文件

```java
SearchSession searchSession = Search.session(entityManager);
try {
    searchSession.massIndexer().startAndWait();
} catch (InterruptedException e) {
    LoggerUtil.getLogger().error("生成索引异常", e);
    Thread.currentThread().interrupt();
}
```

发现仅在Document索引文件夹下存在内容，在SubDocument中都是空的。

https://stackoverflow.com/questions/19857177/create-separate-hibernate-search-indexes-for-each-subclass

参考这个回答，将Document上的Indexed注解去掉，在SubDocument索引文件夹生成了内容。

### 2.搜索问题

#### 2.1should如何实现or

`where ( conditionA or conditionB) and conditionC`如何用query实现

```java
SearchResult<Article> articlePage = Search.session(entityManager).search(Article.class)
        .where(f -> f.bool(b -> {
            b.must(ff -> ff.bool()
                    .should(f.match().field(TITLE).matching(searchText))
                    .should(f.match().field(CONTENT).matching(searchText).skipAnalysis())
            );
            b.must(ff -> ff.bool(
                    bb -> {
                        for (Tree treeItem : treeList) {
                            bb.should(f.match().field(TREE).matching(treeItem));
                        }
                    }
            ));
            b.must(f.match().field(PUBLISH_FLAG).matching(publishFlag));
            b.must(f.match().field(DELETED).matching(false));
        }))
        .fetch(offset, pageSize);
```

错误实现：

```java
SearchResult<Article> articlePage = Search.session(entityManager).search(Article.class)
        .where(f -> f.bool(b -> {
            b.must(f.matchAll());
            for (Tree treeItem : treeList) {
                b.should(f.match().field("tree").matching(treeItem));
            }
            b.must(f.match().fields("title", "content").matching(searchLike).skipAnalysis());
            b.must(f.match().field("publishFlag").matching(publishFlag));
            b.must(f.match().field("deleted").matching(false));
        }))
        .fetch(offset, pageSize);
```

没有现成的`IN SQL`，使用should替代or需要外层封装must；

`fields`虽然是or的意思，但是对应的analyzer不同。

```java
@FullTextField(analyzer = "enHtmlAnalyzer", norms = Norms.NO)
private String content;
@FullTextField(analyzer = "normalText")
private String title;
```

