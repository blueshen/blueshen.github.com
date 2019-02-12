---
title: ElasticSearch-Nested嵌套类型解密
date: 2019-01-10 10:20:53
tags: [ elasticsearch, lucene, nested ]
categories: [ elasticsearch ]
---

Lucene Field本身并不支持嵌套类型，最多也就支持多值类型。ElasticSearch进行了扩展，使得Field可以是object对象类型或者nested嵌套类型。

object类型，本质上是把字段**路径打平**，最终在索引里还是一个正常的Field字段，object类型转化后，并不会保留**对象内的属性对应关系**，这在查询中可能需要特别注意。然后nested嵌套类型却实现的对应关系。

使用上的区别，可以参考官方[Nested Data Type](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html)。

本文，我们主要关注的是elasticsearch内部是如何实现的？



### ElasticSearch的官方说法

Because nested documents are indexed as separate documents, they can only be accessed within the scope of the `nested` query, the `nested`/`reverse_nested`aggregations, or [nested inner hits](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-inner-hits.html#nested-inner-hits).

For instance, if a string field within a nested document has [`index_options`](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-options.html) set to `offsets` to allow use of the postings during the highlighting, these offsets will not be available during the main highlighting phase. Instead, highlighting needs to be performed via [nested inner hits](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-inner-hits.html#nested-inner-hits).

Indexing a document with 100 nested fields actually indexes 101 documents as each nested document is indexed as a separate document. To safeguard against ill-defined mappings the number of nested fields that can be defined per index has been limited to 50. See [Settings to prevent mappings explosion](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html#mapping-limit-settings)[edit](https://github.com/elastic/elasticsearch/edit/6.5/docs/reference/mapping.asciidoc).

从这里的介绍，我们所知有限。能得到的结论如下：

- nested字段在索引里是作为Document**单独存储**的
- 普通的query对nested字段查询无效，必须使用**nested Query**
- highlight高亮也需要使用专门的
- 对于nested类型的field个数是有限制的。

### Nested类型怎么分开存储的

`org.elasticsearch.index.mapper.DocumentParser`类是专门用于解析要索引的Document的。

其中有这样一个方法来解析对象类型。

```java
static void parseObjectOrNested(ParseContext context, ObjectMapper mapper) throws IOException {
    ...
        
    ObjectMapper.Nested nested = mapper.nested();
    if (nested.isNested()) {
        //如果是nested类型，这里单独处理了
        context = nestedContext(context, mapper);
    }

  ...
}
```

下面来看是怎么处理的。

```java
private static ParseContext nestedContext(ParseContext context, ObjectMapper mapper) {
    context = context.createNestedContext(mapper.fullPath());
    ParseContext.Document nestedDoc = context.doc();
    ParseContext.Document parentDoc = nestedDoc.getParent();

    // We need to add the uid or id to this nested Lucene document too,
    // If we do not do this then when a document gets deleted only the root Lucene document gets deleted and
    // not the nested Lucene documents! Besides the fact that we would have zombie Lucene documents, the ordering of
    // documents inside the Lucene index (document blocks) will be incorrect, as nested documents of different root
    // documents are then aligned with other root documents. This will lead tothe nested query, sorting, aggregations
    // and inner hits to fail or yield incorrect results.
    // 这里把根/父Document的id,同时赋予nested Document，保证删除父Document的时候，nested Document同时被删除
    IndexableField idField = parentDoc.getField(IdFieldMapper.NAME);
    if (idField != null) {
        // We just need to store the id as indexed field, so that IndexWriter#deleteDocuments(term) can then
        // delete it when the root document is deleted too.
        // 这里增加一个_id字段
        nestedDoc.add(new Field(IdFieldMapper.NAME, idField.binaryValue(), IdFieldMapper.Defaults.NESTED_FIELD_TYPE));
    } else {
        throw new IllegalStateException("The root document of a nested document should have an _id field");
    }

    // the type of the nested doc starts with __, so we can identify that its a nested one in filters
    // note, we don't prefix it with the type of the doc since it allows us to execute a nested query
    // across types (for example, with similar nested objects)
    // 这里增加一个_type字段，值为__开头，这里需要注意
    nestedDoc.add(new Field(TypeFieldMapper.NAME, mapper.nestedTypePathAsString(), TypeFieldMapper.Defaults.FIELD_TYPE));
    return context;
}
```

总之，经过DocumentParser的处理，Nested Document多了这样的2个字段

- _id，值为父Document的id，用来关联
- _type，值为‘__’开头的，标志特定nested 类型。

### Nested类型普通Query如何隐藏

从上面存储知道，nested document是和普通Document一起存在索引中的，但对外是隐藏的，甚至是MatchAllQuery。

在`org.elasticsearch.common.lucene.search.Queries`类中：

```java
/**
 * Creates a new non-nested docs query
 * @param indexVersionCreated the index version created since newer indices can identify a parent field more efficiently
 */
public static Query newNonNestedFilter(Version indexVersionCreated) {
    if (indexVersionCreated.onOrAfter(Version.V_6_1_0)) {
        // ES 6.1.0之后。 只保留有_primary_term这个元字段的，所有的父Document都带有这个Field
        return new DocValuesFieldExistsQuery(SeqNoFieldMapper.PRIMARY_TERM_NAME);
    } else {
        // 老模式
        return new BooleanQuery.Builder()
            .add(new MatchAllDocsQuery(), Occur.FILTER)
            .add(newNestedFilter(), Occur.MUST_NOT)  //取非。过滤掉nested Document
            .build();
    }
}

public static Query newNestedFilter() {
    // _type以“__”为前缀
    return new PrefixQuery(new Term(TypeFieldMapper.NAME, new BytesRef("__")));
}

```

下面来看一下`_primary_term`是如何放进去的。这里涉及到`org.elasticsearch.index.mapper.SeqNoFieldMapper`这个类。他主要是用来为Document添加`_seq_no`元字段的，附带着把`_primary_term`给加了进去。

```java
@Override
protected void parseCreateField(ParseContext context, List<IndexableField> fields) throws IOException {
    // see InternalEngine.innerIndex to see where the real version value is set
    // also see ParsedDocument.updateSeqID (called by innerIndex)
    SequenceIDFields seqID = SequenceIDFields.emptySeqID(); // 默认SeqID
    context.seqID(seqID);
    fields.add(seqID.seqNo);
    fields.add(seqID.seqNoDocValue);
    fields.add(seqID.primaryTerm); // 添加到fields里
}
```



```java
public static SequenceIDFields emptySeqID() {
    return new SequenceIDFields(new LongPoint(NAME, SequenceNumbers.UNASSIGNED_SEQ_NO),
            new NumericDocValuesField(NAME, SequenceNumbers.UNASSIGNED_SEQ_NO),
            // 主要这里把_primary_term=0复制为了primaryTerm Field
            new NumericDocValuesField(PRIMARY_TERM_NAME, 0), new NumericDocValuesField(TOMBSTONE_NAME, 0));
}
```

下面，我们来看下，都什么地方使用到了`Queries.newNonNestedFilter`.这里我在IDEA里做了一个截图。

![ElasticSearch Nested](/images/elasticsearch-nested/elasticsearch-non-nested.png)

从上可以看出，基本的正常查询都默认加了这个filter的，只有是nestedQuery才做特别的处理。



**总结下如何隐藏的**：

| ElasticSearch版本 | 隐藏实现方式                               |      |
| ----------------- | ------------------------------------------ | ---- |
| 小于6.1.0         | 过滤掉_type以“__”为前缀的nested document   |      |
| 大于等于6.1.0     | 只获取有`__primary_term` Field的父Document |      |
|                   |                                            |      |



### 综述

无论是ElasticSearch，还是Solr, 其底层都是通过Lucene来做索引的。Lucene本身并不支持这种嵌套类型。ElasticSearch通过一个比较hack的方式支持了这样的功能，用起来更加的顺手，功能更强大。

同时，我们通过上面的分析，Nested也是有缺点的。

- nested无形中增加了索引量，如果不了解具体实现，将无法很好的进行文档划分和预估。ES限制了Field个数和nested对象的size，避免无限制的扩大
- nested Query 整体性能慢，但比parent/child Query稍快。应从业务上尽可能的避免使用NestedQuery, 对于性能要求高的场景，应该直接禁止使用。