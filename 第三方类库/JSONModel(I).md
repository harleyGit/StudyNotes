>#字段id、description 冲突处理
##方法一
`json 字段展示`
![description 冲突](https://upload-images.jianshu.io/upload_images/2959789-b9eff7f5e785f6df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

***正常解析，使用 po 打印得：***
![因为description在JOSNMondel源码中为一个方法，所以不能为一个字段](https://upload-images.jianshu.io/upload_images/2959789-a7a6a60f00689ec7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



`HGMyInformationModel.h 文件`
```
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface HGMyInformationModel : HGMyModel

//用户高清头像
@property(nonatomic, copy) NSString *avatar_hd;
//用户大图头像180*180
@property(nonatomic, copy) NSString *avatar_large;
//性别
@property(nonatomic, copy) NSString *gender;
//用户个人描述
@property(nonatomic, copy) NSString *introduction;  //description

//......还有其他字段，不便都列举 
@end

NS_ASSUME_NONNULL_END
```

`HGMyInformationModel.m文件`
```
#import "HGMyInformationModel.h"


@implementation HGMyInformationModel

//initWithDictionary 这个方法在 JSONMondel 1.8.0 版本已弃用，建议使用initWithModelToJSONDictionary，但是这个方法对于关键字段id有用，对于descripton无用，解析为空。
+ (JSONKeyMapper *)keyMapper {
      return [[JSONKeyMapper alloc] initWithDictionary:@{@"description":@"introduction"}];
}

//设置所有的属性为可选(所有属性值可以为空),这里这段注释是不需要的，加上也无所谓
+(BOOL)propertyIsOptional:(NSString *)propertyName {
    return YES;
}

@end

```
![字段替换成功](https://upload-images.jianshu.io/upload_images/2959789-fbd04ec9d84169f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
###方法二
![id 关键字](https://upload-images.jianshu.io/upload_images/2959789-947b85e2801cd1ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Order_details.h`
![属性字段](https://upload-images.jianshu.io/upload_images/2959789-627f166d57511847.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Order_details.m`
![新定义的字段处理](https://upload-images.jianshu.io/upload_images/2959789-da9eba3c925b8130.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`网络请求字段进行解析`
![调用方法进行解析](https://upload-images.jianshu.io/upload_images/2959789-b1e72ec761479343.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)








<br/>
***
<br/>
参考资料：[JSONModel的使用](https://www.jianshu.com/p/606f421bc8aa)
                    [JSONModel的使用](https://www.jianshu.com/p/606f421bc8aa)
[IOS-JSONModel使用](https://www.jianshu.com/p/ae05d9410d3c)
[iOS中JSONModel的使用](https://www.jianshu.com/p/3cce56f374b4)
