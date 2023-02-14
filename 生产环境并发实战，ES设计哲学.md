### 1、引言

今天遇到一个线上问题，因为这个问题涉及到搜索引擎ES。我们都知道ES相比我们传统的关系型数据库它的一些差异。

ES既然定义为搜索引擎，首先强调的是它灵活的查询能力，但是关系型数据库它也支持用这种结构化的查询语言（SQL查询），那为什么我们还需要ES呢？

关系数据库它非常的成熟，它强调的是关系，而且绝大部分关系数据库它也支持事务的概念，所以如果要去满足它的这些特性（ACID），那对于数据扩容这一块是非常大的挑战。

搜索引擎它就是分布式的也是集群，它可以横向的去扩容它的数据节点的存储容量，因为它的倒排索引也有更强的这种文本索引的能力。

<img src="https://blog.arcanna.ai/hubfs/Imported_Blog_Media/61be38b25fdb7878686b93e0_605c9c8427508704ace5ef96_ES_shards.png" alt="arcanna.ai" style="zoom:50%;" />

<img src="https://aravind.dev/static/3768f3f352da47aad193daae9e2df260/c0566/es-data-mapping.png" alt="Everything you need to know about index in Elasticsearch! | Aravind Putrevu" style="zoom:50%;" />

那既然是分布式的，所以它对一致性的要求就相对于关系数据库要较低一些，它就比较适合一些大数据量、模糊搜索、全文匹配的这些场景，而不太适合一些关系数据库它强调的强事务这些场景。

了解了ES的特性后，我来分享一下前两天在线上的遇到的ES问题：前两天操作了一个ES的接口，突然报一个版本异常的错误，错误信息如下所示：

```]
primary term[5]] method_name:saveDoc exception_type:org.elasticsearch.ElasticSearchException
[type=version_confict_engine_exception,reason=[8039961662]]:version confict,rquired sqlNo[1773199],primary term[5].current document has seqNo[1788475]and primary term [5]]at org.elasticsearch.rest.BytesRestResponse.errorFromXContent(BytesRestResponse.java:187)at
```

通过这个文案的意思我们大概猜测一下它的意思：它预期的是一个版本号，但是实际上存在的是另外一个版本号。带着这个问题于是去搜索了一下这个关键字，发现并不是在客户端这边报出来的。这个接口的逻辑也是比较简单，其实就是更新一个文档的字段。ES中的文档就相当于关系型数据库表里面的一行记录，这里的场景就是更新关系数据库里面的一个字段，并且同步更新ES里面的一个字段。

虽然，ES的服务端它也是使用Java编写的，我们可以在github上把它的源代码clone下来，但是这样做可能效率有点低。于是我就拿着这个关键字去官方的社区去查了一下。其实在更新的时候是采用这种乐观锁的机制。

> 乐观锁：先查出来一个版本，然后去更新的时候这个过程是没有加任何锁的，只需要在更新的时候去对比一下中间有没有被别人更新（通过一个版本号的方式），如果没有其它并发的线程更新，那就可以更新成功。如果有被其它线程更新那就更新失败。
>
> 悲观锁：查询出来结果之后，就会一直锁住这个资源，其它任何资源都是不能去更新的。

了解了乐观锁和悲观锁的区别之后，所以我们提倡如果更新并发不是特别高的情况尽量就用乐观锁，这样可以降低锁资源的竞争。但是，如果更新的并发量很高，频繁的CAS失败，这个资源的开销可能也是比较大的。

![Optimistic vs. Pessimistic locking in Rails | by Rohan Daxini | Medium](https://miro.medium.com/max/1022/1*8MxJby4K_drkloWWZW0wpw.jpeg)

而且，CAS失败了之后是不断的重试还是再去加一个悲观锁。所以整体考虑下来如果更新的并发很高那就可能加悲观锁它反而整体的吞吐量会更高一些。

### 2、ES官方说明

了解了背景之后，我们来看一下ES的官方文档对于乐观锁的实现的说明：

> 乐观并发控制：当一个文档创建、更新或删除的时候，会生成一个版本复制到集群中的其他节点。Elasticsearch 也是异步和并发的，这意味着这些复制请求是并行发送的，并且可能会乱序到达目的地。Elasticsearch 需要一种方法来确保文档的旧版本永远不会覆盖新版本。
>
> 为确保文档的旧版本不会覆盖新版本，对文档执行的每个操作都由协调该更改的主分片分配一个序列号。序列号随着每个操作而增加，因此新操作保证比旧操作具有更高的序列号。然后，Elasticsearch 可以使用操作的序列号来确保较新的文档版本永远不会被分配给它的序列号较小的更改覆盖。

- ES并发控制：https://www.elastic.co/guide/en/elasticsearch/reference/current/optimistic-concurrency-control.html
- ES版本支持：https://www.elastic.co/cn/blog/elasticsearch-versioning-support

ES提供了两种版本的方式，一种是它自动生成的，另外一种是自定义的(外部版本号)。

如果出现版本冲突，那我们的解决方案是什么？

### 3、版本冲突解决方案

对于版本冲突的情况，可以采用让业务方直接就报错，当然也可以采用一些其它的机制，比如：重试次数，如果并发不是特别高的情况采用一定的重试次数，这也是一定程度上能够缓解这个问题的。

明确了这种异常之后，于是我去确认了一下我们的业务场景确实存在这种并发更新同一个文档的情况，那我们如何来解决这个问题呢？

首先想到的第一个解决方案是加一个分布式锁，但是这并不是一个很好的解决方案，因为它违背了ES的设计初衷：ES为什么要设计成乐观锁而不设计成悲观锁，强调这种强大的索引能力、搜索能力，它能解决一定程度的高并发问题，但是它只限于这种读多写少的流量情况。

如果你是写入的高并发它其实是支持并不好的，所以对于写它采用这种乐观锁的机制，如果你在它乐观锁的基础上去加一个悲观锁就违背了它的设计初衷了。当然，官方并没有说明更新的时候使用悲观锁的方式去更新。

结合我们自身的业务场景：因为之前的一个改动导致这个接口的调用它变成了一个多线程的方式，才有这种并发更新同一个文档的情况。

所以，我建议尽量在业务上去规避掉这种并发更新的场景，就像我们要去保证消息队列消费消息的顺序性一样，我们尽量去保证它的一个partition一个线程去消费。

你在业务上这样去规避了之后当然也不能保证你完全就不会冲突，可能有一些网络超时、重试导致这些并发的情况，那你这个时候就加一个重试次数一般就可以解决这个问题了，ES的客户端它是可以支持你定义一个重试次数的，代码如下：

```java
/**
 * Sets the number of retries of a version conflictoccurs because the document was updated between
 * getting it and updating it.Defaults to 0.
 */
public UpdateRequest retryOnConfict(int retryOnConfict) {
  this.retryOnConfict = retryOnConfict;
  return this;
}
```

我认为通过这两种方式就可以解决这个问题了，但是如果你业务上就是要高并发的去更新同一个文档，我个建议就不要直接去调ES了。

这个场景可以采用先缓存到一个中间的地方，比如说可以使用Redis或者Mysql把一个中间结果给它做一个合并，你再去更新ES。

