## springboot中使用redis进行数据缓存

[TOC]

1. application.yml中设置redis
2. 在启动类中使用@EnableCaching注解
3. 缓存测试
4. 在需要缓存的类文件中使用@CacheConfig(cacheNames = "users")，代表即将在redis中创建users文件夹，并将本类中执行的缓存结果放在该文件夹下
5. 在方法上使用@Cacheable(key ="#p0")进行缓存，缓存的key为传入的第一个参数即'#p0'
6. @Cacheable(key ="#p0")，@CachePut(key = "#p0")，@CacheEvict(key ="#p0",allEntries=true)的区别

### 1. application.yml配置文件

```yaml
spring:
  # redis配置
  redis:
    # Redis数据库索引（默认为0）
    database: 1
    # Redis服务器地址
    host: 127.0.0.1
    # Redis服务器连接端口
    port: 6379
    # Redis服务器连接密码（默认为空）
    # password:
    # 连接超时时间（毫秒）
    timeout: 1000ms
    lettuce:
      pool:
        # 连接池最大连接数
        max-active: 200
        # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-wait: -1ms
        # 连接池中的最大空闲连接
        max-idle: 10
        # 连接池中的最小空闲连接
        min-idle: 1
```

### 2. 启动类使用@EnableCaching

```java
@EnableCaching
@SpringBootApplication
public class project_jwt_authApplication {
    public static void main(String[] args) {
        SpringApplication.run(project_jwt_authApplication.class,args);
    }
}
```

### 3. 缓存测试

1. redisTemplate.opsForValue();//操作字符串
2. redisTemplate.opsForHash();//操作hash
3. redisTemplate.opsForList();//操作list
4. redisTemplate.opsForSet();//操作set
5. redisTemplate.opsForZSet();//操作有序set

```java
@SpringBootTest
public class redistest {
    @Autowired
    StringRedisTemplate stringRedisTemplate;//操作k-v都是字符串

    @Autowired
    RedisTemplate redisTemplate;//操作k-v对象的
    @Autowired
    club.codeqi.bean.user.userMapper userMapper;
    @Test//使用stringTemplate缓存string类型的数据
    public void redistemplent(){
        stringRedisTemplate.opsForList().leftPush("test1","value1","value2");
        stringRedisTemplate.opsForList().rightPushAll("test3","value1","value2","value3");
    }

    @Test//使用redisTemplate缓存对象类型的数据
    public void redisobjtest(){
       	stringRedisTemplate.opsForHash().put("user::usertoken","1","qwertyyuiisdfgsdfg");
		redisTemplate.opsForValue().set();
       	redisTemplate.opsForHash().put("user:usertoken","1",userMapper.findByName("test02"));
		redisTemplate.expire("user:usertoken",100, TimeUnit.SECONDS);
    }

    @Test//获取缓存并将其转换为user类型的对象
    public void redisobjgettest(){
        Object o = redisTemplate.opsForHash().get("user::usertoken","1");
        System.out.println((user) o);

    }
    @Test//设置缓存过期时间
    public void redisexpretest(){
        redisTemplate.expire("user::usertoken",100, TimeUnit.SECONDS);
    }

    @Test//删除缓存
    public void redisdeletetest(){
        redisTemplate.delete("user:usertoken");
    }
}
```



### 4. Mapper接口使用@CacheConfig(cacheNames = "users")

```java
@Mapper
@CacheConfig(cacheNames = "users")
public interface userMapper extends BaseMapper<user> {
    public int insert(user user);
    @CachePut(key = "#p0")
    public int update(user user);
    public ArrayList<user> selectAll();
    @Cacheable(key ="#p0")
    public user selectByid(Integer uid);
    @CacheEvict(key ="#p0",allEntries=true)
    public int delete(int uid);

    public ArrayList<user> select_roleAll();
    @Cacheable(key ="#p0")
    public user findByName(String name);
}
```

### 5.使用@Cacheable(key ="#p0")

```java
@Cacheable(key ="#p0")
public user findByName(String name);
```

也可以使用@Cacheable(value="myuser",key ="#p0")

此注解将在redis中创建myuser文件夹，并在该文件夹下存储缓存

### 6.@Cacheable(key ="#p0")，@CachePut(key = "#p0")，@CacheEvict(key ="#p0",allEntries=true)的区别

@Cacheable：将方法执行的结果进行缓存，用于select

@CachePut：无论缓存中是否有数据，都调用方法，并将返回结果进行缓存，用于update

@CacheEvict：清除该缓存，用于delete

## 自定义缓存过期时间

如果不自己设置，系统默认的缓存过期时间为-1即永久保存

1. 设置application.yml
2. 自定义类继承CachingConfigurerSupport并使用@Configuration
3. 设置CacheManager

### 1. 设置application.yml

```yaml
spring:
  # redis配置
  redis:
    # Redis数据库索引（默认为0）
    database: 1
    # Redis服务器地址
    host: 127.0.0.1
    # Redis服务器连接端口
    port: 6379
    # Redis服务器连接密码（默认为空）
    # password:
    # 连接超时时间（毫秒）
    timeout: 1000ms
    expire: 200 #自定义缓存过期时间
    lettuce:
      pool:
        # 连接池最大连接数
        max-active: 200
        # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-wait: -1ms
        # 连接池中的最大空闲连接
        max-idle: 10
        # 连接池中的最小空闲连接
        min-idle: 1

```

### 2. 自定义类并配置CacheManager

```java
@Configuration
public class redisCache extends CachingConfigurerSupport {
    @Value("${spring.redis.expire}")//自定义的缓存过期时间
    private int timeout;

    //缓存管理器
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
        config = config.entryTtl(Duration.ofSeconds(timeout))     // 设置缓存的默认过期时间，也是使用Duration设置
                .disableCachingNullValues();     // 不缓存空值
        RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(factory);
        RedisCacheManager cacheManager = new RedisCacheManager(redisCacheWriter,config,true);
        return cacheManager;
    }
}
```



### 3. 自定义各缓存空间的配置

```java
@Configuration
public class redisCache extends CachingConfigurerSupport {
    @Value("${spring.redis.expire}")//自定义的缓存过期时间
    private int timeout;

    //缓存管理器
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
        config = config.entryTtl(Duration.ofSeconds(timeout))     // 设置缓存的默认过期时间，也是使用Duration设置
                .disableCachingNullValues();     // 不缓存空值
        RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(factory);
        //自定义缓存空间及相关配置
        Set<String> cacheNames =  new HashSet<>();
        cacheNames.add("my-redis-cache1");
        cacheNames.add("my-redis-cache2");

        // 对每个缓存空间应用不同的配置
        Map<String, RedisCacheConfiguration> configMap = new HashMap<>();
        configMap.put("my-redis-cache1", config);
        configMap.put("my-redis-cache2", config.entryTtl(Duration.ofSeconds(120)));
        RedisCacheManager cacheManager1 = RedisCacheManager.builder(factory)     // 使用自定义的缓存配置初始化一个cacheManager
                .initialCacheNames(cacheNames)  // 注意这两句的调用顺序，一定要先调用该方法设置初始化的缓存名，再初始化相关的配置
                .withInitialCacheConfigurations(configMap)
                .build();
        return cacheManager1;
    }
}
```



