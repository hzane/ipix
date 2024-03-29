# 埃文区县库本地查询服务

- [ipplus360](http://www.ipplus360.com)  重要的事情说几遍？， 几遍都不重要，听的人认为重要才重要
- ip区县库免费，宣称每月更新，压缩文件有密码，如果用MacOS，会比较悲剧。用Windows直接就能找到密码
- 可以导入数据库中使用，大约24M条不算少，我在3.22上放了一份. use-conf db-aiwen， 会找到数据库访问方式的
- points_3 是利用区县数据库导出的

```bash
#echo aiwen ip database
use-conf db-aiwen
```

## 解释

- 客户端集成百度定位SDK什么的最简单，免费而又高精度；当然也有缺点：`不是我想出来的，就是缺点`
- 到写这篇文档的时间，公司客户端尚不支持Android 6.0及以上的定位授权申请，好在只有35% 的用户而已
- 目前已经没有免费高精度的IP定位服务可以利用。普通精度的有很多选择。收费的更多了，但凡公司有点钱，就都不算贵
- 目前短视频客户端上报数据频度不超过100次每秒，用多个并发请求在线服务应该也可以保证不会累积消息
- 多数免费服务都限制多个时间维度的频次，这个可能有些影响，用稍长的过期时间本地缓存重复请求可能会优化些。效果取决于IP重复度
- 既然如此，为什么要些这么个程序
- 这是唯一能找到的免费区县地址库。如果还能找到另用一家免费的，非常欢迎分享
- 埃文仍然在维护这个数据，保持每月更新的频率
- 埃文收集了15万个不重复区县地址，感觉覆盖度应该很好，毕竟中国统共才两千多个县
- 用邻近城市中心划分用户比起精确的行政区划划分差不了什么，甚至可能效果会更好
- 在线服务比本地服务，性能上没法比
- KD-Tree 搜索15万个点 不是个事儿
- 区县坐标库基本不用更新，建国50年也没多少次行政区划变更
- IP地址库跟随埃文更换表很简单

## AiWEN 数据库

- 具体格式不解释了

### 根据IP地址搜索区县

``` sql
select areacode,country,multiarea
  from district_1
  where minip <= inet_aton(?)
  order by minip desc limit 1
```

- 从多个覆盖区间中选择一个

### 根据LatLng搜索区县

- 能够搜集到的数据是各区县的中心坐标
- 利用最邻近算法搜索到某个坐标的最近区县
- 用`haversine`方法计算两个坐标间距离
- 最邻近搜索算法使用KD-Tree

## APIs

### 根据IP判断坐标

> http://{{IPIX}}/v25/geo/first.json?ip=222.22.22.22

Query | Default  | Description
--- | --- | ---
ip | *required* |

### 根据坐标判断区县

> http://{{IPIX}}/v25/geo/nearest.json?location={lat},{lng}

Query | Default | Description
--- | --- | ---
location | *required* | 34,118
