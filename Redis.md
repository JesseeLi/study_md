## Redis 相关

### Redis Bloom Filter 插件安装使用

#### 安装

Redis Bloom Filter地址：https://github.com/RedisBloom/RedisBloom

目前最新版 v2.6.12，但是编译报错。暂时使用 v2.2.18

```
wget https://github.com/RedisBloom/RedisBloom/archive/refs/tags/v2.2.18.tar.gz
tar zxvf v2.2.18.tar.gz
cd RedisBloom-2.2.18
make
```

此时会生成一个 redisbloom.so 文件

redis 安装目录下新建 ext 目录，将 redisbloom.so 复制到该目录内。

修改 redis 配置文件 redis.conf。 MODULES 模块内添加：

```
loadmodule /usr/local/redis/ext/redis_bloom_filter.so
```

保存。重新启动 redis

进入 redis-cli 命令，

```
bf.exists bloom_filter_key test_val
```

返回0表示安装成功。

#### 使用

| 指令       | 含义                                       | 示例                          | 备注                                              |
| ---------- | ------------------------------------------ | ----------------------------- | ------------------------------------------------- |
| BF.RESERVE | 配置多少数量下有多大误差，误差越小性能越差 | BF.RESERVE 布名 0.001 1000000 | 100万条数据允许0.1%的误差                         |
| BF.ADD     | 向某个布隆过滤器中添加数据                 | BF.ADD 布名 值1               | 返回1证明插入成功                                 |
| BF.EXISTS  | 判断某个布隆过滤是否存在一个值             | BF.EXISTS 布名 值1            | 返回1说明存在，返回0说明不存在                    |
| BF.MADD    | 向某个布隆过滤器中插入多个值               | BF.MADD 布名 值2 值3 值4      | 返回 1) (integer) 1 2) (integer) 1 3) (integer) 1 |
| BF.MEXISTS | 判断某个布隆过滤是否存在多个值             | BF.MEXISTS 布名 值2 值3 值5   | 返回 1) (integer) 1 2) (integer) 1 3) (integer) 0 |
