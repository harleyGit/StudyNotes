># UITableViewCell 内存泄漏
#`情况一：`
```
	if indexPath.section == 3, indexPath.row == 2 {
                let items = self.applyItems[indexPath.section]
                inputCell?.setCellContent(cellContent: items[indexPath.row], showContent: self.parameters) {[weak self] (key, value) in
                    self?.parameters[key] = value
                }
            }

```
&emsp; 若是去掉[weak self]则会发生内存泄漏，这是因为`self->tableView->cell->self`,造成了内存泄漏。加上`[weak self]`就可避免了

<br/>






<br/>
***
<br/>
># 根视图切换
#`Applegate.m 文件`
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

   //.....省略逻辑代码

   if (user.taiJiDaoToken.length && user.accountID.length && user.password.length) {
        NSString *url = [NSString stringWithFormat:@"%@%@",TaiChi_PublicURL,USER_USER_LOGIN];
        NSDictionary *parameters = @{@"mode":@"tjd", @"username":user.accountID, @"password":user.password};
        //注释掉下面的代码即可防止内存泄漏
        [LoginViewController requestOfLoginURL:url parameters:parameters];
    }else {
        [self setTaiChiRootVC];
    }

    return YES;
}

-(void)setLoginVC
{
    LoginViewController *loginVC = [[LoginViewController alloc]init];
    UINavigationController *navLoginVC = [[UINavigationController alloc]initWithRootViewController:loginVC];
    self.window.rootViewController = navLoginVC;
    [self.window makeKeyAndVisible];
}
-(void)setTaiChiRootVC
{
    TaiChiTabViewController *tcVC = [[TaiChiTabViewController alloc]init];
    self.window.rootViewController = tcVC;
    [self.window makeKeyAndVisible];

}
```

#`LoginViewController.m 文件`
```
+ (void) requestOfLoginURL:(NSString *)url parameters:(NSDictionary *)parameters {
        {
          //网络请求登录成功之后，切换根视图控制器
          //内存泄漏的地方
          [(AppDelegate *)[UIApplication sharedApplication].delegate setTaiChiRootVC];
        }

}
```

&emsp; 上面的代码会发生内存泄漏,定位的代码是` [(AppDelegate *)[UIApplication sharedApplication].delegate setTaiChiRootVC]`,将` [LoginViewController requestOfLoginURL:url parameters:parameters];`注掉即可。

&emsp; 这是因为每次项目运行后，都会




<br/>
***
<br/>





<br/>
***
<br/>
