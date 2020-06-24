# redisLearnning
Redis学习


在sping boot添加redis依赖以后，可以使用redis的template进行各种内存操作，包括缓存、热值、排序和列表等
redis下有两个Template,分别是：
* RedisTemplate
* StringRedisTemplate
StringRedisTemplate是RedisTemplate的进一步改装，看源码：
public class StringRedisTemplate extends RedisTemplate<String, String> {

	public StringRedisTemplate() {
		setKeySerializer(RedisSerializer.string());
		setValueSerializer(RedisSerializer.string());
		setHashKeySerializer(RedisSerializer.string());
		setHashValueSerializer(RedisSerializer.string());
	}
....
}
从源码可以看出StringRedis继承自Redis,并且参数都是String类型。
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
		redis.opsForValue().set("login","user1");;
		 System.out.println(redis.opsForValue().get("login"));
		assertTrue("user1".equals(redis.opsForValue().get("login")));
	}
  ```
  例3：RedisTemplate掉坑历险记
  ```

private RedisTemplate redis;

@Test
	public void redisTest() {
		redis.opsForValue().set("login","user1");;
		 System.out.println(redis.opsForValue().get("login"));
		assertTrue("user1".equals(redis.opsForValue().get("login")));
	}
  ```
  最后console报错：Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'org.springframework.data.redis.core.RedisTemplate<?, ?>' available: expected single matching bean but found 2: redisTemplate,stringRedisTemplate
总体来说，StringRedisTemplate是实际使用中用的最多的Template,因为String类型对于编程更加友好，而RedisTemplate则属于更加通用，可以支持不同的数据类型，操作起来也更加麻烦和容易出bug;


