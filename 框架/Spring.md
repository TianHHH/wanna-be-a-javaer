# 介绍一下IOC

IOC 的全称是 "Inversion of Control" **(也就是控制反转), 它不是一种具体的技术, 而是一种设计思想**. 其中控制指的是对象创建的权利, 反转就是指将控制权交给外部的 IOC 容器

**<font style="color:blue">使⽤ IOC 后就不需要自己去创建某个类的实例, 而是由 IOC 容器去创建</font>**. 当再需要使用某个对象时, 直接到容器中去获取就可以了

另外将对象之间的相互依赖关系交给 IOC 容器来管理, 并由 IOC 容器完成对象的注入. 这样可以很大程度上简化应用的开发, 把应用从复杂的依赖关系中解放出来

## 和DI的区别

DI, 也就是依赖注入, 它是 IOC 的一种主要实现方式. 所谓依赖注入, **就是<font style="color:blue">通过外部把一个对象所依赖的其它对象注入到其中</font>, 而不是在类内部创建依赖对象**

DI 的实现方式主要有通过**构造函数注入、Setter 方法注入或者直接在字段上注入**. 这个注入的过程, 正是容器在替程序员管理控制权的体现

IOC 和 DI 其实只是同⼀个概念的不同⻆度描述, 都是用来实现对象解耦的. 只不过 IOC 是一种设计思想, 而 DI 是一种具体的实现技术

# @Autowired和@Resource的区别

1. @Autowired 是Spring提供的注解; @Resource 是 JDK (JSR-250) 提供的注解
2. **<font style="color:blue">@Autowired 自身只能按类型注入; </font>@Resource 默认按名称注入, 也支持根据类型注入**
3. `@Autowired`默认情况下要求依赖对象必须存在, 但是如果想设为非必须可以通过**设置required属性为false**来实现. **<font style="color:blue">`@Resource`虽然默认情况下也是要求依赖对象必须存在, 但不能设为非必须</font>**
4. **如果想让 @Autowired 支持按名称装配, 需要搭配 @Qualifirer 使用**. 一起使用时 Spring 会先按类型获取所有候选Bean, 然后在这些候选中用 @Qualifier 指定的名称进行精确匹配


> **@Qualifier 是一个限定符注解, 不能单独使用**, 必须配合 @Autowired 或 @Inject 才能发挥作用. **<font style="color:blue">另外该注解有强约束过滤作用</font>** (**即使类型唯一匹配, 只要名称不匹配也不会注入, 而是抛出异常**)

5. **@Resource 有两个重要的属性: name 和 type**. 如果没有指定 name 属性, 当注解标注在字段上即默认取字段的名称作为Bean名称寻找依赖对象; 当注解标注在属性的 setter ⽅法上, 即默认取属性名作为Bean名称寻找依赖对象 (属性名 = 去掉方法名的 set 前缀, 然后将首字母小写)
- **如果两个属性都没指定的话就会<font style="color:blue">先按默认名称查找Bean, 按照默认的名称找不到依赖对象的话就会回退到按类型装配 (默认行为)</font>**


   - **单独指定name属性的话, 就只能按名称进行装配; 单独指定type属性的话就只能按照类型装配**; 两个都指定时, 必须同时满足指定的name和type才能注入成功

> 当一个接口存在多个实现类的情况下:
>
> @Autowired 和 @Resource 都需要通过名称才能正确匹配到对应的Bean

## 补充说明

JSR 全称是 **Java Specification Request**

翻译成中文就是 **Java 规范提案**, 也可以叫 Java 官方标准草案. **它是由 Java 官方**(以前是 Sun, 现在是 Oracle 和 Jakarta EE 社区 --  Sun 被 Oracle 公司收购了)**<font style="color:blue">提出的一个个标准, 每个标准都有一个编号</font>**

@PostConstruct 和 @PreDestory 就是 JSR-250 定义的规范注解

可以理解成 Spring 只是说:" 哦, JSR-250 是官方标准, 那我也支持这个行为". 所以 Spring 没有发明这个注解, 它只是**遵守标准**. 换一个框架用这个注解也一样能跑

总结: JSR 告诉大家某个功能应该怎么设计; 怎么命名, Spring / Hibernate 等框架不是发明它们, 而是"支持这些标准", 这样写的代码就能在不同框架中通用

# Spring的常用注解

**声明Bean的注解**: `@Component`(标注普通的Bean类), `@Controller`(标注控制层组件), `@Service`(标注业务层组件), `@Repository`(标注数据访问层组件), `@Bean`(标注在方法上, 声明当前方法的返回值为一个Bean)

**注入Bean的注解**: `@Autowired`(由Spring提供), `@Resource`(JSR-250规范定义的)

**配置类相关的注解**: `@Configuration`(声明当前类为配置类), `@ComponentScan`(对选定范围内的Bean进行扫描), `@Import`(导入其它配置类)

Spring AOP相关注解: 

- `@EnableAspectJAutoProxy`(**开启Spring对AspectJ注解的支持**, **<font style="color:blue">SpringBoot项目默认已经开启了AOP自动代理功能, 所以一般不需要加入该注解</font>. 除非要配置一些额外的行为**: 比如强制使用 cglib 动态代理; 允许通过`AopContext.currentProxy()`获取当前代理对象之类的)
- ` @Aspect`: 声明一个切面类, `@Pointcut`: 定义切入点表达式
- 以及标注各种通知类型的注解 (`@Around`、`@Before`等等)

**Spring声明式事务的注解: `@Transactional`**

# Spring事务相关问题

## Spring中事务的分类

> 参考博客: https://blog.csdn.net/code_nn/article/details/143319826

Spring支持两种事务方式

分别是编程式事务(Declarative Transaction)和声明式事务(Programmatic Transaction), 后者最常见

- **声明式事务就是将事务管理的代码从业务方法中抽离出来**, **<font style="color:blue">通过配置或注解的方式, 让 Spring 自动帮开发者管理事务</font>**, 开发者不用关心事务的开始、提交以及回滚细节. 有两种实现方式:

  - 注解方式(最常用): 将`@Transactional`注解加在方法 / 类上. **<font style="color:blue">加在类上就代表类中所有满足被代理条件的方法都默认启用事务</font>**
    - **`@Transactional`底层就是通过AOP实现事务控制的**. Spring 会扫描带有 `@Transactional` 注解的方法或类, 对这些方法通过 JDK / CGLIB 动态代理自动生成代理对象
    - **Spring 会根据是否实现接口来决定使用哪种代理方式**: 如果目标类实现了接口, 就使用 JDK 动态代理(基于接口); 目标类没有实现接口, 就使用 CGLIB 代理
    - 调用方法时走代理逻辑, 在方法执行前开启事务, 执行后提交或回滚
  - XML方式: (现在很少用) 写在Spring的配置文件中. 比如`applicationContext.xml`(惯例命名)

  ```xml
  <tx:advice id="txAdvice" transaction-manager="txManager">
      <tx:attributes>
          <tx:method name="save*" propagation="REQUIRED"/>
          <tx:method name="*" read-only="true"/>
      </tx:attributes>
  </tx:advice>
  ```

---

- **编程式事务: <font style="color:blue">将事务管理的代码嵌入到业务代码中, 开发者手动编写事务的控制逻辑</font>**. 适用于需要更精细地控制事务的场景. 但因为代码中混杂了业务逻辑和事务逻辑, 会导致代码很臃肿

  - 另外因为需要手动控制事务的提交与回滚, 遗漏的话会导致出错

- 实现方式主要有两种: 

  - `TransactionTemplate` (Spring更推荐这种方式), 该类本质上是对`PlatformTransactionManager`进行了封装, **<font style="color:blue">可以自动提交 / 回滚</font>** (但因为需要写出事务的边界逻辑 -- `execute()`方法, 仍然显式参与了事务逻辑控制, 所以并不能归属于声明式事务)

  ```java
  @Autowired
  private TransactionTemplate transactionTemplate;
  
  public void testTransaction() {
  
      transactionTemplate.execute(new TransactionCallbackWithoutResult() {
        	/**
        		该方法中不是必须使用try-catch代码块, 但是推荐使用. 因为:
        		该方法中抛出的RuntimeException(或其子类), Spring会自动rollback; 但是对于编译时异常(如IOException), 默认不会回滚, 除非手动调用
        	*/
          @Override
          protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
              try {
                  // ....业务代码
              } catch (Exception e){
                	/*
                		setRollbackOnly()只是告诉Spring本事务不能成功提交, 请在事务结束时自动回滚 (即真正的回滚是在execute()方法结束之后, 所以后续逻辑不要再进行有副作用的操作(如发消息等))
                		最终rollback是由Spring来调用的, 这里只是打了一个标记
                	*/
                  transactionStatus.setRollbackOnly();
              }
          }
      });
      // java 8以上版本可以使用Lambda表达式进行简化
      /*
        transactionTemplate.execute(status -> {});
      */
  }
  ```

  - `PlatformTransactionManager`, **这只是接口**. 通过这个接口, Spring为各个平台都提供了对应的事务管理器, 但是具体实现就是各个平台自己的事情了. 比如:
    - **Sping JDBC 和 MyBatis 使用 `DataSourceTransactionManager`**
    - Hibernate 使用 `HibernateTransactionManager`

  ```java
  @Autowired
  private PlatformTransactionManager transactionManager;
  
  /*
  	不需要也不能显式关闭事务, 因为事务的提交或回滚就是事务的关闭
  	commit()或rollback()被调用后, Spring会自动清理当前事务上下文, 把资源(连接等)释放回连接池
  */
  public void testTransaction() {
    TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
            try {
                 // ....  业务代码
                transactionManager.commit(status);
            } catch (Exception e) {
                transactionManager.rollback(status);
            }
  }
  ```

Spring将事务管理的核心抽象为一个事务管理器(`TransactionManager`), `PlatformTransactionManager`就是继承自这个接口. **<font style="color:blue">它是一个标记接口(marker interface), 本身没有任何方法</font>**

> 它还有一个子接口是`ReactiveTransactionManager`, 这是 **Spring 5+ 为响应式(Reactive)编程模型**引入的新接口, 专门为 `WebFlux` / `R2DBC`这类异步非阻塞场景设计

---

补充说明:

`TransactionDefinition`是**Spring事务管理中的"事务属性定义接口"**, 用于描述每个事务的**行为参数**. 说白了就是每次开启事务时的"参数模板", 告诉 Spring 这个事务是什么传播行为、隔离级别、是否只读等

- `getPropagationBehavior()` -- 事务的传播行为
- `getIsolationLevel()` -- 事务的隔离级别
- `isReadOnly()` -- 是否是只读事务
- `getTimeout()` -- 获取超时时间, 单位是秒, 默认值为 `-1`(永不超时)
- `getName()` -- 事务的名称 (用于日志 / 调试, 可选)

> **`DefaultTransactionDefinition`**: 最常用的实现类, 在用编程式事务时一般都会实例化该类的对象



**`TransactionStatus`主要用于描述当前事务的运行状态**, 可以通过它来: (**<font style="color:blue">它不能直接控制事务提交或回滚</font>**)

- 判断当前事务是否是新建的 -- `isNewTransaction()`

- 手动设置事务为`rollback-only` -- `setRollbackOnly()`

- 判断事务是否被标记为`rollback-only` -- `isRollbackOnly()`
- 判断事务是否已经提交或回滚完毕 -- `isCompleted()`

## Spring事务的传播行为

**`Propagation.PROPAGATION_REQUIRED`**

> 对应 `TransactionDefinition.PROPAGATION_REQUIRED`(底层 int 值为 0)

**<font style="color:blue">这是`@Transactional`默认的事务传播行为</font>**

- **外部方法没有事务时, 每个单独的`REQUIRED`方法会新建一个事务**, 各自的提交和回滚互不影响
- **如果外部方法开启事务并且是`REQUIRED`的话, <font style="color:blue">所有`REQUIRED`修饰的内部方法和外部方法均属于同一事务</font>**, 只要一个方法回滚, 整个事务都需要回滚

- 如果中间有一个方法传播行为不是`REQUIRED`, 则要看它的具体传播属性, Spring 会根据传播策略决定"是加入 / 挂起 / 新建事务还是抛出异常"

---

**`Propagation.REQUIRES_NEW`**: **<font style="color:blue">无论当前是否存在事务, 都会新建一个事务</font>**. **如果存在外部事务, 则挂起外部事务, 等当前新事务完成后再恢复外部事务**

> 对应的是`TransactionDefinition.PROPAGATION_REQUIRES_NEW` (底层int值为3)

```java
Class A {
    @Transactional(propagation=Propagation.PROPAGATION_REQUIRED)
    public void aMethod {
        // do something
        B b = new B();
        b.bMethod();
    }
}

Class B {
    @Transactional(propagation=Propagation.REQUIRES_NEW)
    public void bMethod {
       // do something
    }
}
```

**如果 `aMethod()` 发生异常回滚, `bMethod()` 不会跟着回滚**, 因为 `bMethod()` 开启了独立的事务



但是要注意: 如果 `bMethod()` 抛出了未被捕获的异常并且这个异常满足事务回滚规则的话, `aMethod()` 同样也会回滚. **<font style="color:blue">但本质并不是因为事务传播链互相影响</font>**

- 而是因为异常从 `bMethod()` 直接抛回了 `aMethod()` -- 即异常会**一路上传到 `aMethod()`**
- Spring事务管理的默认回滚规则是: **遇到未捕获的 RuntimeException 或 Error, 就回滚当前方法所在的事务**. 对于**受检异常(Checked Exception)**则不会自动回滚
  - 如果你想要回滚特定的异常类型的话, 可以使用 `rollbackFor` 属性显式指定: 


```java
@Transactional(rollbackFor = MyException.class)
```

---

**`Propagation.NESTED`**: 如果当前已经存在事务, **则在当前事务内开启一个嵌套事务(通过 Savepoint 实现). 如果没有事务, 则与 REQUIRED 表现一致, 新建一个事务**

> 对应 `TransactionDefinition.PROPAGATION_NESTED` (底层 int 值为 6)

- **<font style="color:blue">嵌套事务不是完全独立的新事务, 而是在当前事务中加了"保存点(Savepoint)"</font>**
- 如果嵌套事务抛异常回滚, 只会回滚到 savepoint, **外部事务可以选择继续提交或回滚**

```java
@Service
public class BatchService {
    @Autowired
    private ItemService itemService;

    @Transactional
    public void batchProcess() {
        itemService.processItemA(); // 成功
        try {
            itemService.processItemB(); // 可能失败, 只回滚自己
        } catch (Exception e) {
/*
  虽然NESTED方法内部抛出异常, 但被外层 try-catch 捕获, Spring只会回滚到savepoint
  只有遇到未被捕获的 RuntimeException 或 Error, 事务才会回滚
*/
            // 这里捕获异常, 不影响整体提交
        }
        itemService.processItemC(); // 成功
        // 最终 batchProcess() 成功提交时, A、C数据都保留, B出错的数据被回滚
    }
}

@Service
public class ItemService {
    @Transactional(propagation = Propagation.NESTED)
    public void processItemB() {
        // ... 失败时抛异常, 只回滚本方法所做的操作
    }
} 
```

---

**`Propagation.MANDATORY`**: **<font style="color:blue">必须在一个已有事务中运行. 如果当前没有事务, 则抛出异常</font>**

- **如果当前存在事务, `MANDATORY`方法会加入当前事务**, 行为和 REQUIRED 一样
- 如果当前没有事务, Spring 会直接抛出异常, 而不是自动新建事务

> 对应 `TransactionDefinition.PROPAGATION_MANDATORY` (底层 int 值为 2)

```java
@Service
public class AuditService {
    @Transactional(propagation = Propagation.MANDATORY)
    public void auditOperation() {
        // 只能在已有事务环境下调用
    }
}

// 正确用法
@Transactional
public void businessOperation() {
    auditService.auditOperation(); // 没问题, 外层有事务
}
```

`Propagation.NEVER`正好相反: **<font style="color:blue">表示当前不能存在事务</font>**

- **有事务环境时, 抛异常**, 不允许方法在事务中被调用
- 没有事务环境时, 方法正常运行, 只是**不受事务保护**

> 对应 `TransactionDefinition.PROPAGATION_NEVER` (底层 int 值为 4)

---

`Propagation.SUPPORTS`: **<font style="color:blue">如果当前存在事务, 则加入该事务; 如果当前没有事务, 则以非事务的方式继续运行</font>**

> 对应 `TransactionDefinition.PROPAGATION_SUPPORTS` (底层 int 值为 1)

- 即如果外层已经有事务, `SUPPORTS`方法会加入当前事务, 受统一事务控制



`Propagation.NOT_SUPPORT`: **<font style="color:blue">遇到外层方法有事务时就"主动挂起事务"</font>**, **本方法和外层事务完全隔离, 不受事务控制**; 外层没有事务时, 则以非事务的方式继续运行

- 挂起就是暂时让外层事务"失效", 让当前方法完全不在任何事务环境下, 执行完毕后再继续之前的事务流程. 如: 事务A(开启) -> B方法(非事务，执行完) -> 事务A(恢复继续)

### 补充说明

Q: 没加 @Transactional 的方法被加了事务的方法调用时, 事务行为是什么样的?

```java
@Transactional
public void transactionalMethod() {
    // ... some code
    nonTransactionalMethod();  // 这个方法没有 @Transactional
    // ... more code
}

public void nonTransactionalMethod() {
    // ... some code
}
```



A: 事务边界由外层加了 @Transactional 的方法控制, **`nonTransactionalMethod()`运行期间, 依然在这个事务的上下文中**, 本质上和 `transactionalMethod()` 的其他代码没有区别, 只是这个方法没有显式声明事务而已

- 即: **<font style="color:blue">只要是同一个调用栈, 外层事务开启后</font>, 里面所有代码(不管有没有加`@Transactional`)都在这个事务里运行**

## Spring中事务的隔离级别

TransactionDefinition 中一共定义了5种事务隔离级别:

- **`ISOLATION_DEAFULT`: 使用数据库默认的隔离级别, <font style="color:blue">依赖于具体数据库</font>**
  - 如Oracle是`READ_COMMITTED`, MySQL是`REPEATABLE_READ`
- `ISOLATION_READ_UNCOMMITTED`: 最低的级别, 无法防止任何并发问题, **可能会出现脏读 / 幻读或者不可重复读**. 效率最高但不安全
- `ISOLATION_READ_COMMITTED`: 只能读取已提交数据, 但仍然可能发生不可重复读或幻读

- `ISOLATION_REPEATABLE_READ`: 对同一字段的多次读取结果都是一致的, 除非数据是被自身事务所修改. 可以阻止脏读和不可重复读, 但幻读仍有可能发生
- `ISOLATION_SERIALIZABLE`: 最高的隔离级别, 事务串行执行. 虽然可以阻止脏读 / 幻读和不可重复读, 但会严重影响程序性能

---

**这些常量就是在 @Transactional 注解中设置 isolation 参数的取值来源**, 例如:

```java
// 其中的 Isolation.REPEATABLE_READ 实际上是引用了: 
// TransactionDefinition.ISOLATION_REPEATABLE_READ
@Transactional(isolation = Isolation.REPEATABLE_READ)
```

> **原理简述: Spring 在开启事务时, 会通过 `PlatformTransactionManager`**
>
> 比如 `DataSourceTransactionManager` **读取 TransactionDefinition 中的这些配置, <font style="color:blue">然后调用底层 JDBC 的 API 来设置真正的数据库隔离级别</font>**
>
> ```java
> connection.setTransactionIsolation(int level)
> ```

---

补充说明:

**Spring 并不重新实现事务, 而是<font style="color:blue">通过 TransactionDefinition 把隔离级别等事务特性抽象出来, 转交给底层数据库执行</font>**

- **即 Spring 的常量只是一个中间层抽象, 真正起作用的还是底层数据库的事务机制**

Spring在每个事务开始前, 会获取当前数据库连接, 然后通过 JDBC API 设置当前连接的隔离级别

- 所以虽然在代码层"看起来像是每个方法设置不同的隔离级别", **但本质依然是"给每次使用的数据库连接设置隔离级别", <font style="color:blue">所以还是会话级别的实现</font>**

## Spring事务的失效场景

1. **方法定义问题**: Spring 事务依赖 AOP 代理, 而代理不能拦截非 public 方法以及 final / static 修饰的方法, 因此这些方法上的 @Transactional 注解不会生效
   - **<font style="color:blue">解决方案: 确保事务方法为 public 且不被 final / static 修饰</font>**

> protected修饰的方法要看情况: 如果使用的是CGLIB代理(即当前类没有实现接口, 或配置强制使用 CGLIB)就可以生效. 如果是JDK动态代理的话就不会生效
>
> **但为了事务安全 & 兼容性和可维护性, 永远使用  public 方法来声明事务**. 即使 protected 方法有可能生效, **也不是标准做法**

2. **同类方法互调(自调用)**: **当一个@Transactional方法<font style="color:blue">在同一类中被另一个方法直接调用时, 不会通过代理对象进行调用</font>, 事务无法生效**

   - Spring事务依赖代理对象对方法调用做"切面增强", **如果用 this.method() 调用另一个事务方法, 是直接调用原始类代码**, 绕过了 Spring 的代理机制, 事务逻辑完全不会触发

   - 解决方案: 

     - **<font style="color:blue">将事务方法抽取到另一个类中, 通过依赖注入调用</font>**
     - **通过 `AopContext.currentProxy()` 获取当前代理对象**

     ```java
     // 需要启用 exposeProxy = true (只能在配置类 / 启动类中使用)
     @EnableAspectJAutoProxy(exposeProxy = true)
     ```

     - 使用 `TransactionTemplate` 手动控制事务

3. 异常类型不匹配: Spring默认只在出现 `RuntimeException` 或 `Error` 时才会回滚事务. **<font style="color:blue">如果抛出的是检查异常(也叫编译时异常)不会触发事务回滚</font>**

   - 解决方案: **通过 rollbackFor 指定**需要回滚的异常类型, 比如:

   ```java
   @Transactional(rollbackFor = Exception.class)
   ```


4. 多线程环境: **Spring事务仅在当前线程内有效**, 若在一个事务方法中启动新线程, 则新线程不受事务的控制. 这是**<font style="color:blue">因为新线程中的操作脱离了原事务的上下文</font>**

   - Spring使用一个叫 `TransactionSynchronizationManager` 的类将事务状态绑定到当前线程, **也就是基于 ThreadLocal 绑定. 而子线程是新的线程, ThreadLocal 是线程隔离的**, 子线程没有原始事务的上下文, 自然也不会被 AOP 增强

   - 解决方案:
     - **<font style="color:blue">在子线程中手动使用编程式事务进行管理</font>**
     - 避免在事务中创建新线程
     - **将目标方法上加入 @Async, 使其在新线程中执行** -- **加上该注解就不需要手动开启新线程了**, Spring 会自动将方法提交到它管理的线程池中执行. 同时再加上 @Transactional, **使得该方法在新线程中也能开启自己的独立事务**

> 使用 @Async 时**新线程中开启的事务和原事务是两个完全独立的事务上下文**
>
> **新事务出错回滚**, 不会影响原事务; **原事务回滚**, 也不会影响已提交的异步事务

5. 未使用 Spring 管理的类: **<font style="color:blue">若在未被 Spring 管理的类中使用 @Transactional, 事务不会生效</font>**, 因为 Spring 无法对该类生成代理
   - 解决方案: **确保 @Transactional 用于由 Spring 管理的 Bean**
6. 事务的传播行为设置不当: 在使用 Spring 的事务传播机制时, 像 `REQUIRES_NEW` 这样的传播行为会挂起外部事务, 并开启一个完全独立的新事务. 因此即使外部事务回滚, 内部事务也不会被回滚
   - 因此**在设置事务传播属性时, 必须根据业务需要明确是要"隔离"还是"嵌套"行为, 以防事务失效或产生意外提交 / 回滚的问题**
7. 事务超时时间设置不当: 当给一个事务设置了 timeout 属性, 但是它没有在规定时间内完成的话, Spring会抛出`TransactionTimedOutException`, 并自动触发回滚 (timeout 的单位是**秒**)
   - 这种场景较常见于执行较长时间的数据库操作
   - 解决方案: 为长时间任务合理设置事务超时时间

8. 某些数据库和存储引擎并不支持事务, **比如 MySQL 中的 MyISAM 引擎. <font style="color:blue">在这种条件下即使使用了 @Transactional 注解, 事务也不会生效</font>**

   - 解决方案: 使用支持事务的存储引擎或更换支持事务的数据库系统

9. 事务管理器未启用: 在使用 Spring 时, 若未在配置类中添加 `@EnableTransactionManagement`, 或未正确配置 `PlatformTransactionManager`, @Transactional 注解将不会生效

   - 解决方案: 检查 Spring 配置是否启用了事务注解驱动, 并确保已正确绑定数据源与事务管理器
   - **<font style="color:blue">但在 Spring Boot 中, 上述配置由自动装配完成, 通常无需手动指定</font>**

# Bean的生命周期

为了方便理解, 这里会由浅入深地演示多个版本

- 先演示最简易的五步骤版本:

1. 实例化  2. 依赖注入  3. 初始化  4. 使用Bean  5. 销毁Bean

```java
public class Dog{
    private String name;
    
    public Dog{
        System.out.println("1. 实例化");
    }
    
    public String getName(){return name;}
    public void setName(){
        System.out.println("2. 依赖注入");
        this.name = name;
    }
    
    public void myInit(){ System.out.println("3. 初始化"); }
    public void myDestory(){ System.out.println("5. 销毁Bean"); }
}
```

```xml
<!-- spring.xml中配置Bean -->
<bean id="dog" class="com.java.bean.Dog" init-method="myInit" destory-method="myDestory">
    <property name="name" value="张三"></property>
</bean>
```

```java
// 测试代码
ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
context.getBean(Dog.class);
System.out.println("4. 使用Bean");
((ClassPathXmlApplicationContext) context).close();
```

- 然后是七步骤版本

```markdown
1. 实例化  
2. 依赖注入  
	3. 初始化前 BeanPostProcessor -- postProcessBeforeInitialization方法
4. 初始化  
	5. 初始化后 BeanPostProcessor -- postProcessAfterInitialization方法
6. 使用Bean  
7. 销毁Bean

- Spring AOP的代理对象就是在 `postProcessAfterInitialization()` 阶段生成的
> 这个阶段不会执行具体的"切面逻辑", 它只是判断是否需要增强 (这个Bean是否匹配任何一个已注册的切面逻辑, 即是否命中了某个切面), **并生成一个代理对象**

```

手动重写一下 BeanPostProcessor 方便观察执行流程

```java
public class MyProcessor implements BeanPostProcessor{
    /**
    	第一个参数: 正在被创建的 Bean 实例, Spring已经通过构造方法创建好了对象, 并完成了属性注入, 这个参数就是那个真实的 Bean 对象. 可以在这个阶段: 修改它的内部属性; 判断它是否实现了某些接口做条件处理等
    	
    	第二个参数: 这个Bean在Spring容器中的名字(注册时的ID), 可能来自xml中配置的id / 注解注册的类名首字母小写(注解方式默认) / 显式命名: @Component("customName")
    */
    @Override
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException{
        if("dog".equals(s)){
            System.out.println("3. 初始化前 BeanPostProcessor - before方法执行");
        }
        return o; // 返回原始对象
    }
    
    @Override
    public Object postProcessAfterInitialization(Object o, String s) throws BeansException{
        if("dog".equals(s)){
            System.out.println("5. 初始化后 BeanPostProcessor - after方法执行");
        }
        return o;
    }	
}
```

之后把自己写的 BeanPostProcessor 交给 Spring 容器管理

```xml
<bean id="processor" class="com.java.bean.MyProcessor"></bean>
```

---

- 最后是最详细的十步骤版本

```markdown
1. 实例化
	- Spring调用构造方法或工厂方法创建 Bean 实例(此时对象已存在, 但还没有注入属性)
	
2. 依赖注入 (属性注入)
	- 对成员变量或setter方法进行依赖注入
	
3. Aware接口的回调 (顺序是从上到下依次执行) -- **Aware是感知接口, 用于感知容器的信息, 把容器里的信息拿出来做自己想做的事**
	- 如果Bean实现了`BeanNameAware`接⼝, 会回调该接⼝的`setBeanName()`⽅法. 传⼊的参数是该Bean的id, 此时就知道这个Bean在容器中的名字了. (方便做内部记录日志或条件判断等)
	- 如果Bean实现了`BeanClassLoaderAware`接口, 会回调该接口的`setBeanClassLoader`方法. Spring会把用于加载当前Bean的类加载器传进来
	- 如果Bean实现了`BeanFactoryAware`接⼝, 会回调该接⼝的`setBeanFactory()`⽅法. 传⼊该Bean的BeanFactory (允许在当前Bean中动态获取其它Bean)
	- 如果Bean实现了`ApplicationContextAware`接⼝,会回调该接⼝的`setApplicationContext()`⽅法. Spring会注入整个ApplicationContext (提供了访问容器全局的能力)
	
4. BeanPostProcessor的前置处理 - `postProcessBeforeInitialization`方法
	- 在这可以对即将完成初始化的Bean做一些加工 (如修改它的某个属性值 / 给它加上一些额外功能等)
	
5. 初始化方法执行 (顺序是从上到下依次执行, Spring会按顺序调用这些初始化钩子)
	- (如果有) 执行`@PostConstruct`注解标注的方法 
	- 如果Bean实现了`InitializingBean`接⼝，则会回调该接⼝的`afterPropertiesSet()`⽅法
	- 执行XML或注解中指定的 `init-method` (如`@Bean(initMethod = "xxx")`)
	
6. BeanPostProcessor的后置处理 - `postProcessAfterInitialization`方法
	- AOP 的代理对象创建就是在这里发生的
	
7.  Bean初始化完成, 使用阶段

8. 容器关闭前回调`@PreDestory`修饰的方法
	- 即如果Bean中定义了`@PreDestory`注解的方法, 会在销毁前自动执行
	
9. 如果实现了`DisposableBean`接口, 则执行`destroy()`方法

10. 如果配置了`destroy-method`, 则调用对应的方法, 然后Bean从容器中被正式移除
	- 至于内存是否立即释放, 则由JVM的垃圾回收机制决定, 不属于Spring的控制范围
```

# AOP相关问题

## 介绍一下AOP

AOP 全称是 "Aspect Oriented Programming", 翻译过来就是面向切面编程, **是 Spring 框架的两大特性之一. AOP 其实就是 OOP (也就是面向对象编程) 的有益补充**, 与 OOP 相辅相成



需要 AOP 的原因其实就是 OOP 的设计有局限性, **<font style="color:blue">OOP 因为其继承、多态的特性可以很好地提高代码的复用性</font>, 比较擅长处理功能的纵向扩展**. 每个类各司其职, 新增功能可以通过继承或组合的方式实现, 对原有代码改动最小

**但它并不适合定义横向的关系**, 比如日志功能. 日志代码往往横向地散布在所有对象层次中, 而与它对应的对象核心功能毫无关系. 这种散布在各处的无关代码被称为横切, **在 OOP 设计中, 它导致了大量代码的重复,** 不利于各个模块的重用



**而 AOP 技术恰恰相反, 擅长处理功能的水平扩展, 在多个模块之间统一增加某些"通用功能"**, 与业务逻辑本身无关, 但对所有模块都适用. **<font style="color:blue">比如加日志记录、事务控制、权限校验等功能</font>**

AOP会把这些会影响多个模块的功能从业务逻辑中剥离出来, 抽取成切面来统一维护

## 怎么使用AOP

首先启用 AOP 功能, **<font style="color:blue">然后使用 @Aspect 注解标注某个类是一个切面类</font>, 同时配合 @Component 等注解将切面类同时注册为 Bean**, 使其能被 Spring 容器扫描到



**之后在 @Pointcut 注解中定义切入点表达式**, 有两种主要使用的表达式形式

- **第一种是使用 execution**, 在其括号内部要按顺序分别指定 **<font style="color:blue">`访问修饰符 方法返回值 方法所在类的全类名.方法名 (参数列表)`</font>**
  - 其中在访问修饰符和方法返回值的部分可以用一个 * 替代, 表示不做限制. **<font style="color:blue">但是如果要明确指定其中的某一个, 就也必须同时写明另一个</font>** (除非将访问修饰符省略)
  - 在指定全类名的部分中可以用 * 表示包名和类名任意, **用 `..` 可以表示包的层次深度任意**
  - **此外类名和方法名支持使用 * 进行模糊匹配**, 比如 `get*` 就表示匹配以 get 开头的方法
  - **<font style="color:blue">参数列表部分可以用 `(..)` 表示参数列表任意</font>**
- **第二种是使用 自定义注解 + @annotation 的方式定义切入点表达式**. 想用这种方式的话: 
  - 首先应该自定义一个注解, 用于标记哪些方法需要被 AOP 拦截, **并且在 @Retention 元注解中指定`RetentionPolicy.RUNTIME`, 保证注解在运行时仍然可以被识别**
  - 然后在 @Pointcut 注解中使用 @annotaion 指定切入点, **<font style="color:blue">@annotation 括号中指定的就是自定义注解的全类名</font>**
  - **最后在业务方法上使用自定义注解即可, 想拦截哪个方法就在哪贴注解**, 可以精细控制, 灵活性很强, 另外因为不依赖包名和方法名也降低了耦合度
  - 但是要注意使用了这种方式后 Spring 会在运行时检查所有 Bean 方法是否被切面中指定的注解标记, 匹配成功后这些方法就会被代理并织入增强逻辑
    - **<font style="color:blue">所以想被增强的前提是: 这个方法所在的类必须被注册为 Spring 的 Bean</font>**



最后一步就是在切面类中使用 @Around、@Before、@After 等注解定义具体的增强逻辑, **并在这些注解的参数中直接写明切入点表达式, 或者复用先前用 @Pointcut 注解定义好的切入点**

## AOP的通知类型都有哪些

Spring AOP 提供了五种通知类型, 分别是:

- 前置通知: 使用 @Before 注解标识, 在被代理的目标方法调用之前触发
- 后置通知: 使用 @After 注解标识, 在被代理的目标方法调用之后触发. **另外从 Spring 5.3 版本开始, 后置通知准确的说是在被代理的目标方法最终结束后执行**
  - **<font style="color:blue">也就是说后置通知的执行顺序在返回通知或异常通知之后</font>**, 而在 Spring 5.3 版本之前顺序相反
- 返回通知: 使用 @AfterReturning 注解标识, 在目标方法调用完成, 成功结束后执行
- 与返回通知相对应的是异常通知: 使用 @AfterThrowing 注解标识, 在目标方法运行时抛出异常后触发. 这里能看出来**<font style="color:blue">返回通知和异常通知两者互斥</font>, 因为如果方法抛出了异常就不会有返回值**
- 最后是环绕通知: 使用 @Around 注解标识, **环绕通知是所有通知类型中可操作范围最大的一种**, 因为它可以直接拿到目标对象以及要执行的方法
  - **<font style="color:blue">环绕通知中通过 joinPoint.proceed() 来触发目标方法调用</font>, 可以在其前后都插入自定义逻辑**. 因此它能够完全控制目标方法的执行流程, **甚至可以不调用目标对象的方法**

## AOP中各种通知的使用场景

参考博客: https://www.jb51.net/program/330902a9y.htm

前置通知因为在目标方法执行前执行, 所以比较适用于做**权限校验、参数验证、缓存预检** (也就是缓存命中就提前中断流程) 以及 进行一些初始化操作



**<font style="color:blue">返回通知适用于数据脱敏、统计方法成功率等场景</font>**. 数据脱敏就是在展示或传输数据时, 对敏感信息进行模糊化、替换等处理, 以防止敏感数据泄露

```java
@Aspect
@Component
public class DesensitizationAspect {

 @AfterReturning(pointcut = "execution(* com.example.service.UserService.getUser(..))",
                 returning = "user")
 public void desensitizeUser(User user) {
     if (user != null) {
         // 脱敏处理: 手机号、身份证号
         user.setPhone(maskPhone(user.getPhone()));
         user.setIdCard(maskIdCard(user.getIdCard()));
     }
 }

 private String maskPhone(String phone) {
     return phone.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2");
 }

 private String maskIdCard(String idCard) {
     return idCard.replaceAll("(\\d{4})\\d{10}(\\d{4})", "$1**********$2");
 }
}

// ---------- 演示效果
String phone = "13812345678";
//  通过分组 $1 $2, 保留了前3位和后4位, 中间用星号代替, 实现常见脱敏样式
String masked = phone.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2");
System.out.println(masked); // 输出：138****5678
```



与返回通知互斥的异常通知可以进行**异常记录、告警通知、失败重试**等操作

后置通知无论是否抛出异常都会在目标方法调用后执行, 因此比较适合进行**资源清理、连接关闭**等操作



环绕通知作为最强大的通知类型, 可以完全控制目标方法的执行, 因此也最为全面. **适用于事务管理、性能监控** 以及 方法的限流和缓存等场景

## AOP的实现原理

> 这里对于实现原理的说明也只是简述, 想了解详细过程可以参阅网上对于 AOP 源码的解读

1. 首先在 Spring 容器启动阶段, 如果启用了 AOP (例如加了 @EnableAspectJAutoProxy 注解, 或者在 SpringBoot 项目中添加了 AOP 相关依赖)
   - **<font style="color:blue">Spring会自动向容器中注册一个额外的后置处理器</font>, 作用是在Bean初始化完成后, 判断是否需要为该Bean创建代理对象**. 这个后置处理器叫 `AnnotationAwareAspectJAutoProxyCreator`



2. **在容器初始化过程中, Spring 会扫描并解析所有被 @Aspect 注解标记的类**. 对于这些切面类, Spring 会提取其中定义的切入点和通知方法
   - **<font style="color:blue">解析后的切面信息会被封装成 Advisor 对象</font>**, 用于后续判断哪些 Bean 需要增强



3. **当 Bean 初始化完成后, 根据 Bean 的生命周期来说也就是进入了 Bean 的后置处理阶段**, 这个阶段会执行容器中所有注册的后置处理器的处理方法
   - **<font style="color:blue">所以在这一步中容器启动阶段注册的 AOP 专用后置处理器也会被调用</font>**

   - 此时 Bean 对象虽然已经初始化完毕, **但还没有暴露给用户使用**, 所以 AOP 在此时切入最为合适



4. 随后在 AOP 后置处理器的处理逻辑中, **<font style="color:blue">Spring 会用目标类中的方法去匹配先前解析出来的 Advisor</font>**
   - 如果目标类中的方法不匹配任何切入点, 就直接返回原始对象; **如果匹配上了, 说明这个类需要 AOP 增强, 也就需要创建代理对象**



5. **除非配置了强制使用 CGLIB 代理**, 否则如果这个 Bean 实现了接口, 就使用 JDK 动态代理; 如果没有接口, 就使用 CGLIB 动态代理

   - **<font style="color:blue">代理对象内部会维护一个拦截器链</font>**. 生成代理对象时, Spring 会调用 `AdvisorAdapterRegistry` **把每个匹配的 Advisor 封装成一个拦截器**, 并加入到链中
   - **<font style="color:blue">这里的拦截器通常是 MethodInterceptor 类型</font>**, 这就是后续执行通知逻辑的关键. **之后 Spring 会将这个代理对象替代原始的 Bean 注册到容器中**

   > 原始对象在代理对象内部持有, 存储在 AdvisedSupport 类的 targetSource 属性中
   >
   > 通过 `targetSource.getTarget()` 方法就能拿到真正的原始对象

   

6. 之后当调用这个代理对象的方法时, **Spring 会根据多个切面的优先级以及通知类型**, 自动按顺序执行对应的增强逻辑, 并在合适的时机调用目标方法



























