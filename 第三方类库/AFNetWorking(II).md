># 属性
**`@property (nonatomic, assign) BOOL removesKeysWithNullValues;`**
&emsp;  在`AFNetWorking`只要把这个`removesKeysWithNullValues=YES.`后台返回的JSON数据中存在空的键值对,将会被自动删除,可以避免空值做操作,造成崩溃问题。
