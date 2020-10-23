# 持久化

# 两种方式
```
RDB: 指定的时间间隔进行快照存储
AOF：记录每次服务器的写操作，服务器重启会恢复这些数据
```

# RDB配置文件配置
```
#时间策略
save 100 1    100s内有1条写操作就产生快照
save 300 10   300s内有10条写操作就产生快照
save ""       禁用RDB配置
```

# AOF配置文件配置
```
# 同步方式
appendfsync everysec  每秒同步一次，速度折中，损失1s数据
appendfsync always 立即同步，很慢
appendfsync no redis不处理，交给os处理，很快，不安全
```

# 工作原理

### 创建定时任务
```
配置文件进行设置
hz 10  表示1s内执行10次
hz 100 不建议超过100
hz 500 最大500，频率越高，cpu消耗越大
```

# RDB持久化原理
```
#手动触发
save 阻塞主进程，不推荐线上使用
bgsave fork子进程，阻塞子进程
# bgsave实现流程
bgsave->检测子进程->触发持久化操作->rdbSaveBackground->fork子进程->rdb持久化
```

# AOF持久化原理
```
#手动触发
bgrewriteaof,主要流程是写入命令、重写，重写写入命令后先写入buf，再同步到磁盘
# bgrewriteaof实现流程
bgrewriteaof->检测子进程->触发持久化->rewriteAppendOnlyFileBackgroud->fork进程->重写入到新aof->rename替换旧aof
                                  ->主进程->buf->新aof
                                         ->buf->旧aof

```

# 从持久化中恢复数据
```
#重新启动redis
#同时存在rdb和aof
启动时检查->aof存在即恢复aof：因为数据比较完整，最多损失1s数据
         ->rdb存在->恢复rdb
```

# 性能优化
```
#持久化都会设计fork，而fork操作涉及阻塞导致影响性能，怎么优化？
降低fork频率：自动触发降低自动评论，手动触发降低手动触发效率
加大Redis最大使用内存：防止fork耗时过长
```
