> <h1></h1>
- [**数据库**](#数据库)
	- [**SQL数据库**](#SQL数据库)
		- [外键](#外键)
			- [如果两个表之间有关系,是否一定要有外键](#如果两个表之间有关系,是否一定要有外键)
			- [建和不建外键约束有什么区别？](#建和不建外键约束有什么区别？)
		- [约束](#约束)
			- [为什么不想要 null 的值](#为什么不想要null的值)
			- [带AUTO_INCREMENT约束的字段值是从1开始的吗？](#带AUTO_INCREMENT约束的字段值是从1开始的吗？)
			- [并不是每个表都可以任意选择存储引擎？](#并不是每个表都可以任意选择存储引擎？)

- **资料**
	- [前端少年汪-掘金笔记](https://juejin.cn/collection/7114526702730117127)
	- [100道MySQL数据库经典面试题解析（收藏版）](https://juejin.cn/post/6844904166939164680)
	- [oppo后端16连问](https://mp.weixin.qq.com/s?__biz=MzkyMzU5Mzk1NQ==&mid=2247506444&idx=1&sn=73473586c81c1bd44fe6cfd7edd62581&source=41#wechat_redirect)
	- [记一次接口性能优化实践总结：优化接口性能的八个建议](https://mp.weixin.qq.com/s?__biz=MzkyMzU5Mzk1NQ==&mid=2247505910&idx=1&sn=232ef078235aa1ba1ed1551f3898323e&source=41#wechat_redirect)
	- [后端程序员必备：书写高质量SQL的30条建议](https://mp.weixin.qq.com/s?__biz=MzkyMzU5Mzk1NQ==&mid=2247505878&idx=1&sn=c37653719ef771c13b681cc42366d8bd&source=41#wechat_redirect)
	- [Go Channel（收藏以备面试）](https://juejin.cn/post/7004673581342785543)
	- [推荐几个可以写到简历上的Go方向优质开源项目（需花点心思研究）](https://juejin.cn/post/7038967716459315208)



<br/><br/><br/>

***
<br/>
> <h1 id="数据库">数据库</h1>
<br/>

> <h1 id="SQL数据库"> SQL数据库 </h1>

***
<br/><br/><br/>
> <h2 id="外键">外键</h2>

<br/><br/>
> <h3 id="如果两个表之间有关系,是否一定要有外键">如果两个表之间有关系,是否一定要有外键</h3>

不需要,使用外键若是分布式、数据量非常大的时候,比如十万往上以后非常消耗性能,这个时候可以**使用业务逻辑代码**进行联系,避免资源阻塞.

<br/><br/>

># <h3 id="建和不建外键约束有什么区别？">[建和不建外键约束有什么区别?](./数据库SQL(III).md#外键-开发场景)</h3>

***
<br/><br/><br/>
> <h2 id="约束">约束</h2>

<br/><br/>

># <h3 id="为什么不想要null的值">[为什么不想要null的值](./数据库SQL(III).md#约束-思考题)</h3>


<br/><br/>

># <h3 id="带AUTO_INCREMENT约束的字段值是从1开始的吗？">[带AUTO_INCREMENT约束的字段值是从1开始的吗？](./数据库SQL(III).md#约束-思考题)</h3>


<br/><br/>

># <h3 id="">[并不是每个表都可以任意选择存储引擎？](./数据库SQL(III).md#约束-思考题)</h3>

)





