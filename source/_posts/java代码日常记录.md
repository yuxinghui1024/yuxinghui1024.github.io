---
title: java代码日志
date: 2020-02-26 15:38:06
---
### java8计算两个日期之间的工作日
``` java
/**
* 计算两个日期之间的工作日，包含起始日不包含结束日期
* @param start 开始时间
* @param end 截止时间
* @param legalHolidays 法定节假日
* @param legalWorkdays 法定工作日：周末调休
* @return 工作日
*/
public static int workDaysBetweenDate(LocalDate start, LocalDate end, Set<String> legalHolidays, Set<String> legalWorkdays){
   if (Objects.isNull(start) || Objects.isNull(end) 
           || Objects.isNull(legalHolidays) || Objects.isNull(legalWorkdays) ||start.isAfter(end)){
       throw new IllegalArgumentException("输入的参数不合法");
   }
   int betweenDays = Period.between(start, end).getDays();
   DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
   int minusDays = 0;
   while (start.isBefore(end)){
       String startString = dateTimeFormatter.format(start);
       if (legalHolidays.contains(startString)){
           minusDays--;
       }
       if ((start.getDayOfWeek() == DayOfWeek.SATURDAY || start.getDayOfWeek() == DayOfWeek.SUNDAY)
               && !legalWorkdays.contains(startString)){
           minusDays--;
       }
       start = start.plusDays(1);
   }
   return betweenDays+minusDays;
}
```
### 雪花算法
``` java
public class IdWorker {

    //因为二进制里第一个 bit 为如果是 1，那么都是负数，但是我们生成的 id 都是正数，所以第一个 bit 统一都是 0。

    //机器ID  2进制5位  32位减掉1位 31个
    private long workerId;
    //机房ID 2进制5位  32位减掉1位 31个
    private long datacenterId;
    //代表一毫秒内生成的多个id的最新序号  12位 4096 -1 = 4095 个
    private long sequence;
    //设置一个时间初始值    2^41 - 1   差不多可以用69年
    private long twepoch = 1585644268888L;
    //5位的机器id
    private long workerIdBits = 5L;
    //5位的机房id
    private long datacenterIdBits = 5L;
    //每毫秒内产生的id数 2 的 12次方
    private long sequenceBits = 12L;
    // 这个是二进制运算，就是5 bit最多只能有31个数字，也就是说机器id最多只能是32以内
    private long maxWorkerId = -1L ^ (-1L << workerIdBits);
    // 这个是一个意思，就是5 bit最多只能有31个数字，机房id最多只能是32以内
    private long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);

    private long workerIdShift = sequenceBits;
    private long datacenterIdShift = sequenceBits + workerIdBits;
    private long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;
    private long sequenceMask = -1L ^ (-1L << sequenceBits);
    //记录产生时间毫秒数，判断是否是同1毫秒
    private long lastTimestamp = -1L;
    public long getWorkerId(){
        return workerId;
    }
    public long getDatacenterId() {
        return datacenterId;
    }
    public long getTimestamp() {
        return System.currentTimeMillis();
    }



    public IdWorker(long workerId, long datacenterId, long sequence) {

        // 检查机房id和机器id是否超过31 不能小于0
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(
                    String.format("worker Id can't be greater than %d or less than 0",maxWorkerId));
        }

        if (datacenterId > maxDatacenterId || datacenterId < 0) {

            throw new IllegalArgumentException(
                    String.format("datacenter Id can't be greater than %d or less than 0",maxDatacenterId));
        }
        this.workerId = workerId;
        this.datacenterId = datacenterId;
        this.sequence = sequence;
    }

    // 这个是核心方法，通过调用nextId()方法，让当前这台机器上的snowflake算法程序生成一个全局唯一的id
    public synchronized long nextId() {
        // 这儿就是获取当前时间戳，单位是毫秒
        long timestamp = timeGen();
        if (timestamp < lastTimestamp) {

            System.err.printf(
                    "clock is moving backwards. Rejecting requests until %d.", lastTimestamp);
            throw new RuntimeException(
                    String.format("Clock moved backwards. Refusing to generate id for %d milliseconds",
                            lastTimestamp - timestamp));
        }

        // 下面是说假设在同一个毫秒内，又发送了一个请求生成一个id
        // 这个时候就得把seqence序号给递增1，最多就是4096
        if (lastTimestamp == timestamp) {

            // 这个意思是说一个毫秒内最多只能有4096个数字，无论你传递多少进来，
            //这个位运算保证始终就是在4096这个范围内，避免你自己传递个sequence超过了4096这个范围
            sequence = (sequence + 1) & sequenceMask;
            //当某一毫秒的时间，产生的id数 超过4095，系统会进入等待，直到下一毫秒，系统继续产生ID
            if (sequence == 0) {
                timestamp = tilNextMillis(lastTimestamp);
            }

        } else {
            sequence = 0;
        }
        // 这儿记录一下最近一次生成id的时间戳，单位是毫秒
        lastTimestamp = timestamp;
        // 这儿就是最核心的二进制位运算操作，生成一个64bit的id
        // 先将当前时间戳左移，放到41 bit那儿；将机房id左移放到5 bit那儿；将机器id左移放到5 bit那儿；将序号放最后12 bit
        // 最后拼接起来成一个64 bit的二进制数字，转换成10进制就是个long型
        return ((timestamp - twepoch) << timestampLeftShift) |
                (datacenterId << datacenterIdShift) |
                (workerId << workerIdShift) | sequence;
    }

    /**
     * 当某一毫秒的时间，产生的id数 超过4095，系统会进入等待，直到下一毫秒，系统继续产生ID
     * @param lastTimestamp
     * @return
     */
    private long tilNextMillis(long lastTimestamp) {

        long timestamp = timeGen();

        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }
    //获取当前时间戳
    private long timeGen(){
        return System.currentTimeMillis();
    }

    /**
     *  main 测试类
     * @param args
     */
    public static void main(String[] args) {
        IdWorker worker = new IdWorker(1,1,0);
        for (int i = 0; i < 22; i++) {
            System.out.println(worker.nextId());
        }
    }
}
```

### 分布式锁redis实现

``` java
/**
  * 分布式锁接口类，只有一台机器可以获得锁
  *
*/
public interface RedisLock {
    /**
     * 尝试加锁，不可重入
     * 发生在执行任务之前，只有申请到锁才可以继续执行
     * @param lockName
     * @return
     */
    String tryLock(String lockName, long timeout);

    /**
     * 尝试加锁,使用默认超时时间
     *
     * @param lockName the lock name
     * @return the string
     */
    String tryLock(String lockName);

    /**
     * 释放锁
     * 发生在获得锁但是后续处理失败，否则之后无法再获得锁。
     * @param lockName 锁实例名字，唯一
     * @return
     */
    boolean unLock(String lockName, String identifier);
}
```
``` java
/**
  * 设置过期时间：可以防止锁删除失败，导致线程无线等待；带来的问题，A线程设置的锁过期后，B线程成功设置锁，A线程执行完后删除了B线程的锁；
  * 解决方案，删除的时候获取锁的value，相等则删除；带来的问题，删除锁操作不是原子操作（判断相等的时候，C线程成功设置锁，删除了C线程的锁），
  * 解决方案，rubby脚本。
  * 第三方实现，Redisson
  *
*/
public class RedisLockImpl implements RedisLock {

    private static final String REDIS_LOCK_NAME_PREFIX = "REDIS_LOCK_";
    private static final String LOCK_SUCCESS = "OK";

    //NX是不存在时才set， XX是存在时才set， EX是秒，PX是毫秒
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";

    /**
     * 锁的释放时间, 上锁后超过此时间则自动释放锁. 单位为毫秒.
     */
    private static final long DEFAULT_TIMEOUT_MILLISECOND = 1000 * 60 * 10;



    @Override
    public String tryLock(String lockName, long timeout) {
        if (Objects.isNull(lockName))
            return null;
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            String value = UUID.randomUUID().toString();
            String key = REDIS_LOCK_NAME_PREFIX + lockName;
            String result = jedis.set(key, value, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, timeout);
            if (LOCK_SUCCESS.equals(result)){
                return value;
            }
        }catch (Exception e){
            System.err.printf("获取锁失败,lockName:%s", lockName);
        }finally {
            if (Objects.nonNull(jedis)){
                jedis.close();    
            }
        }

        return null;
    }

    @Override
    public String tryLock(String lockName) {
        return tryLock(lockName, DEFAULT_TIMEOUT_MILLISECOND);
    }

    @Override
    public boolean unLock(String lockName, String identifier) {
        if (Objects.isNull(lockName) || Objects.isNull(identifier))
            return false;

        Jedis jedis = null;
        String key = REDIS_LOCK_NAME_PREFIX + lockName;
        try {
            jedis = jedisPool.getResource()；
            StringBuilder script = new StringBuilder();
            //if redis.call('get','orderkey')=='1111' then return redis.call('del','orderkey') else return 0 end
            script.append("if redis.call('get','").append(key).append("')").append("=='").append(identifier).append("'").
                    append(" then ").
                    append("    return redis.call('del','").append(key).append("')").
                    append(" else ").
                    append("    return 0").
                    append(" end");
            return Integer.parseInt(jedis.eval(script.toString()).toString()) > 0 ? true : false;
        }catch (Exception e){
            System.err.printf("释放锁失败，lockName:%s,identifier:%s", lockName, identifier);
        }finally {
            if (Objects.nonNull(jedis)){
                jedis.close();
            }
        }
        return false;
    }
}
```
