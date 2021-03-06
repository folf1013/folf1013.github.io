---
layout: post
title: redis的数据结构(上)
description: redis的数据结构
keywords: redis
categories : [缓存]
tags : [redis]
---

redis支持List,Set,SortedSet,Hash,String等数据结构。本文分析了Set和SortedSet缓存。

# SortedSet

- string类型，通过hash表实现
- 不重复
- 根据score有序

## 命令

sortedSet中命令基本从三个维度设计：index，member,score，从这个维度出发，用倒序，正序，区间，交集，并集设计了20个命令。

命令 | 描述 | 返回值
---|---|---
zadd key score1 value1 score2 value2 | 添加value1,value2俩个元素 | 新添加的元素个数 
zcard key | 元素的个数 | 元素的个数
zcount key min max | 分数在min,max之间分数元素的个数 | 返回个数
zincrby key increment member | 给指定member分数添加increment | 返回添加之后member的分数
zinterstore destination keysnum key1... | 将key1...作交集，(member相同,score相加)，将结果存到desination中 | destination存入数据个数
zlexcount key min max | score一致时，member从[min,[max的个数，（-为最小值，+最大值) | 返回值为个数
zrange key start stop [withscores] | 获取start,stop的member[是否带score](0为开始，-1为最后一个元素) | 返回的数据member[score]
zrangebylex key min max [limit offset count] | score一致时，根据字段顺序[min,[max获取字段member，offset从第几个开始，count取几个 | 返回的member
zrangebyscore key min max [withscores] [limit offset count] | 返回有序集合中min,max的成员列表 | 返回member
zrank key member | 返回有序集中指定成员的排名 | long(从0开始)
zrem key member[member...] | 在key中删除[member...] | 返回删除的元素个数
zremrangebylen key min max  | score一致时，字典顺序删除[min,[max | 返回删除的元素个数
zremrangebyrank key min max | 自然数顺序删除min,max | 返回删除的元素个数
zremrangebyscore key min max | 分数顺序删除min,max | 返回删除的元素个数
zrevrange key max min[withscores] | 与zrange相比，只是显示倒叙 | 返回member
zrevrangebyscore key max min[withscores] |  按照分数倒序，指定区间内member，(max,min) | 返回member
zrerange key member | 倒序的第几位 | 倒序的数值
zscore key member | member的分数 | score
zunionscore destination numkeys key [key...] | 将keys求并集放到destination中 | 返回destination新增个数
ZSCAN key cursor [MATCH pattern] [COUNT count]  | 迭代有序集合中的元素（包括元素成员和元素分值）| 返回匹配到的元素

sortedSet提供了丰富的方法，可以分类为：
- 向集合中添加元素(zadd)。
- 元素个数，总个数，member区间个数，score区间个数，index区间个数（zcard,zlencount）
- 根据index区间，member区间，member区间倒序，score区间，score区间倒序获取元素(zrange,zrangebylen,zrerangebylen,zrangebyscore,zrerangebyscore)
- 根据多个，index区间，member区间，score区间删除元素(zrem，zremrangebyrank,zremrangebylen,zremrangebyscore)
- member在顺序中的index，在逆序中的index(zrank,zrerank)
- member的分数(zcore)
- 俩个key的交集，并集存到另外一个key中(zinterstore,zunionscore)
- lex的前提是score一致


# Set

- string类型的无序集合
- 集合成员是唯一的
- 集合中最大的成员数为 2^32 - 1 

## 命令

命令 | 描述 | 返回值
---|---|---
sadd key member1 [member2...] | 添加member... | 返回添加的个数 
scard key  | 返回key中set的个数 | 
sdiff key [key2...] | key与key2...的差集 | 返回key与[key2...]的差集的元素
sdiffstore destination key [key2...] | key与key2...的差集存到destination中 | 返回元素的个数
sinter key [key2...] | 交集 | 返回交集的元素
sinterstore destination key [key2...] | key与key2...的交集存到destination中 | 返回元素的个数
sismember key member | member是否是key中元素 | 是：1 ，否:0
smembers key | 返回key的所有元素
smove source destination member | 将member从source移动到destination中
spop key | 返回第一个元素，先进先出 | member
srandmember key count | 从key中随机返回count个元素 | 返回的member
srem key member [member...] | 从key中删除member... | 返回删除的元素个数
sunion key [key...] | 求key的并集 | 返回并集的元素
sunionstore destination key [key...] | 将key的并集存放到destination中 | 返回元素的个数
sscan key curse [Match pattern] [offset count] | 匹配遍历key | 返回匹配上的元素

set提供的方法与sorted的方法类似，提供了添加，删除，交集，并集，个数等防范。
