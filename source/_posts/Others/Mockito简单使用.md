---
title: Mockito简单使用
excerpt: Mockito简介，使用Mockito在Spring Boot项目中进行单元测试
tags:
  - 单元测试
categories:
  - Others
banner_img: /img/banner/catiger.png
index_img: /img/index/code.jpg
category: Others
abbrlink: '21040449'
date: 2021-09-13 14:12:01
updated: 2021-09-25 18:23:01
subtitle:
---

## 1. 简介
1. Mockito框架  
  * Mockito是一个强大的用于 Java 开发的 Mock 测试框架, 通过 Mockito 我们可以创建和配置 Mock 对象, 进而简化有外部依赖的类的测试
  * Mockito让我们得以避开数据库等的配置，保证我们的单测不依赖于外部条件。
2. 常用注解
    * `@Mock` 创建 mock 对象
    * `@InjectMocks` 创建 mock 对象，属性含有被 @Mock 或 @Spy 标注的对象时会自动注入
    * `@Spy` 或 spy() 方法：封装 Java 对象，封装后对象的方法进行打桩，实际调用时会执行打桩方法
    ```java
    @Test
    public void SpyTest(){
        
        List<Integer> list = new ArrayList<>();
        List<Integer> spy = spy(list);
        // 打桩
        doReturn("spy test").when(spy).get(anyInt());

        System.out.println(spy.get(1));
    }
    ```   
## 2. 使用

### 2.1 创建 mock 对象：Mockito 提供了两种方式创建 mock 对象
1. 使用静态方法 
   * `Mockito.mock(clazz)` 直接创建对象

    ```java
    jedis = mock(Jedis.class);
    jedisConnectionFactory = mock(JedisConnectionFactory.class);
    redisLock = new RedisLock(jedisConnectionFactory);
    ```
2. 使用上面提到的两个注解
   * 使用`@Mock`, `@InjectMocks` 标注需要创建的对象
   * 调用 `MockitoAnnotations.openMocks(this)` 方法创建 mock 对象
   
   ```java
    @Mock
    private JedisConnectionFactory jedisConnectionFactory;

    @Mock
    private Jedis jedis;

    @InjectMocks
    private RedisLock redisLock;

    @BeforeEach
    public void setBefore() {

        MockitoAnnotations.openMocks(this);

        RedisConnection redisConnection = new JedisConnection(jedis) ;
        Mockito.when(jedisConnectionFactory.getConnection()).thenReturn(redisConnection);

        jedis = (Jedis)redisConnection.getNativeConnection();
    }
   ```

### 2.2 Junit + Mockito

1. 在测试类上使用注解：
   * Junit4: `@RunWith(MockitoJUnitRunner.class)`
   * Junit5: `@ExtendWith(MockitoExtension.class)`

   该注解作用同 `MockitoAnnotations.openMocks(this)` 方法, Mockito 文档：
   > Until now in JUnit there were two ways to initialize fields annotated by Mockito annotations such as @Mock, @Spy, @InjectMocks, etc.
   > * Annotating the JUnit test class with a @RunWith(MockitoJUnitRunner.class)
   > * Invoking MockitoAnnotations.openMocks(Object) in the @Before method
2. 创建 mock 对象
    * `@Mock`
    * `@InjectMocks`
    
### 2.3 Spring Boot + Mockito

1. @MockBean 将被标注对象放入 spring 容器中
2. @SpyBean 封装对象

## 3. 应用

### 3.1 案例项目
1. 说明：
   * 基于 Redis 的分布式锁
   * 代码是从 crossoverJie 大佬的GitHub项目 [distributed-redis-tool](https://github.com/crossoverJie/distributed-redis-tool.git) 中抽取的

2. 配置类

    ```java
   @Configuration
   public class RedisConfig {
    
       @Bean
       JedisConnectionFactory jedisConnectionFactory() {
           return new JedisConnectionFactory();
       }
   }
    ```

3. RedisLock 类

   ```java
   @Component
   public class RedisLock {
   
       private static Logger log = LoggerFactory.getLogger(RedisLock.class);
   
       private static final String SET_IF_NOT_EXIST = "NX";
   
       private static final String SET_WITH_EXPIRE_TIME = "EX";
   
       public static final String LOCK_MSG = "OK";
   
       private static final Long UNLOCK_MSG = 1L;
   
       private static final String LOCK_PREFIX = "lock_";
   
       // 过期时间(s)
       private static final int TIME = 10;
   
       // lua 脚本
       private static final String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
   
       private final JedisConnectionFactory jedisConnectionFactory;
   
       public RedisLock(JedisConnectionFactory jedisConnectionFactory) {
           this.jedisConnectionFactory = jedisConnectionFactory;
       }
   
       private Object getConnection(){
   
           RedisConnection redisConnection = jedisConnectionFactory.getConnection();
           Object connection = redisConnection.getNativeConnection();
   
           return connection;
       }
   
       /**
        * 分布式锁，加锁
        * @param key 锁唯一标识
        * @param request 线程唯一标识
        * @return
        */
       public boolean tryLock(String key, String request) {
   
           SetParams params = new SetParams().ex(TIME).nx();
   
           Jedis connection = (Jedis) this.jedisConnectionFactory.getConnection().getNativeConnection();
           String result = connection.set(LOCK_PREFIX + key, request, params);
   
           return LOCK_MSG.equals(result);
       }
   
       /**
        * 分布式锁，解锁
        * @param key 锁唯一标识
        * @param request 线程唯一标识
        * @return
        */
       public boolean unlock(String key, String request){
   
           Jedis connection = (Jedis) getConnection();
   
           Object result = connection.eval(script, Collections.singletonList(LOCK_PREFIX + key), Collections.singletonList(request));
           connection.close();
   
           return UNLOCK_MSG.equals(result);
   
       }
   }
   ```
   
### 3.2 单元测试

1. `Mockito.mock(clazz)`

    ```java
   public class MockitoMockTest {
   
       private Jedis jedis;
   
       private RedisLock redisLock;
   
       private JedisConnectionFactory jedisConnectionFactory;
   
       @BeforeEach
       public void setBefore() {
   
           jedis = mock(Jedis.class);
           jedisConnectionFactory = mock(JedisConnectionFactory.class);
           redisLock = new RedisLock(jedisConnectionFactory);
   
           RedisConnection redisConnection = new JedisConnection(jedis) ;
           Mockito.when(this.jedisConnectionFactory.getConnection()).thenReturn(redisConnection);
   
           jedis = (Jedis)redisConnection.getNativeConnection();
       }
   
       @Test
       public void redisLockTest(){
   
           Mockito.when(jedis.eval(Mockito.anyString(), Mockito.anyList(), Mockito.anyList())).thenReturn(1L) ;
   
           boolean unlock = redisLock.unlock("test", "ec8ebca0-14ba0-4b23-99a8-b35fbba3629e");
   
           assertTrue(unlock);
   
           Mockito.verify(jedis).eval(Mockito.anyString(), Mockito.anyList(), Mockito.anyList());
       }
   
       @Test
       public void redisUnlockTest(){
   
           when(jedis.set(anyString(), anyString(), any(SetParams.class))).thenReturn("OK");
   
           boolean lock = redisLock.tryLock("key", UUID.randomUUID().toString());
   
           assertTrue(lock);
   
           verify(jedis).set(anyString(), anyString(), any(SetParams.class));
       }
   
   }
    ```

2. `MockitoAnnotations.openMocks(this)`

    ```java
   public class MockitoOpenMocksTest {
   
       @Mock
       private JedisConnectionFactory jedisConnectionFactory;
   
       @Mock
       private Jedis jedis;
   
       @InjectMocks
       private RedisLock redisLock;
   
       @BeforeEach
       public void setBefore() {
   
           MockitoAnnotations.openMocks(this);
   
           RedisConnection redisConnection = new JedisConnection(jedis) ;
           Mockito.when(jedisConnectionFactory.getConnection()).thenReturn(redisConnection);
   
           jedis = (Jedis)redisConnection.getNativeConnection();
       }
   
       @Test
       public void redisLockTest(){
   
           Mockito.when(jedis.eval(Mockito.anyString(), Mockito.anyList(), Mockito.anyList())).thenReturn(1L) ;
   
           boolean unlock = redisLock.unlock("test", "ec8ebca0-14ba0-4b23-99a8-b35fbba3629e");
   
           assertTrue(unlock);
   
           Mockito.verify(jedis).eval(Mockito.anyString(), Mockito.anyList(), Mockito.anyList());
       }
   
       @Test
       public void redisUnlockTest(){
   
           when(jedis.set(anyString(), anyString(), any(SetParams.class))).thenReturn("OK");
   
           boolean lock = redisLock.tryLock("key", UUID.randomUUID().toString());
   
           assertTrue(lock);
   
           verify(jedis).set(anyString(), anyString(), any(SetParams.class));
       }
   }
    ```
   
3. Junit + Mockito

    ```java
   @ExtendWith(MockitoExtension.class)
   public class JunitMockitoTest {
   
       @Mock
       private JedisConnectionFactory jedisConnectionFactory;
   
       @Mock
       private Jedis jedis;
   
       @InjectMocks
       private RedisLock redisLock;
   
       @BeforeEach
       public void setBefore() {
   
           RedisConnection redisConnection = new JedisConnection(jedis) ;
           Mockito.when(jedisConnectionFactory.getConnection()).thenReturn(redisConnection);
   
           jedis = (Jedis)redisConnection.getNativeConnection();
       }
   
       @Test
       public void redisLockTest(){
   
           Mockito.when(jedis.eval(Mockito.anyString(), Mockito.anyList(), Mockito.anyList())).thenReturn(1L) ;
   
           boolean unlock = redisLock.unlock("test", "ec8ebca0-14ba0-4b23-99a8-b35fbba3629e");
   
           assertTrue(unlock);
   
           Mockito.verify(jedis).eval(Mockito.anyString(), Mockito.anyList(), Mockito.anyList());
       }
   
       @Test
       public void redisUnlockTest(){
   
           when(jedis.set(anyString(), anyString(), any(SetParams.class))).thenReturn("OK");
   
           boolean lock = redisLock.tryLock("key", UUID.randomUUID().toString());
   
           assertTrue(lock);
   
           verify(jedis).set(anyString(), anyString(), any(SetParams.class));
       }
   }
    ```
   
4. Spring Boot + Mockito

    ```java
   @SpringBootTest
   public class SpringMockitoTest {
   
       @Mock
       private Jedis jedis;
   
       @Autowired
       private RedisLock redisLock;
   
       @MockBean
       private JedisConnectionFactory jedisConnectionFactory ;
       
       @BeforeEach
       public void setBefore() {
   
           MockitoAnnotations.openMocks(this);
   
           RedisConnection redisConnection = new JedisConnection(jedis) ;
           Mockito.when(jedisConnectionFactory.getConnection()).thenReturn(redisConnection);
   
           jedis = (Jedis)redisConnection.getNativeConnection();
       }
   
       @Test
       public void unlock() {
   
           Mockito.when(jedis.eval(Mockito.anyString(), Mockito.anyList(), Mockito.anyList())).thenReturn(1L) ;
   
           boolean unlock = redisLock.unlock("test", "ec8ebca0-14ba0-4b23-99a8-b35fbba3629e");
   
           assertTrue(unlock);
   
           Mockito.verify(jedis).eval(Mockito.anyString(), Mockito.anyList(), Mockito.anyList());
       }
   
       @Test
       public void lock() {
           Mockito.when(jedis.set(Mockito.anyString(), Mockito.anyString(), Mockito.any(SetParams.class))).thenReturn("OK");
   
           boolean tryLock = redisLock.tryLock("key", UUID.randomUUID().toString());
   
           assertEquals(true, tryLock);
   
           Mockito.verify(jedis).set(Mockito.anyString(), Mockito.anyString(), Mockito.any(SetParams.class));
   
       }
   }  
    ```
   
### 相关链接
1. [Mockito官网](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
2. [Spring Boot Mock](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.mocking-beans)
3. [基于 Redis 的分布式锁](https://crossoverjie.top/2018/03/29/distributed-lock/distributed-lock-redis/)