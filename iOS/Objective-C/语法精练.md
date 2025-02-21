- **Block 为空判断**
- **synchronized**
- **宏定义dispatch_semaphore**



<br/>

***
<br/>

># Block 为空判断


```
#define SAFE_CALL_BLOCK1(blockFunc, ...)    \
    if (blockFunc) {                        \
        blockFunc(__VA_ARGS__);              \
    }


- (void)testHong:(NSString *) title value:(int)value completeHandle:(void (^)(int value, bool isSuccess, NSString *method )) completeHandler {
    
    SAFE_CALL_BLOCK1(completeHandler, value, true, title);
    
    NSLog(@"SAFE_CALL_BLOCK1 调用完了");
}

- (void) downloadPicAction {
    
    [self testHong:@"SAFE_CALL_BLOCK1 测试" value:0 completeHandle:^(int value, bool isSuccess, NSString *method) {
        NSLog(@"-------->>    没了");
    }];
}

```


<br/>

打印：

```

2020-12-05 16:03:19.957618+0800 ImageLoadSDK[42472:428784] -------->>    没了
2020-12-05 16:03:21.638529+0800 ImageLoadSDK[42472:428784] SAFE_CALL_BLOCK1 调用完了

```



<br/>


***
<br/>


># synchronized


```
- (void) downloadPicAction {
    @synchronized (self) {
        NSLog(@"🍊 <<<<<<<<<   @synchronized start ");
    }
   
    
    NSString *gifUrl = @"https://user-gold-cdn.xitu.io/2019/3/27/169bce612ee4dc21";
    
    @synchronized (self) {
        NSLog(@"🍊 @synchronized end >>>>>>>>>>>>>>");
    }
}

```

<br/>
打印：

```

2020-12-05 17:40:32.419736+0800 ImageLoadSDK[43544:487862] 🍊 <<<<<<<<<   @synchronized start 
2020-12-05 17:40:35.441956+0800 ImageLoadSDK[43544:487862] 🍊 @synchronized end >>>>>>>>>>>>>>

```



<br/>


***
<br/>

># 宏定义dispatch_semaphore


```

#define LOCK(lock) dispatch_semaphore_wait(lock, DISPATCH_TIME_FOREVER);
#define UNLOCK(lock) dispatch_semaphore_signal(lock);


- (void) downloadPicAction {
  
    
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
    
    NSLog(@"🍎 🍎 《〈《〈《〈《〈《 start");
    
    LOCK(semaphore);
    
    NSLog(@"🍎 🍎 =============== middle");

    UNLOCK(semaphore);
    
    NSLog(@"🍎 🍎 》〉》〉》〉》〉》 end");
}


```

<br/>
打印：

```

2020-12-05 17:47:01.673416+0800 ImageLoadSDK[43629:493010] 🍎 🍎 《〈《〈《〈《〈《 start
2020-12-05 17:47:04.755571+0800 ImageLoadSDK[43629:493010] 🍎 🍎 =============== middle
2020-12-05 17:47:08.697352+0800 ImageLoadSDK[43629:493010] 🍎 🍎 》〉》〉》〉》〉》 end

```

