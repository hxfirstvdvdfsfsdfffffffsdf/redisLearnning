# redisLearnning
## Redis学习

### 1.Template
在sping boot添加redis依赖以后，可以使用redis的template进行各种内存操作，包括缓存、热值、排序、列表和分布式锁等，很有名的焦点问题有缓存穿透、缓存击穿和缓存雪崩等；
redis下有两个Template,分别是：
* RedisTemplate
* StringRedisTemplate

StringRedisTemplate是RedisTemplate的进一步改装，RedisTemplate使用JDK序列化方式，但是StringRedisTemplate修改了序列化策略，看源码：
```
public class StringRedisTemplate extends RedisTemplate<String, String> {

	public StringRedisTemplate() {
		setKeySerializer(RedisSerializer.string());
		setValueSerializer(RedisSerializer.string());
		setHashKeySerializer(RedisSerializer.string());
		setHashValueSerializer(RedisSerializer.string());
	}
....
}
```

从源码可以看出StringRedisTemplate继承自RedisTemplate,并且参数都是String类型。
例1：StringRedisTemplate的基本使用
```
@Autowired
private StringRedisTemplate stringRedisTemplate;

@Test
	public void StringRedisTest() {
		stringRedisTemplate.opsForValue().set("login","user1");
		 System.out.println(stringRedisTemplate.opsForValue().get("login"));
		assertTrue("user1".equals(stringRedisTemplate.opsForValue().get("login")));
	}
  
  ```
  例2：RedisTemplate的正确基本使用
  ```
 @Autowired//必须声明类型
private RedisTemplate<String,String> redis;

@Test
	public void redisTest() {
		redis.opsForValue().set("login","user1");
		 System.out.println(redis.opsForValue().get("login"));
		assertTrue("user1".equals(redis.opsForValue().get("login")));
	}
  ```
  例3：RedisTemplate爬坑历险记
  ```

private RedisTemplate redis;

@Test
	public void redisTest() {
		redis.opsForValue().set("login","user1");
		 System.out.println(redis.opsForValue().get("login"));
		assertTrue("user1".equals(redis.opsForValue().get("login")));
	}
  ```
  最后console报错：
  >Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'org.springframework.data.redis.core.RedisTemplate<?, ?>' available: expected single matching bean but found 2: redisTemplate,stringRedisTemplate
  
  原因：使用Template的时候StringRedisTemplate的参数就是<String,String>,在底层实现中已经写死，在Autowaird时才不用声明，但是RedisTemplate的参数具有多种数据类型，所有Autowaird的时候要精确的声明，如果参数全是String则声明为<String,String>,如果有Integer则声明为<String,Integer>类型，切记！
总体来说，StringRedisTemplate是实际使用中用的最多的Template,因为String类型对于编程更加友好，而RedisTemplate则属于更加通用，可以支持不同的数据类型，操作起来也更加麻烦和容易出bug;
### 2.opsFor
代码 redis.opsForValue().set("login","user1")中opsForValue返回的是一个ValueOperations对象，除了这个方法外，opsFor还包括：
* opsForHash(),返回HaspOperations
* opsForList(),返回ListOperations
* opsForSet(),返回SetOperations
* opsForZSet(),返回ZSetOperations
* opsForGeo(),返回GeoOperations

 ```
stringRedisTemplate.opsForSet().add("set1","1");
		stringRedisTemplate.opsForSet().add("set2", "2");
	System.out.println(stringRedisTemplate.opsForSet().members("set2"));
 ```
### 3.分布式锁
目前分布式锁的主流方案包括一下4种：
* 数据库(很少使用，性能是最大障碍)
* Redis
* Zookeeper
* Chubby(妈妈是谷歌，国内用的比较少，和Zookeeper归为一类)

实现原理基本相同：
* 操作的原子性
* 资源的唯一性（边界资源）

实现的3种方式
1.数据库方式：乐/悲观锁+唯一约束
2.Redis方式： setnx命令
3.Zookeeper方式：临时顺序节点

Redis实现分布式锁用的是SETNX ———— Set if not exists，在指定的key不存在时，为key设置指定的值。并且当设置成功时返回1，失败返回0，因此我们可以通过返回值来判断加锁是否成功。
 ```
 一个Redis锁的一般实现，在多数情况下，这个锁是安全的，但是在超高并发量的情况下，还是可能出现问题
 @Component
 public class RedisLock{
 @Autowaird
 private StringRedisTemplate redis;
 public boolean lock(String key,String value){
 return redis.opsForValue().setIfAbsent(key,value);//setnx命令封装成了setIfAbsent()方法，返回值封装成了boolean
 }
 public void unlock(String key,String vlue){
 String oldValue=redis.opsForValue().get(key);
 if(object.nonNull(oldValue)&&oldValue.equals(value));
 redis.delete(key);
 }
 }
 
  ```
  Redis分布式锁还有一种简单的实现方法，并且这种方法是原子性的，绝对线程安全可靠————Lua脚本，Lua脚本一般是用c编写，具有轻量快捷的特点，通过redisTemplate的execute()方法来调用Lua脚本；
  Redission是一个redis官方推荐的分布式解决方案，更加推荐使用Redission来作为分布式锁的解决方案。
