
- **Block 属性变量使用**
- **Block 作为参数使用**



<br/>

***
<br/>



># Block 属性变量使用

```
typedef void(^AddGroupMember)(id memberModel);


@property(nonatomic, copy) AddGroupMember addMember;

//①先要执行它，这是对块的初始化，否则在调用self.addMember() 会崩溃
self.addMember = ^(NSMutableArray* memberModels) {
            ccc.contancts = memberModels;
            ccc.isInOrRe = InvitedOrRemvoveMemberTypeInvited;
            [weakSelf.controller presentViewController:navigationController animated:YES completion:nil];
        };

//②传递值,然后跳转到①中执行块内的代码
self.addMember(friendList);

```


<br/>

***
<br/>


># Block 作为参数使用

```
//PrefectureViewModel.h  文件声明
+ (void) requestInteractPrefectureCommentForPageNum:(NSInteger) page  success:(void (^)(NSMutableArray<JSONModel *> *models))interactPrefecture;


//PrefectureViewModel.m  文件定义
+ (void) requestInteractPrefectureCommentForPageNum:(NSInteger) page  success:(void (^)(NSMutableArray<JSONModel *> *models))interactPrefecture{
         NSMutableArray *comments = [NSMutableArray arrayWithCapacity:1];
         for(int i = 0; i < 9; i ++){
                JSONModel *jm = [JSONModel new];
                [comments addObject: jm];
         }
        
        interactPrefecture(comments);
}


//其他类中使用
[PrefectureViewModel requestInteractPrefectureCommentForPageNum:1 success:^(NSMutableArray<JSONModel *> * _Nonnull models) {
        //使用，打印
        self.dataArray = models;
        
        NSLog(@"------->> %@", models);
    }];
```




<br/>

***
<br/>

参考资料：

[Block 详解](https://blog.csdn.net/majiakun1/article/details/80741215)

[Block的用法和实现](https://www.jianshu.com/p/d28a5633b963)
