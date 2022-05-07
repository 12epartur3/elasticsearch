1、es的分片保存着集群的一部分索引和文档，分片被分配到集群内的各个节点中，当集群规模扩大或者缩小时，es会自动在各个节点中迁移分片，使得数据任然均匀分布在集群中。我们可以发送请求到集群中的任一节点。
2、一个分片可以是主分片或者副本分片，副本分片是主分片的拷贝。
3、索引建立时就确定了主分片的数量，但是副本分片的数量可以随时修改。
4、一个节点可以保存多个分片。
5、读操作——搜索和返回数据——可以同时被主分片或副本分片所处理，所以当你拥有越多的副本分片时，也将拥有越高的吞吐量。
6、当我们之前讨论 index ， GET 和 delete 请求时，我们指出每个文档都有一个 _version （版本）号，当文档被修改时版本号递增。 Elasticsearch 使用这个 _version 号来确保变更以正确顺序得到执行。如果旧版本的文档在新版本之后到达，它可以被简单的忽略。所有文档的更新或删除 API，都可以接受 version 参数，这允许你在代码中使用乐观的并发控制，这是一种明智的做法。https://www.elastic.co/guide/cn/elasticsearch/guide/current/optimistic-concurrency-control.html
7、可以通过 _update参数来实现部分更新文档，这个操作能减少网络开销，在分片内部实际上还是重建了这个文档的索引。https://www.elastic.co/guide/cn/elasticsearch/guide/current/partial-updates.html
8、可以通过 _mget参数来实现一次获取多个文档。https://www.elastic.co/guide/cn/elasticsearch/guide/current/_Retrieving_Multiple_Documents.html
9、可以通过 _bulk参数来在单个请求中进行多次 create 、 index 、 update 或 delete 请求,实现批量操作。https://www.elastic.co/guide/cn/elasticsearch/guide/current/bulk.html
10、 每个节点都有能力处理任意请求。 每个节点都知道集群中任一文档位置，所以可以直接将请求转发到需要的节点上，
当发送请求的时候，为了扩展负载，更好的做法是轮询集群中所有的节点。https://www.elastic.co/guide/cn/elasticsearch/guide/current/how-primary-and-replica-shards-interact.html
11、es返回字段介绍：https://www.elastic.co/guide/cn/elasticsearch/guide/current/empty-search.html
12、支持同时在多个索引中搜索结果，如/index1*,index2*/_search。https://www.elastic.co/guide/cn/elasticsearch/guide/current/multi-index-multi-type.html
13、通过size和from参数来控制翻页，如GET /_search?size=5&from=10会查看第10~14条结果。https://www.elastic.co/guide/cn/elasticsearch/guide/current/pagination.html
14、es内的数据分为精确和全文（全文一般来说就是字符串类型）两种大类型，精确数据是二进制的，一般在指定字段（域名）查询时会采取精确匹配的方式，该过程不会进行标准化和分词处理，全文类型检索时会进行标准化和分词处理。https://www.elastic.co/guide/cn/elasticsearch/guide/current/_exact_values_versus_full_text.html
15、当索引一个文档的时候，Elasticsearch 取出所有字段的值拼接成一个大的字符串，作为 _all 字段进行索引。https://www.elastic.co/guide/cn/elasticsearch/guide/current/search-lite.html
16、通过参数 _analyze可以查看分词器效果。建立索引之前会对文本和查询进行标准化的流程，分析器将完成这个任务，分析器包装了三个功能：字符过滤器、分词器和Token过滤器，es附带了可以直接使用的分析器。https://www.elastic.co/guide/cn/elasticsearch/guide/current/analysis-intro.html
17、通过参数 _mapping可以查看各个域和其类型、用的分析器等信息，string类型（全文数据类型）域的 index属性默认是 analyzed。如果我们想映射这个string类型字段为一个精确数据类型，我们需要设置它为 not_analyzed，可以增加一个存在的映射，你不能修改存在的域映射，可以通过参数 GET /index/_analyze测试某个域的分析效果。https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping-intro.html
18、match、term、range是叶子语句，bool是复合语句，复合语句主要用于合并其他查询语句，复合语句可以合并其他任何查询语句，包括复合语句，这意味着复合语句之间可以相互嵌套，实现非常复杂的逻辑。一条复合语句可以将多条语句 — 叶子语句和其它复合语句 — 合并成一个单一的查询语句。https://www.elastic.co/guide/cn/elasticsearch/guide/current/query-dsl-intro.html
19、match_all 查询简单的匹配所有文档。在没有指定查询方式时，它是默认的查询。match既可以查询精确类型也可以查询全文类型，如果你在一个全文字段上使用 match查询，在执行查询前，它将用正确的分析器去分析查询字符串，如果在一个精确值的字段上使用它，例如数字、日期、布尔或者一个 not_analyzed字符串字段，那么它将会精确匹配给定的值，term查询被用于精确值匹配，这些精确值可能是数字、时间、布尔或者那些 not_analyzed的字符串，term查询对于输入的文本不分析 ，所以它将给定的值进行精确查询。multi_match支持在多个字段上执行 match操作。terms允许进行多值匹配。exists 查询和 missing 查询被用于查找那些指定字段中有值 (exists) 或无值 (missing) 的文档。查询 all_域直接用参数/_search?q=即可。https://www.elastic.co/guide/cn/elasticsearch/guide/current/_most_important_queries.html
20、bool查询语句：must，文档必须匹配这些条件，must_not，文档必须不匹配这些条件，should，如果满足这些语句中的任意语句，将增加 _score，否则无影响，主要用于修正每个文档的相关性得分，filter，必须匹配，但是以不评分、过滤模式来进行，这个语句对评分没有贡献，只用于过滤。如果没有 must语句，那么至少需要能够匹配其中的一条 should语句。但，如果存在至少一条 must语句，则对 should语句的匹配没有要求。constant_score查询可以取代只有filter的bool查询，在性能上是完全相同的，但是对于提高查询简洁性和清晰度有很大帮助。https://www.elastic.co/guide/cn/elasticsearch/guide/current/combining-queries-together.html
21、可以通过GET /index/_validate/query?explain接口来查看查询语法是否正确，查询语句的写法，1）先写query，再写match、match_all、term、terms、range。2）先写query，再写bool或者constant_score（一般只跟filter，强调不评分的查询，全部给1分），再写must、must_not、should、filter，里面再写match、term、range等。
22、可以通过sort参数对返回结果进行排序，返回的结果中也会带上sort的值，还可以通过指定mode实现多值字段的排序。https://www.elastic.co/guide/cn/elasticsearch/guide/current/_Sorting.html
23、一个字段可以通过配置fields参数来达成以两种方式来索引同一个字段的效果。https://www.elastic.co/guide/cn/elasticsearch/guide/current/multi-fields.html
24、相关性解释https://www.elastic.co/guide/cn/elasticsearch/guide/current/relevance-intro.html
25、Doc Values，应该就是文档的信息，因为文档值被序列化到磁盘，我们可以依靠操作系统的帮助来快速访问。当working set 远小于节点的可用内存，系统会自动将所有的文档值保存在内存中，使得其读写十分高速；当其远大于可用内存，操作系统会自动把Doc Values 加载到系统的页缓存中，从而避免了jvm 堆内存溢出异常。Doc Values通常用来排序、聚合、过滤地理位置好人脚本计算等。https://www.elastic.co/guide/cn/elasticsearch/guide/current/docvalues-intro.html
26、查询阶段：当一个搜索请求被发送到某个节点时，这个节点就变成了协调节点。 这个节点的任务是广播查询请求到所有相关分片并将它们的响应整合成全局排序后的结果集合，这个结果集合会返回给客户端。分片返回一个轻量级的结果列表到协调节点，它仅包含文档 ID 集合以及任何排序需要用到的值，例如 _score。协调节点将这些分片级的结果合并到自己的有序优先队列里，它代表了全局排序结果集合。至此查询过程结束。https://www.elastic.co/guide/cn/elasticsearch/guide/current/_query_phase.html
27、取回阶段：协调节点辨别出哪些文档需要被取回并向相关的分片提交多个 GET 请求，每个分片加载并 丰富 文档，如果有需要的话，接着返回文档给协调节点，每个分片加载并 丰富 文档，如果有需要的话，接着返回文档给协调节点。https://www.elastic.co/guide/cn/elasticsearch/guide/current/_fetch_phase.html
28、当索引一个文档的时候，文档会被存储到一个主分片中，这个过程是根据下面这个公式决定的：shard = hash(routing) % number_of_primary_shards，routing 是一个可变值，默认是文档的 _id ，也可以设置成一个自定义的值。 routing 通过 hash 函数生成一个数字，然后这个数字再除以 number_of_primary_shards （主分片的数量）后得到 余数 。这个分布在 0 到 number_of_primary_shards-1 之间的余数，就是我们所寻求的文档所在分片的位置。这就解释了为什么我们要在创建索引的时候就确定好主分片的数量 并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。https://www.elastic.co/guide/cn/elasticsearch/guide/current/routing-value.html
29、scroll 查询 可以用来对 Elasticsearch 有效地执行大批量的文档查询，而又不用付出深度分页那种代价。游标查询允许我们 先做查询初始化，然后再批量地拉取结果。 这有点儿像传统数据库中的 cursor 。游标查询会取某个时间点的快照数据。 查询初始化之后索引上的任何变化会被它忽略。 它通过保存旧的数据文件来实现这个特性，结果就像保留初始化时的索引 视图 一样。https://www.elastic.co/guide/cn/elasticsearch/guide/current/scroll.html
30、通过PUT index/id 的方式，可以将文章写入es并且自动创建索引，无需进行执行管理任务，如创建一个索引或指定每个属性的数据类型之类的，可以直接只索引一个文档。Elasticsearch 默认地完成其他一切，因此所有必需的管理任务都在后台使用默认设置完成。https://www.elastic.co/guide/cn/elasticsearch/guide/current/_indexing_employee_documents.html
31、通过PUT index 的方式，可以主动创建一个索引，比如我们想要确保这个索引有数量适中的主分片，并且在我们索引任何数据之前 ，分析器和映射已经被建立好。https://www.elastic.co/guide/cn/elasticsearch/guide/current/_creating_an_index.html
32、通过DELETE index 的方式，可以删除一个索引。https://www.elastic.co/guide/cn/elasticsearch/guide/current/_deleting_an_index.html
33、索引设置，创建和修改。https://www.elastic.co/guide/cn/elasticsearch/guide/current/index-settings.html
34、第三个重要的索引设置是 analysis 部分，用来配置已存在的分析器或针对你的索引创建新的自定义分析器。https://www.elastic.co/guide/cn/elasticsearch/guide/current/configuring-analyzers.html
35、虽然Elasticsearch带有一些现成的分析器，然而在分析器上Elasticsearch真正的强大之处在于，你可以通过在一个适合你的特定数据的设置之中组合字符过滤器、分词器、词汇单元过滤器来创建自定义的分析器。https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-analyzers.html
36、类型 在 Elasticsearch 中表示一类相似的文档。 类型由 名称 —比如 user 或 blogpost —和 映射 组成。映射, 就像数据库中的 schema ，描述了文档可能具有的字段或 属性 、每个字段的数据类型—比如 string, integer 或 date —以及Lucene是如何索引和存储这些字段的。类型可以很好的抽象划分相似但不相同的数据。但由于 Lucene 的处理方式，类型的使用有些限制。https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping.html
37、properties 节点，列出了文档中可能包含的每个字段的映射。_source 字段存储代表文档体的JSON字符串。和所有被存储的字段一样， _source 字段在被写入磁盘之前先会被压缩。然而，存储 _source 字段的确要使用磁盘空间。如果上面的原因对你来说没有一个是重要的，你可以映射禁用 _source 字段。在一个搜索请求里，你可以通过在请求体中指定 _source 参数，来达到只获取特定的字段的效果。Stored Fields 被存储字段：为了之后的检索，除了索引一个字段的值，你 还可以选择 存储 原始字段值。有 Lucene 使用背景的用户使用被存储字段来选择他们想要在搜索结果里面返回的字段。事实上， _source 字段就是一个被存储的字段。在Elasticsearch中，对文档的个别字段设置存储的做法通常不是最优的。整个文档已经被存储为 _source 字段。使用 _source 参数提取你需要的字段总是更好的。_all 字段：一个把其它字段值当作一个大字符串来索引的特殊字段。 query_string 查询子句(搜索 ?q=john )在没有指定字段时默认使用 _all 字段。如果你不再需要 _all 字段，你可以禁用。通过 include_in_all 设置来逐个控制字段是否要包含在 _all 字段中，默认值是 true。在一个对象(或根对象)上设置 include_in_all 可以修改这个对象中的所有字段的默认行为。https://www.elastic.co/guide/cn/elasticsearch/guide/current/root-object.html
38、当 Elasticsearch 遇到文档中以前 未遇到的字段，它用 dynamic mapping 来确定字段的数据类型并自动把新的字段添加到类型映射。有时这是想要的行为有时又不希望这样。通常没有人知道以后会有什么新字段加到文档，但是又希望这些字段被自动的索引。也许你只想忽略它们。如果Elasticsearch是作为重要的数据存储，可能就会期望遇到新字段就会抛出异常，这样能及时发现问题。
