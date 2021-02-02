- [**NSOperation**](https://www.jianshu.com/p/d8caf596d5d0)
- **NSOperationQueue 控制串行执行、并发执行**
- **NSOperation 属性、方法**
	- 优先级
	- 控制串行执行、并发执行
	- completionBlock
	- 操作依赖
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




># NSOperation 属性、方法


> **属性**

- 队列优先级
	- 	**`queuePriority`:**

```
typedef NS_ENUM(NSInteger, NSOperationQueuePriority){
        NSOperationQueuePriorityVeryLow = -8L,
        NSOperationQueuePriorityLow = -4L,
        NSOperationQueuePriorityNormal = 0,
        NSOperationQueuePriorityHigh = 4,
        NSOperationQueuePriorityVeryHigh = 8
}

```

- 控制串行执行、并发执行

	- `maxConcurrentOperationCount`：控制的不是并发线程的数量，而是一个队列中同时能并发执行的最大操作数。而且一个操作也并非只能在一个线程中运行。

- completionBlock

	completionBlock	可读可写，当 finished 的值更改为 YES 时，将执行该代码块。
	
	completionBlock通常是一个子线程执行；因此，不应该使用这个块来做刷新UI。
	
	finished=YES可能因为它被取消或者因为它完成任务而结束；在编写代码块时，应该考虑到这一点。
 
	completionBlock 类似于GCD的函数  dispatch_block_notify() ，监听 dispatch_block 完成时 dispatch_queue 调用 notification_block




<br/>
<br/>

> **方法**

- 操作依赖

	- `- (void) addDependency:(NSOperation *)op`  :添加依赖，使当前操作依赖于操作 op 的完成;

	- `- (void) removeDependency:(NSOperation *)op`  :移除依赖，取消当前操作对操作 op 的依赖;

	- `@property(readonly, copy) NSArray<NSOperation *> *dependencies;` :  在当前操作开始执行之前完成执行的所有操作对象数组.





<br/>

***
<br/>

>#  NSOperation 子类





<br/>


***
<br/>
