### 问题说明

在我们的实际工作当中，更多的是用声明式事务去进行处理的，也就是在方法上面加一个@Transaction注解，代码示例如下所示：

```java
@Transactional
public void doTx() {
  // start tx
  
  // end tx
}
```

但是大家也要注意一点，这个注解的话其实在两种情况下它是会失效的。

**第一种情况：在方法内去调用的时候，因为它没有经过Bean的代理，所以它没办法依赖Spring的AOP增强**。

**第二种情况：在这个方法里面启了一个异步线程，异步线程如果里面有事务处理是没办法用事务去保证的**。

对于第二种情况，异步线程拿了连接和主线程拿到的连接肯定不是同一个，所以只有当一个数据库同一个连接才能去做事务控制。

这种异步线程的情况在spring里面的事务控制是不支持的

### 分布式事务

在一些场景下，可能有分布式事务问题，但是由于各种历史原因或者引入的成本太高又或者这个场景本身对一致性的要求并不是特别高，我们是尽量的去保证做到一致，并没有引入这种本地消息表、事务消息这种比较重的分布式事务实现。

另外，我们要尽量保证我们的事务尽量小，因为，开启关闭事务这个是有资源消耗成本的。另外就是数据库的连接沲它也有限的，如果有大事务它会一直持有这个连接不释放，对于整个线程沲的吞吐量这个是有影响的。所以，我们在写代码的时候要避免大事务，比如说：能批量的就尽量批量，不要用循环。也尽量不要在事务里面去做一些RPC这种比较耗时的操作。

### 业务代码中的事务

明确了上面几个问题之后，最近看到一些团队的事务代码发现一些问题：

**问题一：在事务里面去调用MQ的情况，而且在业务上其实本地事务提交成功了才希望去发这个MQ**

![截屏2022-11-26 上午10.45.04](/Users/sghl/Library/Application Support/typora-user-images/截屏2022-11-26 上午10.45.04.png)

针对上面的这种情况显示是有问题的，比如说本地事务回滚了，但是中间消息已经发出去了，但这个消息是没办法撤回的（发消息和本地事务是无法保证原子性）

![截屏2022-11-26 上午10.49.42](/Users/sghl/Library/Application Support/typora-user-images/截屏2022-11-26 上午10.49.42.png)

这里处理其实也非常的简单，就是将发送MQ消息放在本地事务执行之后，当然这个并不是分布式事务的解决方案。

![截屏2022-11-26 上午10.53.54](/Users/sghl/Library/Application Support/typora-user-images/截屏2022-11-26 上午10.53.54.png)

因为在极端情况下，本地事务提交了之后还没有发消息，突然机器重启了或者挂掉了，理论上来说也是有这种风险的 。所以这种方案并不是一种分布式事务的解决方案，这里更多的侧重点是优化代码结构，但是，基于前面提到的Spring的代理，我们必须要把发送MQ消息移动到方法外，并且从上层方法去进行一个调用，没办法在方法内去实现这个方法，这样才能基于Spring去做代理。所以，现在已有的这些代码如果要去做变更的话挪动的代码就会比较多。

我认为代码一方面你要告诉计算机它怎么去执行，另外一方面你也要让别人看得懂且易于理解。所以很多时候这个事务里面去发消息，确实在你的这个事务里面去发消息更容易得到别人的理解。我想说的是能不能就在这个声明式事务里面去完成这个代码的编写？能不能通过某种方式在本地事务执行成功之后再去做一个回调的操作，当然答案是肯定可以的。因为Spring它是提供了这种扩展的机制，Spring事务代码如下：

```java
public interface TransactionSynchronization extends Ordered, Flushable {
  
  // completion status in case of proper commit
  int STATUS_COMMITED = 0;
  
  // completion staus in case of proper rollback.
  int STATUS_ROLLED_BACK = 1;
  
  // completion status in case of heuristic mixed completion or system errros.
  int STATUS_UNKNOW = 2; 
}
```

这个类是就是Spring的事务这个包下面它提供的一个接口的扩展，这是一个事务同步回调的接口，基于这个事务管理器。而且，这个接口也有Order的能力，就是多个回调接口它可以控制回调的顺序，并且也提供了事务执行状态的几个常量(STATUS_COMMITED,STATUS_ROLLED_BACK)。

另外一个问题就是我们要判断当前上下文(Context)里面有没有事务，有事务的话我们就去做这么一个回调，没有的话就不去做任何处理。当然Spring同样也提供了这样的能力。

```java
public abstract class TransactionSychronizationManager {
	
  private static final Log logger = LogFactory.getLog(TransactionSychronizationManager.class);
  
  private static final ThreadLocal<Map<Object,Object>> resources = 
          new NamedThreadLocal<>("Transaction resources");
  
  private static final ThreadLocal<Set<TransactionSychronization>> synchronizations = 
    			new NamedThreadLocal<>("Transaction synchronizations");
  
  private static final ThreadLocal<String> currentTransactionName = 
    			new NamedThreadLocal<>("Current transaction name");
  
  private static final ThreadLocal<Boolean> currentTransactionReadOnly = 
    			new NamedThreadLocal<>("Current transaction read-only status");
  
  // Return whether there is an actual transcation active.This indicates whether the current
  // thread is associated with an actual transaction rather than just with active transaction synchronization.
  public static boolean isActualTranscationActive() { 
    return (actualTransactionActive.get() != null);
  }
  
  
  // Clear the entire transaction synchronization state for the current thread:registered synchroinzations
  // as well as the various transcation chracteristics.
  public static void clear() {
    synchronizations.remove();
    currentTransactionName.remove();
    currentTransactionReadOnly.remove();
    currentTransactionIsolationLevel.remove();
  }
}
```

这个抽象类里面它有**静态方法(isActualTransactionActive())**就能够判断当前有没有事务它被激活。

### 事务工具类编写

了解了Spring事务的扩展点后，我们就可以封装自己的工具类来实现，代码如下：

```java
public class TranscationUtils {
  
  @Transactional
  public void doTx() {
    // 开启事务
    TranscationUtils.doAfterTransaction(new DoTranscationCompletion(() -> {
      // 发送MQ消息或RPC
    }));
    // 结束事务
  }

  public static void doAfterTransaction(DoTranscationCompletion doTranscationCompletion) {
    // 判断上下文里面有没有事务激活，如果有则把这个回调接口注册进去
  	if (TransactionSychronizationManager.isActualTranscationActive()) {
       // 把SPI的实现去注册到当前的事务上下文里面的同步器时面
       TransactionSychronizationManager.registerSynchronization(doTranscationCompletion);
    }
  }

}

class DoTranscationCompletion implements TransactionSynchronization {
  
  private Runnable runnable;
  
  public DoTranscationCompletion(Runnable runnable) {
    this.runnable = runnable;
  }
  
  @Override
  public void afterCompletion(int status) {
    // 判断当前上下文，只有当事务成功提交的时候才去处理 
    if(status == TransactionSynchronization.STATUS_COMMITED) {
      this.runnable.run();
    }
  }
}
```



 
