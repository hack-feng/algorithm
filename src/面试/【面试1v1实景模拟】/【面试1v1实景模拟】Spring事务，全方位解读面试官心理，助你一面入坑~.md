### 老面👴：小伙子，了解Spring的事务吗？

解读🔔：这个必须了解，不了解直接挂~😂😂😂，但面试官肯定不是想听你了解两个字，他是想让你简单的介绍下。

笑小枫🍁：了解，事务在逻辑上是一组操作，要么执行，要不都不执行。主要是针对数据库而言的，比如说 MySQL。为了保证事务是正确可靠的，在数据库进行写入或者更新操作时，就必须得表现出 ACID 的 4 个重要特性。

---

### 老面👴：你刚刚提到了ACID，那你简单的介绍一下这4个特性

解读🔔：在介绍事务的时候，本可以把特性介绍了，但是偏偏不，故意一提，但不展开描述。为的就是让老面进入到我们的节奏😏😏😏虽然是你面试我，但我要牵着你的鼻子走，面试就是一场心理战，这才刚刚开始~

笑小枫🍁：ACID分别是**原子性（Atomicity）**、**一致性（Consistency）**、**事务隔离（Isolation）**、**持久性（Durability）**。

其中：**原子性**（Atomicity）是一个事务中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

**一致性**（Consistency）是在事务开始之前和事务结束以后，数据库的完整性没有被破坏。

**事务隔离**（Isolation）是数据库允许多个并发事务同时对其数据进行读写和修改，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。

**持久性**（Durability）是事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

正是因为这4个特性，我们在项目中会频繁的使用事务。

---

### 老面👴：你刚刚说，在项目中频繁使用事务，都是什么场景下使用的呢？

解读🔔：又跟着节奏来了，这个比较简单

笑小枫🍁：事务使用场景分为两种**读写事务**和**只读事务**。

我们常用的场景中，多表操作的时候会使用**读写事务**，例如A转账给B的时候，需要A扣款成功，

B到账成功，就要用到事务的原子性、一致性和持久性；在多表查询的时候，我们会用到**只读事务**，例如A有1000元，B有2000元，这个时候总金额应该为3000元。
~~~sql
select amount from user where id = 'A';
select amount from user where id = 'B';
~~~
如果查询完A后取到了1000元，然后A转账给B 1000元，如果没有事务，查询到B为3000元，这个时候总额就是4000元了，所以可以使用只读事务，保证数据的隔离性和一致性。

---

### 老面👴：你能说说具体是怎么使用的吗？

笑小枫🍁：事务使用一般有管理机制，一种是**编程式事务**，就是在代码中手动的管理事务的提交、回滚等操作，代码侵入性比较强，我们很少使用；另外一种是**声明式事务**，使用`@Transactional`注解，它是基于AOP面向切面的，将具体业务与事务处理部分解耦，代码侵入性很低，我们项目中使用这种方式比较多。

---

### 老面👴：你刚刚提到了Transactional，你知道它有几种作用域吗？

笑小枫🍁：可以作用在**类**、**方法**、**接口**上；

当把注解放在类上时，表示所有该类的public方法都配置相同的事务属性信息。

当类配置了@Transactional，方法也配置了@Transactional，方法的事务会覆盖类的事务配置信息。

不推荐作用在接口上，因为这只有在使用基于接口的代理时它才会生效，一旦配置了Spring AOP使用CGLib动态代理，将会导致@Transactional注解失效。

---

### 老面👴：除了作用在类上时，Spring AOP使用CGLib动态代理，@Transactional会失效外，你还知道其它的场景会导致事务失效吗？

解读🔔：这还不简单，这不就是我给你挖的坑嘛😏😏😏罗列几种情况吧，抽几条说就行，老面肯定会觉着你这小子基础不错

笑小枫🍁：

* **@Transactional 应用在非 public 修饰的方法上**

  如果Transactional注解应用在非public 修饰的方法上，Transactional将会失效。

  原因是：TransactionInterceptor （事务拦截器）使用AOP，在目标方法执行前后进行拦截，会检查目标方法的修饰符是否为 public，不是 public则不会获取@Transactional 的属性配置信息。（spring要求被代理方法必须是public的。）

* **@Transactional 注解属性 propagation 设置错误**
  这种是因为propagation的参数设置错误，根据需求选择合适的@Transactional的propagation属性的值

* **@Transactional 注解属性 rollbackFor 设置错误**

  rollbackFor默认是error和RuntimeException会触发回滚，其他异常或自定义异常不会回滚，需要回滚则需要指定或者直接指定@Transactional(rollbackFor=Exception.class)

* **同一个类中方法调用，导致@Transactional失效**

  这个和redis缓存失效场景之一一样，如果在一个类中，B方法加了事务，A方法没加事务调用B方法，另外一个类调用A方法，则B方法的事务失效。这种情况可以加一层Mamager层对Service层通用能力的下沉。

* **异常被捕获，导致@Transactional失效**

  当我们是用catch把异常捕获了，那么该方法就不会抛出异常了，那么出现该异常就不会回滚了

* **数据库引擎不支持事务**

  这种情况出现的概率并不高，事务能否生效数据库引擎是否支持事务是关键。常用的MySQL数据库默认使用支持事务的innodb引擎。一旦数据库引擎切换成不支持事务的myisam，那事务就从根本上失效了。

* **没有被 Spring 管理**

  如下面例子所示：

```java
@Service
public class OrderService {
    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
}
```
  如果此时把 @Service 注解注释掉，这个类就不会被加载成一个 Bean，那这个类就不会被 Spring 管理了，事务自然就失效了。

* **方法被定义为final**

  方法被定义成了final的，这样会导致spring aop生成的代理对象不能复写该方法，而让事务失效。

---

### 老面👴：小伙子不错呀，你刚刚说propagation设置错误，会导致事务失效，那你能说一下有哪几种propagation吗？

解读🔔：假装思考一下，让老面觉着你是一个知难不退，会思考问题的优秀人才，但不要太久，省的让老面

笑小枫🍁：propagation是用来配置Spring事务的传播机制，共有7种传播机制，声明式事务的传播行为可以通过 @Transactional 注解中的 propagation 属性来定义，默认是Propagation.REQUIRED。

Propagation的事务传播机制如下：

1. **REQUIRED**：如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。
2. **SUPPORTS**：如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。
3. **MANDATORY**：如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常。
4. **REQUIRES_NEW**：重新创建一个新的事务，如果当前存在事务，暂停当前的事务。 
5. **NOT_SUPPORTED**：以非事务的方式运行，如果当前存在事务，暂停当前的事务。 
6. **NEVER**：以非事务的方式运行，如果当前存在事务，则抛出异常。
7. **NESTED** ：表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为和 Propagation.REQUIRED 效果一样。

---

### 老面👴：除了propagation，你还知道 @Transactional 其他的属性吗？

笑小枫🍁：我们之前已经说了一些：
* **propagation**：可选的事务传播行为设置
* **readonly**：读写或只读事务，默认读写
* **rollbackFor**：导致事务回滚的异常类数组

除此之外@Transactional 还有以下的属性：

* **value**：可选的限定描述符，指定使用的事务管理器
* **isolation**：可选的事务隔离级别设置
* **timeout**：事务超时时间设置
* **rollbackForClassName**：导致事务回滚的异常类名字数组
* **noRollbackFor**：不会导致事务回滚的异常类数组
* **noRollbackForClassName**：不会导致事务回滚的异常类名字数组

---

### 老面👴：小伙子不错嘛，那你再简单的说说事务有哪几种隔离级别

解读🔔：一个Spring事务你能问这么多，问完这个该结束了吧，我快扛不住了~

笑小枫🍁：事务的隔离级别是通过Isolation属性控制的，共有5个属性，其中DEFAULT为默认值，代表使用底层数据库默认的隔离级别，其他的四个对应着数据库的4种隔离级别。分别是：

1. **DEFAULT：使用底层数据库默认的隔离级别。** MySQL的默认隔离级别：REPEATABLE_READ（可重复读）；Oracle默认的隔离级别：READ_COMMITTED（读已提交）
2. **READ_UNCOMMITTED：读未提交**（存在脏读、不可重复读、幻读）基本不使用
3. **READ_COMMITTED：读已提交**（解决度脏数据，存在不可重复读与幻读）
4. **REPEATABLE_READ：可重复读**（解决脏读、不可重复读，存在幻读）
5. **SERIALIZABLE：串行化** 。由于事务是串行执行，所以效率会大大下降，应用程序的性能会急剧降低。如果没有特别重要的情景，一般都不会使用 Serializable 隔离级别。

使用方法：@Transactional(isolation = Isolation.READ_UNCOMMITTED) 

---

### 老面👴：既然提到了 脏读、不可重复读、幻读，那你解释下分别是什么意思吧

解读🔔：这个还好，千万别问原理😥😥😥问原理我也不会~

笑小枫🍁：

**脏读（dirty reads）**，事务可以看到其他事务“尚未提交”的修改。如果另一个事务回滚，那么当前事务读到的数据就是脏数据。

**不可重复读（Non Repeatable Read）**，在一个事务内，多次读同一数据，在这个事务还没有结束时，如果另一个事务恰好修改了这个数据，那么，在第一个事务中，两次读取的数据就可能不一致。

**幻读（Phantom Read）**，幻读是指，在一个事务中，第一次查询某条记录，发现没有，但是，当试图更新这条不存在的记录时，竟然能成功，并且，再次读取同一条记录，它就神奇地出现了。

---

### 老面👴：最后，你再说下事务实现的原理吧

解读🔔：怕什么，来什么，巴拉巴拉巴拉，我也不会，面试到此结束，本文也到此结束啦~

当然言归正传，这个还是要自己去看源码，一步一步去理解，自己不理解，就算背会了，你背的答案也不一定对，而且说原理的过程也很多变，原理中也用到了很多技术点，还是要多学习，多积累，纯说原理很抽象~

笑小枫🍁：简单点说，Spring的事务实现的原理，就是通过这样一个动态代理对所有需要事务管理的Bean进行加载，并根据配置在invoke方法中对当前调用的方法名进行判定，并在method.invoke方法前后为其加上合适的事务管理代码，这样就实现了Spring式的事务管理。

详细的底层实现去这个地址看看吧~我就不班门弄斧了，肯定总结不到大佬这么全面😂😂😂

https://www.processon.com/view/link/5fab6edf1e0853569633cc06
