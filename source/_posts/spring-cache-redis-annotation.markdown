---
layout: post
title: "Spring Redis 注解式Cache那些事"
date: 2018-7-23 17:05
comments: true
categories: spring
tags: [ spring, redis, cache ]
---

### 前言：

>spring-data-redis使得Spring项目可以快速简单的通过RedisTemplate来操作Redis。而spring-boot-starter-data-redis更是让redis集成更加的方便。

### SpringBoot如何与Redis集成，作为cache

application.xml里如下配置：

 	spring:
  		redis:
    		host: 127.0.0.1
    		port: 6379
    		database: 0
    		timeout: 1000
    		pool:
      			max-idle: 200
      			min-idle: 0
      			max-active: 200
      			max-wait: 1000

spring boot可以自动组装相关配置，注意其中使用到了jedis pool，用于提升性能，非必须。
通过以下的annotation加入方法名上，可以无侵入的使用cache。

 - @Cacheable   缓存
 - @CachePut    设置缓存
 - @CacheEvict  失效或更新缓存
 - @Caching   组合操作

 以上annotation不做详细展开。

 <!--more-->

 做到上面似乎已经可以了，但有一些问题需要我们来解决。

- a.redis连接报错\超时怎么办？此时应该是可降级的。
- b.使用连接池，连接不可用如何破？

下面贴一个比较成熟的做法，继承`CachingConfigurerSupport`：

	@Configuration
	@EnableCaching //启用
	public class RedisConfig extends CachingConfigurerSupport {

	    // 过期时间
	    private static final long expire = 600;

	    // application.yml配置参数有限，注入并扩展用。
	    @Autowired
	    private RedisProperties redisProperties;

	    //此处自定义jedis pool配置，设置TestOnBrrow等等
	    @Bean
	    public JedisPoolConfig jedisPoolConfig() {
	        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
	        RedisProperties.Pool pool = redisProperties.getPool();
	        jedisPoolConfig.setMaxIdle(pool.getMaxIdle());
	        jedisPoolConfig.setMaxTotal(pool.getMaxActive());
	        jedisPoolConfig.setMinIdle(pool.getMinIdle());
	        jedisPoolConfig.setMaxWaitMillis(pool.getMaxWait());
	        jedisPoolConfig.setTestOnBorrow(true);
	        jedisPoolConfig.setTestWhileIdle(true);
	        return jedisPoolConfig;
	    }

	   //生成redisConnectionFactory，使用自定义的jedis pool
	    @Bean
	    public RedisConnectionFactory redisConnectionFactory(JedisPoolConfig jedisPoolConfig) {
	        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory();
	        jedisConnectionFactory.setHostName(redisProperties.getHost());
	        jedisConnectionFactory.setPort(redisProperties.getPort());
	        jedisConnectionFactory.setDatabase(redisProperties.getDatabase());
	        jedisConnectionFactory.setTimeout(redisProperties.getTimeout());
	        if (null != redisProperties.getPassword()) {
	            jedisConnectionFactory.setPassword(redisProperties.getPassword());
	        }
	        jedisConnectionFactory.setPoolConfig(jedisPoolConfig);
	        return jedisConnectionFactory;
	    }

	    // 设置cacheManager相关，主要涉及默认过期时间。
	    @Bean
	    public CacheManager cacheManager(RedisTemplate redisTemplate) {
	        RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
	        //设置缓存过期时间，可单独对某个cache制定过期时间
	        cacheManager.setDefaultExpiration(expire);
	        //设置redis key是否使用前缀，默认前缀是cacheName
	        cacheManager.setUsePrefix(true);
	        return cacheManager;
	    }

	    //定义redisTemplate，主要是定义key\value的序列化器
	    @Bean
	    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
	        StringRedisTemplate template = new StringRedisTemplate(redisConnectionFactory);
	        template.setValueSerializer(getValueSerializer());
	        template.afterPropertiesSet();
	        return template;
	    }

	    private RedisSerializer getValueSerializer() {
	        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
	        ObjectMapper om = new ObjectMapper();
	        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
	        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
	        jackson2JsonRedisSerializer.setObjectMapper(om);
	        return jackson2JsonRedisSerializer;
	    }

	    // 设置redis key生成策略
	    @Bean
	    @Override
	    public KeyGenerator keyGenerator() {
	        return new RequestKeyGenerator();
	    }

	    // 重点：设置和redis交互报错时的错误处理器。
	    @Bean
	    @Override
	    public CacheErrorHandler errorHandler() {
	        return new CallbackCacheErrorHandler();
	    }

下面看一下`CallbackCacheErrorHandler`	:

	public class CallbackCacheErrorHandler implements CacheErrorHandler {

	    private static final Logger LOGGER = LoggerFactory.getLogger(CallbackCacheErrorHandler.class);

	    @Override
	    public void handleCacheGetError(RuntimeException exception, Cache cache, Object key) {
	        LOGGER.error("cache get error, cacheName:{}, key:{}, msg:", cache.getName(), key, exception);
	    }

	    @Override
	    public void handleCachePutError(RuntimeException exception, Cache cache, Object key, Object value) {
	        LOGGER.error("cache put error, cacheName:{}, key:{}, msg:", cache.getName(), key, exception);

	    }

	    @Override
	    public void handleCacheEvictError(RuntimeException exception, Cache cache, Object key) {
	        LOGGER.error("cache evict error, cacheName:{}, key:{}, msg:", cache.getName(), key, exception);

	    }

	    @Override
	    public void handleCacheClearError(RuntimeException exception, Cache cache) {
	        LOGGER.error("cache clear error, cacheName:{}, msg:", cache.getName(), exception);
	    }
	}

此处当报错的时候只进行了日志记录，当然如果有其他需求，都可以在这里扩展。自此，spring boot与redis集成大功告成，一切都是那么的完美。

### 关于RedisCacheManager是否`setUsePrefix`的坑
首先，我们要知道是否使用`prefix`的区别是什么？
区别如下：

- 1. 使用`prefix`的时候，redis cache的key都会默认添加上cacheName，用于区分不同的cache。
- 2. 使用`prefix`的时候，当清除或者失效所有的key的时候，使用的是key prefix*获取所有的key,然后依次清楚。而不使用`prefix`的时候，需要清除或者失效所有key的时候，则是从一个维护了所有key的zset中获取的，这个zset通常叫做`${cacheName}~keys`。

下面通过源代码来证实一下：
RedisCache.java内`RedisWriteThroughCallback `负责往redis设置缓存：

		static class RedisWriteThroughCallback extends AbstractRedisCacheCallback<byte[]> {

		public RedisWriteThroughCallback(BinaryRedisCacheElement element, RedisCacheMetadata metadata) {
			super(element, metadata);
		}

		@Override
		public byte[] doInRedis(BinaryRedisCacheElement element, RedisConnection connection) throws DataAccessException {

			try {
              //加锁
				lock(connection);

				try {

					byte[] value = connection.get(element.getKeyBytes());

					if (value != null) {
						return value;
					}

					if (!isClusterConnection(connection)) {

						connection.watch(element.getKeyBytes());
						// 开始事务
						connection.multi();
					}

					value = element.get();

					if (value.length == 0) {
						connection.del(element.getKeyBytes());
					} else {
					   // 设置缓存key-value
						connection.set(element.getKeyBytes(), value);
						// 设置失效日期
						processKeyExpiration(element, connection);
						// 维护key到已知zset内
						maintainKnownKeys(element, connection);
					}

					if (!isClusterConnection(connection)) {
						connection.exec();
					}

					return value;
				} catch (RuntimeException e) {
					if (!isClusterConnection(connection)) {
						connection.discard();
					}
					throw e;
				}
			} finally {
			   // 释放锁
				unlock(connection);
			}
		}
	};

	 protected void maintainKnownKeys(RedisCacheElement element, RedisConnection connection) {

			if (!element.hasKeyPrefix()) { //不使用prefix
              // 则zadd到已知的key集合内
				connection.zAdd(cacheMetadata.getSetOfKnownKeysKey(), 0, element.getKeyBytes());

				if (!element.isEternal()) {
					connection.expire(cacheMetadata.getSetOfKnownKeysKey(), element.getTimeToLive());
				}
			}
		}

从上面分析得知，设置缓存的时候有以下几步：

- 1.设置key-value
- 2.设置key的过期时间
- 3.维护key到已知key的zset列表

清理所有key的时候，是怎么操作的呢？

	public void clear() {
		redisOperations.execute(cacheMetadata.usesKeyPrefix() ? new RedisCacheCleanByPrefixCallback(cacheMetadata)
				: new RedisCacheCleanByKeysCallback(cacheMetadata));
	}

可以看出依据是否使用前缀，使用不同的回调方法。

	/**
	 * @author Christoph Strobl
	 * @since 1.5
	 */
	static class RedisCacheCleanByKeysCallback extends LockingRedisCacheCallback<Void> {

		private static final int PAGE_SIZE = 128;
		private final RedisCacheMetadata metadata;

		RedisCacheCleanByKeysCallback(RedisCacheMetadata metadata) {
			super(metadata);
			this.metadata = metadata;
		}

		/*
		 * (non-Javadoc)
		 * @see org.springframework.data.redis.cache.RedisCache.LockingRedisCacheCallback#doInLock(org.springframework.data.redis.connection.RedisConnection)
		 */
		@Override
		public Void doInLock(RedisConnection connection) {

			int offset = 0;
			boolean finished = false;

			do {
				// need to paginate the keys
				Set<byte[]> keys = connection.zRange(metadata.getSetOfKnownKeysKey(), (offset) * PAGE_SIZE,
						(offset + 1) * PAGE_SIZE - 1);  //使用zrange遍历，删除
				finished = keys.size() < PAGE_SIZE;
				offset++;
				if (!keys.isEmpty()) {
					connection.del(keys.toArray(new byte[keys.size()][]));
				}
			} while (!finished);

			connection.del(metadata.getSetOfKnownKeysKey());
			return null;
		}
	}

	/**
	 * @author Christoph Strobl
	 * @since 1.5
	 */
	static class RedisCacheCleanByPrefixCallback extends LockingRedisCacheCallback<Void> {

		private static final byte[] REMOVE_KEYS_BY_PATTERN_LUA = new StringRedisSerializer().serialize(
				"local keys = redis.call('KEYS', ARGV[1]); local keysCount = table.getn(keys); if(keysCount > 0) then for _, key in ipairs(keys) do redis.call('del', key); end; end; return keysCount;");
		private static final byte[] WILD_CARD = new StringRedisSerializer().serialize("*");
		private final RedisCacheMetadata metadata;

		public RedisCacheCleanByPrefixCallback(RedisCacheMetadata metadata) {
			super(metadata);
			this.metadata = metadata;
		}

		/*
		 * (non-Javadoc)
		 * @see org.springframework.data.redis.cache.RedisCache.LockingRedisCacheCallback#doInLock(org.springframework.data.redis.connection.RedisConnection)
		 */
		@Override
		public Void doInLock(RedisConnection connection) throws DataAccessException {

			byte[] prefixToUse = Arrays.copyOf(metadata.getKeyPrefix(), metadata.getKeyPrefix().length + WILD_CARD.length);
			System.arraycopy(WILD_CARD, 0, prefixToUse, metadata.getKeyPrefix().length, WILD_CARD.length);

			if (isClusterConnection(connection)) {

				// load keys to the client because currently Redis Cluster connections do not allow eval of lua scripts.
				Set<byte[]> keys = connection.keys(prefixToUse);  //集群模式下，使用keys获取所有的key
				if (!keys.isEmpty()) {
					connection.del(keys.toArray(new byte[keys.size()][]));
				}
			} else {
			   // 非集群模式下，使用LUA脚本，keys删除。
				connection.eval(REMOVE_KEYS_BY_PATTERN_LUA, ReturnType.INTEGER, 0, prefixToUse);
			}

			return null;
		}
	}

从以上源码可以看出使用prefix的区别。总结下，坑在哪儿，应该如何根据业务来选择。

-  坑1：不使用`prefix`,需要额外的zset来保存已知key集合，风险点是zset有可能很大，占用空间，如果被置换出去，功能则不一致
-  	坑2：使用`prefix`, 没有额外的zset。但是失效或者清理所有key的时候，使用`keys * `可能导致redis被拖死，清理时间内无响应。
-  坑3：设置缓存，使用了multi，对redis压力不小，高并发下尤其明显，需要注意。

### 关于Redis Cache默认使用lock的问题
在高并发下，发现spring redis cache的put效率并不高，经过排查发现put操作有lock机制，切lock时间无法更改。

如上`RedisWriteThroughCallback`所示，有lock和unlock操作，其实就是往redis写一个key作为lock, 删除这个key作为unlock。这个操作在分布式系统中，可以保证其一致性，但是也损失了性能。尤其在仅作为缓存使用的场景，key对应的value具备幂等性，完全可以忽略。

源码重点在这个`waitForLock `方法里：

		protected boolean waitForLock(RedisConnection connection) {

			boolean retry;
			boolean foundLock = false;
			do {
				retry = false;
				if (connection.exists(cacheMetadata.getCacheLockKey())) {
					foundLock = true;
					try {
						Thread.sleep(WAIT_FOR_LOCK_TIMEOUT); //此处WAIT_FOR_LOCK_TIMEOUT=300ms
					} catch (InterruptedException ex) {
						Thread.currentThread().interrupt();
					}
					retry = true;
				}
			} while (retry);

			return foundLock;
		}

       // 加锁
		protected void lock(RedisConnection connection) {
			waitForLock(connection);
			connection.set(cacheMetadata.getCacheLockKey(), "locked".getBytes());
		}
      // 解锁
		protected void unlock(RedisConnection connection) {
			connection.del(cacheMetadata.getCacheLockKey());
		}

可以看出每次加锁，如果lock已经存在的情况下，会额外sleep 300ms,这在高并发、高性能的缓存场景是**极其低效**的。并且在极端情况下，unlock删除key没成功，将会导致所有key都无法设置或更新,并陷入死循环。spring内部也没有提供相关的行为覆盖机制，这是一个较大的坑。


### Spring-Data-Redis 2.0 RC1的优化

[官方DATAREDIS-481](https://jira.spring.io/browse/DATAREDIS-481)注意到了Lock的优化，并对cache manager做了颠覆性的升级。
下面跟着我来看看，spring-data-redis 2.0之后如何使用注解式cache.
由于底层依赖的[Jedis](https://github.com/xetorthio/jedis),自从发布2.9.0版本之后，升级缓慢，目前也仅支持到2.8.x和3.x.x版本，所以Spring推荐使用[lettuce](https://github.com/lettuce-io/lettuce-core).

先看application.xml里如何写：

	spring:
	    redis:
	      host: 127.0.0.1
	      database: 0
	      port: 6379
	      timeout: 1000
	      lettuce:
	        pool:
	          max-active: 500
	          min-idle: 0
	          max-idle: 500
	          max-wait: 1000
开始使用lettuce了，jedis提示deprecated了。
pool提供的参数有限，如果想自己定制，参见如下设置：

   	//继承CachingConfigurerSupport
	@Configuration
	@EnableCaching
	public class RedisConfig extends CachingConfigurerSupport {

	    //注入默认参数
	    @Autowired
	    private RedisProperties redisProperties;
	    //默认超时
	    private long expire = 600L;

	    @Bean
	    public RedisConnectionFactory redisConnectionFactory() {
	        //commons-pool2包
	        GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
	        poolConfig.setMaxIdle(500);
	        poolConfig.setMinIdle(0);
	        poolConfig.setMaxTotal(500);
	        poolConfig.setMaxWaitMillis(1000);
	        poolConfig.setTestOnBorrow(true);   //额外设置

	        // 基本连接信息：host port database password
	        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
	        redisStandaloneConfiguration.setHostName(redisProperties.getHost());
	        redisStandaloneConfiguration.setPort(redisProperties.getPort());
	        redisStandaloneConfiguration.setDatabase(redisProperties.getDatabase());
	        if (null != redisProperties.getPassword()){
	            redisStandaloneConfiguration.setPassword(RedisPassword.of(redisProperties.getPassword()));
	        }

	        //这里单独配置超时时间，连接池管理
	        LettuceClientConfiguration lettuceClientConfiguration = LettucePoolingClientConfiguration.builder()
	                .commandTimeout(Duration.ofMillis(200)).shutdownTimeout(Duration.ofMillis(200)).poolConfig
	                        (poolConfig)
	                .build();
	        LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory
	                (redisStandaloneConfiguration, lettuceClientConfiguration);
	        lettuceConnectionFactory.setValidateConnection(true);
	        return lettuceConnectionFactory;
	    }

	    @Bean
	    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
	        // 默认配置使用prefix、单独设置valueSerializer、过期时间
	        RedisCacheConfiguration redisCacheConfiguration =
	                RedisCacheConfiguration.defaultCacheConfig().serializeValuesWith(
	                        RedisSerializationContext.SerializationPair.fromSerializer(getValueSerializer()))
	                        .entryTtl(Duration.ofSeconds
	                                (expire)).disableCachingNullValues();
	        // 使用redisConnectionFactory直接创建无锁的cm
	        RedisCacheManager cm = RedisCacheManager.builder(RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory)).cacheDefaults(redisCacheConfiguration).transactionAware().build();
	        return cm;
	    }

	    private RedisSerializer getValueSerializer() {
	        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
	        ObjectMapper om = new ObjectMapper();
	        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
	        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
	        jackson2JsonRedisSerializer.setObjectMapper(om);
	        return jackson2JsonRedisSerializer;
	    }

	    @Bean
	    @Override
	    public CacheErrorHandler errorHandler() {
	        return new RedisCacheErrorHandler();
	    }

	    @Override
	    public KeyGenerator keyGenerator() {
	        return new MyKeyGenerator()
	    }

从上面可以看出，基本操作是一致的，但是RedisCacheManager创建更加优雅，不在直接依赖redisTemplate。
关于是否使用prefix问题，`RedisCacheConfiguration.defaultCacheConfig()`中代码如下：

	private RedisCacheConfiguration(Duration ttl, Boolean cacheNullValues, Boolean usePrefix,
			CacheKeyPrefix keyPrefix, SerializationPair<String> keySerializationPair,
			SerializationPair<?> valueSerializationPair, ConversionService conversionService) {

		this.ttl = ttl;
		this.cacheNullValues = cacheNullValues;
		this.usePrefix = usePrefix;
		this.keyPrefix = keyPrefix;
		this.keySerializationPair = keySerializationPair;
		this.valueSerializationPair = (SerializationPair<Object>) valueSerializationPair;
		this.conversionService = conversionService;
	}


	public static RedisCacheConfiguration defaultCacheConfig() {

		DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();

		registerDefaultConverters(conversionService);
       // 默认usePrefix为true,是推荐的
		return new RedisCacheConfiguration(Duration.ZERO, true, true, CacheKeyPrefix.simple(),
				SerializationPair.fromSerializer(new StringRedisSerializer()),
				SerializationPair.fromSerializer(new JdkSerializationRedisSerializer()), conversionService);
	}

当然也是可以覆盖禁用的，使用`disableKeyPrefix`, 但明确提出，你需要特别注意，不建议使用。

关于是否使用lock的问题，新版本也提供了可选方案。通过`RedisCacheWriter `来实现：

	static RedisCacheWriter nonLockingRedisCacheWriter(RedisConnectionFactory connectionFactory) {

		Assert.notNull(connectionFactory, "ConnectionFactory must not be null!");

		return new DefaultRedisCacheWriter(connectionFactory);
	}

	static RedisCacheWriter lockingRedisCacheWriter(RedisConnectionFactory connectionFactory) {

		Assert.notNull(connectionFactory, "ConnectionFactory must not be null!");

		return new DefaultRedisCacheWriter(connectionFactory, Duration.ofMillis(50));
	}
可以看出lockingRedisCacheWriter将会有sleep 50ms来处理锁,nonlocking则没有加锁等待，给用户提供了更好的处理方案。

关于全部失效或者清理key的问题，2.0版本处理方案如下：

	@Override
	public void clean(String name, byte[] pattern) {

		Assert.notNull(name, "Name must not be null!");
		Assert.notNull(pattern, "Pattern must not be null!");

		execute(name, connection -> {

			boolean wasLocked = false;

			try {

				if (isLockingCacheWriter()) {
					doLock(name, connection);
					wasLocked = true;
				}
             // 这里仍旧是使用的keys操作
				byte[][] keys = Optional.ofNullable(connection.keys(pattern)).orElse(Collections.emptySet())
						.toArray(new byte[0][]);

				if (keys.length > 0) {
					connection.del(keys);
				}
			} finally {

				if (wasLocked && isLockingCacheWriter()) {
					doUnlock(name, connection);
				}
			}

			return "OK";
		});
	}
这里仍旧使用的是`keys`命令，坑仍在。后续使用`scan`操作也许是更好的选择，但最终还是要依据自己的业务需求来定制。

总结：
> **开源项目的坑无处不在，即使是spring**。
> 无论是什么版本，使用`prefix`是更好的选择，也是趋势所在。
> keys操作对性能的影响始终未能彻底消除，建议使用key expire机制来规避。（生产环境keys操作也是尽可能要避免的）。
> redis缓存key的大小，无论是性能还是存储的影响都很大，强烈建议在业务允许范围内尽可能减小key的大小(比如使用MD5,有一定碰撞率)。
