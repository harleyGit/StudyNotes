># iOS13 UIWindow 适配
**`AppDelegate.m 文件`**
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    if (@available(iOS 13,*)) {
            return YES;
        } else {
            self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
            UINavigationController *rootNavgationController = [[UINavigationController alloc] initWithRootViewController:[ViewController new]];
            self.window.rootViewController = rootNavgationController;
            [self.window makeKeyAndVisible];
            return YES;
        }
    
}

```

`SceneDelegate.m 文件`
```
- (void)scene:(UIScene *)scene willConnectToSession:(UISceneSession *)session options:(UISceneConnectionOptions *)connectionOptions {    
   
        self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
        self.window.windowScene = (UIWindowScene*)scene;
        UINavigationController *rootNavgationController = [[UINavigationController alloc] initWithRootViewController:[ViewController new]];
        self.window.rootViewController = rootNavgationController;
        [self.window makeKeyAndVisible];    
}

```

&emsp; 这样就可以兼容iOS13和iOS12的UIWindow显示了。
