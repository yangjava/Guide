Easy-Es是一款简化ElasticSearch搜索引擎操作的开源框架,简化`CRUD`操作,可以更好的帮助开发者减轻开发负担

底层采用Es官方提供的RestHighLevelClient,保证其原生性能及拓展性.

技术讨论 QQ 群 ：247637156

微信群请先添加作者微信,由作者拉入 (亦可咨询健身问题,作者是健身教练)

项目推广初期,还望大家能够不吝点点三连:⭐Star,👀Watch,fork📌

支持一下国产开源,让更多人看到和使用本项目,非常感谢!

# 优点 | Advantages

------

- **屏蔽语言差异:** 开发者只需要会MySQL语法即可使用Es
- **代码量极少:** 与直接使用RestHighLevelClient相比,相同的查询平均可以节3-5倍左右的代码量
- **零魔法值:** 字段名称直接从实体中获取,无需输入字段名称字符串这种魔法值
- **零额外学习成本:** 开发者只要会国内最受欢迎的Mybatis-Plus语法,即可无缝迁移至Easy-Es
- **降低开发者门槛:** 即便是只了解ES基础的初学者也可以轻松驾驭ES完成绝大多数需求的开发
- **功能强大:** 支持MySQL的几乎全部功能,且对ES特有的分词,权重,高亮,地理位置Geo等功能都支持
- **完善的中英文文档:** 提供了中英文双语操作文档,文档全面可靠,帮助您节省更多时间
- **...**

# 对比 | Compare

------

> 需求:查询出文档标题为 "中国功夫"且作者为"老汉"的所有文档

```
    // 使用Easy-Es仅需3行代码即可完成查询
    LambdaEsQueryWrapper<Document> wrapper = new LambdaEsQueryWrapper<>();
    wrapper.eq(Document::getTitle, "中国功夫").eq(Document::getCreator, "老汉");
    List<Document> documents = documentMapper.selectList(wrapper);
    // 传统方式, 直接用RestHighLevelClient进行查询 需要11行代码,还不包含解析JSON代码
    String indexName = "document";
    SearchRequest searchRequest = new SearchRequest(indexName);
    BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
    TermQueryBuilder titleTerm = QueryBuilders.termQuery("title", "中国功夫");
    TermsQueryBuilder creatorTerm = QueryBuilders.termsQuery("creator", "老汉");
    boolQueryBuilder.must(titleTerm);
    boolQueryBuilder.must(creatorTerm);
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query(boolQueryBuilder);
    searchRequest.source(searchSourceBuilder);
    try {
         SearchResponse searchResponse = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
         // 然后从searchResponse中通过各种方式解析出DocumentList 省略这些代码...
        } catch (IOException e) {
            e.printStackTrace();
        }
```

> - 以上只是简单查询演示,实际使用场景越复杂,效果就越好,平均可节省3-5倍代码量
> - 传统功夫,点到为止! 上述功能仅供演示,仅为Easy-Es支持功能的冰山一角