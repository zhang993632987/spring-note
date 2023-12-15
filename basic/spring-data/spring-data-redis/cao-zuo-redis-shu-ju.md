# 操作 Redis 数据

有了 RedisTemplate（或 StringRedisTemplate）之后，就可以开始保存、获取以及删除 key-value 条目了。

RedisTemplate 的大多数操作都是下表中的子 API 提供的：

| 方法               | 子 API 接口                                                                                       | 描述                                    |
| ---------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------- |
| opsForValue()    | ValueOperations\<K, V>                                                                         | 操作具有简单值的条目                            |
| opsForList()     | ListOperations\<K, V>                                                                          | 操作具有 list 值的条目                        |
| opsForSet()      | SetOperations\<K, V>                                                                           | 操作具有 set 值的条目                         |
| opsForZSet()     | ZSetOperations\<K, V>                                                                          | 操作具有 ZSet 值（排序的 set）的条目               |
| opsForHash()     | HashOperations\<K, HK, HV>                                                                     | 操作具有 hash 值的条目                        |
| boundValueOps(K) | BoundValueOperations\<K, V>                                                                    | 以绑定指定 key 的方式，操作具有简单值的条目              |
| boundListOps(K)  | BoundListOperations\<K, V>                                                                     | 以绑定指定 key 的方式，操作具有 list 值的条目          |
| boundSetOps(K)   | BoundSetOperations\<K, V>                                                                      | 以绑定指定 key 的方式，操作具有 set 值的条目           |
| boundZSet(K)     | BoundZSetOperations\<K, V>                                                                     | 以绑定指定 key 的方式，操作具有 ZSet 值（排序的 set）的条目 |
| boundHashOps(K)  | BoundHashOperations\<K, V>                                                                     | 以绑定指定 key 的方式，操作具有 hash 值的条目          |
