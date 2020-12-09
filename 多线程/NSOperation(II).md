- **NSOperationQueue 控制串行执行、并发执行**
- **NSOperation 属性、方法**
- **NSOperation 子类**




<br/>

***
<br/>


```
NSInvocationOperation *op1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(taskOne) object: nil];

- (void) taskOne {
            for();
}

NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{

}];

NSOperationQueue *operation = [[NSOperationQueue alloc] init];

[operation addOperation: op1];
[operation addOperation: op2];

```


<br/>

**`- (void) addOperationWithBlock:(void (^)(void))block {}`**

```
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
[queue addOperationWithBlock:^{
      
}];

```


<br/>

***
<br/>


># NSOperationQueue 控制串行执行、并发执行

`maxConcurrentOperationCount`：控制的不是并发线程的数量，而是一个队列中同时能并发执行的最大操作数。而且一个操作也并非只能在一个线程中运行。

```


```

<br/>

***
<br/>

># NSOperation 属性、方法

**`queuePriority`:**

```
typedef NS_ENUM(NSInteger, NSOperationQueuePriority){
        NSOperationQueuePriorityVeryLow = -8L,
        NSOperationQueuePriorityLow = -4L,
        NSOperationQueuePriorityNormal = 0,
        NSOperationQueuePriorityHigh = 4,
        NSOperationQueuePriorityVeryHigh = 8
}

```


<br/>

**方法**

`- (void) addDependency:(NSOperation *)op`  :添加依赖，使当前操作依赖于操作 op 的完成;

`- (void) removeDependency:(NSOperation *)op`  :移除依赖，取消当前操作对操作 op 的依赖;

`@property(readonly, copy) NSArray<NSOperation *> *dependencies;` :  在当前操作开始执行之前完成执行的所有操作对象数组.



<br/>

***
<br/>

>#  NSOperation 子类





<br/>

***
<br/>



<br/>

***
<br/>

