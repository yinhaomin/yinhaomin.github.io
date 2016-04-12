---
layout: post
title: How to automatically manage the jedis resource
comments: true
keywords: jedis resource manage
---

```
        Jedis jedis = null;
        try {
            jedis = redisFactory.getClient();
            for (ToRedisQueueTag tag : tagList) {
                if (tag == null) {
                    continue;
                }
                ToRedisQueueTag huiTag = getHuiTagFromAllTag(tag);
                jedis.lpush(InRedisConstant.HUI_TAG_QUEUE_KEY, messageSerializer.serialize(huiTag));

                if (tag.getType() == ItemTypeEnum.ITEM_TYPE.getType()) {
                    // 设置精选百度惠Tag信息
                    String huiTagValue = messageSerializer.serialize(huiTag.getTagList());
                    jedis.hset(InRedisConstant.HUI_SEARCH_ITEM_TAG_KEY, Long.toString(huiTag.getId()), huiTagValue);
                    // 设置精选所有Tag信息
                    String value = messageSerializer.serialize(tag.getTagList());
                    jedis.hset(InRedisConstant.HUI_ALL_SEARCH_ITEM_TAG_KEY, Long.toString(tag.getId()), value);
                } else {
                    log.error("The input tag type error, the type is: " + tag.getType());
                }
            }
        } catch (Exception e) {
            log.error("Exception in writeTagListToRedisQueue: ", e);
            return false;
        } finally {
            redisFactory.returnResource(jedis);
        }
```

如上图为了将redis的代码和和业务代码剥离开来，使用如下的方式:
第一种:使用AOP和Annotation

```
    @Autowired
    private RedisFactory redisFactory;

    @Autowired
    @Qualifier(value = "jsonMessageSerializer")
    private MessageSerializer messageSerializer;

    @Around(value = "@annotation(redisResource)")
    public void redisProcess(ProceedingJoinPoint joinPoint, RedisResource redisResource) throws Throwable {
        Jedis jedis = null;
        try {
            jedis = redisFactory.getClient();

            Object[] args = joinPoint.getArgs();
            for (int i = 0; i < args.length; i++) {
                if (args[i] instanceof Jedis) {
                    args[i] = jedis;
                }
            }
            joinPoint.proceed(args);
        } catch (Exception e) {
            log.error("Exception in writeTagListToRedisQueue: ", e);
        } finally {
            redisFactory.returnResource(jedis);
        }
    }
```

测试代码:

```
    @Test
    public void testDiscoveryTagToRedisQueueService() {
        List<ToRedisQueueTag> tagList = new ArrayList<ToRedisQueueTag>();
        ToRedisQueueTag tag = new ToRedisQueueTag();
        tag.setId(127391);
        // ItemTypeEnum 对应在ES中的item,discovery,sale的枚举
        tag.setType(ItemTypeEnum.DISCOVERY_TYPE.getType());
        List<TagObject> tagObjectList = new ArrayList<TagObject>();
        TagObject obj = new TagObject();
        obj.setId(117322);
        obj.setName("喷雾");
        tagObjectList.add(obj);
        tag.setTagList(tagObjectList);
        tagList.add(tag);

        AspectJProxyFactory factory = new AspectJProxyFactory(tagToRedisQueueService);
        factory.addAspect(redisAOP);
        TagToRedisQueueService proxy = factory.getProxy();
        Jedis jedis = new Jedis("");
        proxy.writeTagListToRedisQueue(tagList, jedis);

        // for (int i = 0; i < 100; i++) {
        // tagToRedisQueueService.writeTagListToRedisQueue(tagList,null);
        // }
    }
```

第二种: 使用回调函数

```
@Component
public class RedisFactory {
    private JedisPool pool;

    @Autowired
    private RedisConfig config;

    @PostConstruct
    public void init() {
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxTotal(config.getMaxTotal());
        poolConfig.setMaxIdle(config.getMaxIdle());
        poolConfig.setMinIdle(config.getMinIdle());
        poolConfig.setMaxWaitMillis(config.getMaxWait());
        poolConfig.setTestOnBorrow(true);
        poolConfig.setTestOnReturn(true);
        pool = new JedisPool(poolConfig, config.getHost(), config.getPort());
        //        pool = new JedisPool(config.getHost(), config.getPort());
    }

    public Jedis getClient() {
        Jedis jedis = pool.getResource();
        if (config.getDb() != 0) {
            jedis.select(config.getDb());
        }
        return jedis;
    }

    public void returnResource(Jedis jedis) {
        pool.returnResource(jedis);
    }

    /**
     * 对每次业务的执行需要进行jedis连接、具体业务逻辑、jedis关闭进行统一封装
     *
     * @param callBack 需要用户自定义实现的业务接口
     * @param <T>
     *
     * @return 返回具体业务逻辑执行结果
     */
    public <T> T execute(JedisCallBack<T> callBack) {
        Jedis jedis = getClient();
        try {
            return callBack.execute(jedis);
        } catch (JedisConnectionException ex) {
            returnResource(jedis);
            jedis = null;
            throw ex;
        } finally {
            if (jedis != null) {
                returnResource(jedis);
            }
        }
    }

    /**
     * 业务逻辑执行回调接口
     */
    public interface JedisCallBack<T> {

        T execute(Jedis jedis);
    }
}
```

调用的方式:

```
    public Map<Long, DiscoveryBO> getDiscoveryBOMap(List<Long> ids) {
        Map<Long, DiscoveryBO> result = redisFactory.execute(new RedisFactory.JedisCallBack<Map<Long, DiscoveryBO>>() {
            @Override
            public Map<Long, DiscoveryBO> execute(Jedis jedis) {
                Map<Long, DiscoveryBO> map = new HashMap<Long, DiscoveryBO>();
                map = getDiscoveryBOMaps(jedis,ids);
                return map;
            }
        });
        return result;
    }
```

