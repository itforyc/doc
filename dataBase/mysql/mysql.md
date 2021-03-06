
[数据库中间件详解](https://mp.weixin.qq.com/s/XisPkWGkB-a3dR2lrEjkwQ)

[MySQL 大表优化方案](https://segmentfault.com/a/1190000006158186)

[MySQL 优化实施方案](https://www.cnblogs.com/clsn/p/8214048.html)

[MySQL 索引优化](https://mp.weixin.qq.com/s/1jnwkifOGTYJhCCwu6zwHg)

[以MySQL为例，详解数据库索引原理及深度优化](https://mp.weixin.qq.com/s/29cvchKqQc7Ng4T_gT2J3A)

[MySQL 索引B+树原理，以及建索引的几大原则](https://mp.weixin.qq.com/s/qvm5GEgLNv5dnbcVA-d5Ag)

[MySQL常用命令，34道练习题](https://segmentfault.com/a/1190000013259974)

[什么影响了 MySQL 性能？](https://mp.weixin.qq.com/s/7W0yQWUxNy_RFwWCnLA3-A)

[什么影响了数据库查询速度?](https://mp.weixin.qq.com/s/6RsM_tXiDwbPrCTPDZAkLw)

[超级全面的MySQL优化面试解析](https://mp.weixin.qq.com/s/1mMyD6kvPZw2piR152hEqQ)

[当数据量达到百万级别的时候，分页该如何处理？](https://blog.csdn.net/Danny_idea/article/details/90664571)

[海量数据下的分库分表最佳实战](https://mp.weixin.qq.com/s/Iv_UGUVlyOX7C3HQ99BdFw)

[Mysql高性能优化规范建议](https://mp.weixin.qq.com/s/AXvFIeehRBIk-mdVrKkKSA)

[一条SQL语句在MySQL中如何执行的](https://mp.weixin.qq.com/s/QU4-RSqVC88xRyMA31khMg)

[深入理解数据库编程中的超时设置](https://mp.weixin.qq.com/s/eZvKBwXkB1H5HInC8HDyjA)

[mysql系列文章](https://www.cnblogs.com/linjiqin/category/283837.html)

[1000行MySQL命令](https://mp.weixin.qq.com/s/t-DHR6nqUIpqsHX_O4npOA)

## 分库分表

[sharding-jdbc源码解析](https://github.com/YunaiV/sharding-jdbc)

[Sharding-Sphere实战：实现类多租户分库分表](https://mp.weixin.qq.com/s/X34qKPAs2aHQTvE_6VStiQ)

[数据库分库分表后，如何部署上线？](https://mp.weixin.qq.com/s/fDNTMx1KCNsIwcbJPdHcwQ)

[记一次生产中的【分表】踩坑经历！](https://mp.weixin.qq.com/s/tUtGgjVXuqeW3QM7t4LKdg)

[一次分表踩坑实践的探讨](https://mp.weixin.qq.com/s/HmUHfFedd7kEW37vnuY1sA)

[“分库分表" ？选型和流程要慎重，否则会失控](https://mp.weixin.qq.com/s/ibU8DUpDgyBjbaN52UdtiQ)

## 读写分离

[SpringBoot + MyBatis + MySQL 读写分离实战](https://www.cnblogs.com/cjsblog/p/9712457.html)

### 一次分库踩坑
1、临时方案

由于需求紧、人手缺的情况下，整个处理的过程分为几个阶段。

第一阶段应该是去年底，当时运维反应 MySQL 所在的主机内存占用很高，整体负载也居高不下，导致整个 MySQL 的吞吐量明显降低（写入、查询数据都明显减慢）。

为此我们找出了数据量最大的几张表，发现大部分数据量在7/8000W 左右，少数的已经突破一亿。

通过业务层面进行分析发现，这些数据多数都是用户产生的一些日志型数据，而且这些数据在业务上并不是强相关的，甚至两三个月前的数据其实已经不需要实时查询了。

因为接近年底，尽可能的不想去动应用，考虑是否可以在运维层面缓解压力；主要的目的就是把单表的数据量降低。

原本是想把两个月之前的数据直接迁移出来放到备份表中，但在准备实施的过程中发现一个大坑。

表中没有一个可以排序的索引，导致我们无法快速的筛选出一部分数据！这真是一个深坑，为后面的一些优化埋了个地雷；即便是加索引也需要花几个小时（具体多久没敢在生产测试）。
如果我们强行按照时间进行筛选，可能查询出 4000W 的数据就得花上好几个小时；这显然是行不通的。

于是我们便想到了一个大胆的想法：这部分数据是否可以直接不要了？

这可能是最有效及最快的方式了，和产品沟通后得知这部分数据真的只是日志型的数据，即便是报表出不来今后补上也是可以的。

于是我们就简单粗暴的做了以下事情：

修改原有表的表名，比如加上( _190416bak)。
再新建一张和原有表名称相同的表。
这样新的数据就写到了新表，同时业务上也是使用的这个数据量较小的新表。

虽说过程不太优雅，但至少是解决了问题同时也给我们做技术改造预留了时间。

2、分表方案

之前的方案虽说可以缓解压力，但不能根本解决问题。

有些业务必须得查询之前的数据，导致之前那招行不通了，所以正好我们就借助这个机会把表分了。

我相信大部分人虽说没有做过实际做过分表，但也见过猪跑；网上一搜各种方案层出不穷。

我认为最重要的一点是要结合实际业务找出需要 sharding 的字段，同时还有上线阶段的数据迁移也非常重要。

3、时间

可能大家都会说用 hash 的方式分配得最均匀，但我认为这还是需要使用历史数据的场景才用哈希分表。

而对于不需要历史数据的场景，比如业务上只查询近三个月的数据。

这类需求完成可以采取时间分表，按照月份进行划分，这样改动简单，同时对历史数据也比较好迁移。

于是我们首先将这类需求的表筛选出来，按照月份进行拆分，只是在查询的时候拼接好表名即可；也比较好理解。

4、哈希

刚才也提到了：需要根据业务需求进行分表策略。

而一旦所有的数据都有可能查询时，按照时间分表也就行不通了。（也能做，只是如果不是按照时间进行查询时需要遍历所有的表）

因此我们计划采用 hash 的方式分表，这算是业界比较主流的方式就不再赘述。

采用哈希时需要将 sharding 字段选好，由于我们的业务比较单纯；是一个物联网应用，所有的数据都包含有物联网设备的唯一标识（IMEI），并且这个字段天然的就保持了唯一性；大多数的业务也都是根据这个字段来的，所以它非常适合来做这个 sharding 字段。

在做分表之前也调研过 MyCAT 及 sharding-jdbc(现已升级为 shardingsphere)，最终考虑到对开发的友好性及不增加运维复杂度还是决定在 jdbc 层 sharding 的方式。

但由于历史原因我们并不太好集成 sharding-jdbc，但基于 sharding 的特点自己实现了一个分表策略。

这个简单也好理解：

int index = hash (sharding字段) % 分表数量;

select xx from 'busy_'+index where sharding 字段 =xxx;
其实就是算出了表名，然后路由过去查询即可。

只是我们实现的非常简单：修改了所有的底层查询方法，每个方法都里都做了这样的一个判断。

并没有像 sharding-jdbc 一样，代理了数据库的查询方法；其中还要做 SQL解析-->SQL路由-->执行SQL-->合并结果 这一系列的流程。

如果自己再做一遍无异于重新造了一个轮子，并且并不专业，只是在现有的技术条件下选择了一个快速实现达成效果的方法。

不过这个过程中我们节省了将 sharding 字段哈希的过程，因为每一个 IMEI 号其实都是一个唯一的整型，直接用它做 mod 运算即可。

还有一个是需要一个统一的组件生成规则，分表后不能再依赖于单表的字段自增了；方法还是挺多的：

比如时间戳+随机数可满足大部分业务。
UUID，生成简单，但没法做排序。
雪花算法统一生成主键ID。
大家可以根据自己的实际情况做选择。

5、业务调整

因为我们并没有使用第三方的 sharding-jdbc 组件，所有没有办法做到对代码的低侵入性；每个涉及到分表的业务代码都需要做底层方法的改造（也就是路由到正确的表）。

考虑到后续业务的发展，我们决定将拆分的表分为 64 张；加上后续引入大数据平台足以应对几年的数据增长。

这里还有个小细节需要注意：分表的数量需要为 2∧N 次方，因为在取模的这种分表方式下，即便是今后再需要分表影响的数据也会尽量的小。
再修改时只能将表名称进行全局搜索，然后加以修改，同时根据修改的方法倒推到表现的业务并记录下来，方便后续回归测试。

当然无法避免查询时利用非 sharding 字段导致的全表扫描，这是所有分片后都会遇到的问题。

因此我们在修改分表方法的底层查询时同时也会查看是否有走分片字段，如果不是，那是否可以调整业务。

比如对于一个上亿的数据是否还有必要存在按照分页查询、日期查询？这样的业务是否真的具有意义？

我们尽可能的引导产品按照这样的方式来设计产品或者做出调整。

但对于报表这类的需求确实也没办法，比如统计表中某种类型的数据；这种我们也可以利用多线程的方式去并行查询然后汇总统计来提高查询效率。

有时也有一些另类场景：

比如一个千万表中有某一特殊类型的数据只占了很小一部分，比如说几千上万条。
这时页面上需要对它进行分页查询是比较正常的（比如某种投诉消息，客户需要一条一条的单独处理），但如果我们按照 IMEI 号或者是主键进行分片后再分页查询那就比较蛋疼了。

所以这类型的数据建议单独新建一张表来维护，不要和其他数据混合在一起，这样不管是做分页还是 like 都比较简单和独立。

6、验证

代码改完，开发也单测完成后怎么来验证分表的业务是否正常也比较麻烦。

一个是测试麻烦，再一个是万一哪里改漏了还是查询的原表，但这样在测试环境并不会有异常，一旦上线产生了生产数据到新的 64 张表后想要再修复就比较麻烦了。

所以我们取了个巧，直接将原表的表名修改，比如加一个后缀；这样在测试过程中观察前后台有无报错就比较容易提前发现这个问题。

上线流程

测试验收通过后只是分表这个需求的80%，剩下如何上线也是比较头疼。

一旦应用上线后所有的查询、写入、删除都会先走路由然后到达新表；而老数据在原表里是不会发生改变的。

7、数据迁移

所以我们上线前的第一步自然是需要将原有的数据进行迁移，迁移的目的是要分片到新的 64 张表中，这样才会对原有的业务无影响。

因此我们需要额外准备一个程序，它需要将老表里的数据按照分片规则复制到新表中；

在我们这个场景下，生产数据有些已经上亿了，这个迁移过程我们在测试环境模拟发现耗时是非常久的。而且我们老表中对于 create_time 这样用于筛选数据的字段没有索引（以前的技术债），所以查询起来就更加慢了。

最后没办法，我们只能和产品协商告知用户对于之前产生的数据短期可能会查询不到，这个时间最坏可能会持续几天（我们只能在凌晨迁移，白天会影响到数据库负载）。

8、总结

这便是我们这次的分表实践，虽说不少过程都不优雅，但受限于条件也只能折中处理。

但我们后续的计划是，修改我们底层的数据连接（目前是自己封装的一个 jar 包，导致集成 sharding-jdbc 比较麻烦）最终逐渐迁移到 sharding-jdbc .

最后得出了几个结论：

一个好的产品规划非常有必要，可以在合理的时间对数据处理（不管是分表还是切入归档）。
每张表都需要一个可以用于排序查询的字段（自增ID、创建时间），整个过程由于没有这个字段导致耽搁了很长时间。
分表字段需要谨慎，要全盘的考虑业务情况，尽量避免出现查询扫表的情况。


[mysql官方下载](https://dev.mysql.com/downloads/mysql/)

[win10安装mysql8](https://www.cnblogs.com/tangyb/p/8971658.html)

mysql8设置时区,my.ini

com.mysql.cj.exceptions.InvalidConnectionAttributeException: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone

```
[mysqld]
default-time-zone='+08:00'
```

### 优化

#### 分页优化
[MySQL 百万级数据量分页查询如何优化](https://mp.weixin.qq.com/s/iLNspcQYdWPUywKClxbdlg)

#### SpringBoot
在 `application.properties` 配置文件中添加 MySQL 数据库的相关配置：
mysql5

##### mysql数据库连接
```properties
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus?useSSL=false
spring.datasource.username=root
spring.datasource.password=123456
```

`mysql8`以上（spring boot 2.1）
注意：`driver`和`url`的变化
```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus?serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=123456
```

注意：

1、这里的 url 使用了 ?serverTimezone=GMT%2B8 后缀，因为Spring Boot 2.1 集成了 8.0版本的jdbc驱动，这个版本的 jdbc 驱动需要添加这个后缀，否则运行测试用例报告如下错误：

java.sql.SQLException: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more 

2、这里的 `driver-class-name` 使用了  `com.mysql.cj.jdbc.Driver` ，在 jdbc 8 中 建议使用这个驱动，之前的 `com.mysql.jdbc.Driver` 已经被废弃，否则运行测试用例的时候会有 WARN 信息
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. 
The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.

