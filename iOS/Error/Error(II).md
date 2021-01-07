># The iOS Simulator deployment target is set to x.x, but the range of supported deployment target versions for this platform is 8.0 to 12.0. (in target 'xxx')

```
post_install do |installer|
  installer.pods_project.targets.each do |target|
 target.build_configurations.each do |config|
  if config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'].to_f < 8.0
    config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '8.0'
     end
   end
  end
end
```
&emsp;  将这个判断添加到Podfile文件的最后，然后执行pod install，再次运行项目，你会发现整个世界都清净了。
&emsp;  上面对第三方库进行了判断若DEPLOYMENT_TARGET<8.0就会切换成8.0，可以说是一劳永逸，下次苹果再去提高最低版本只需要修改版本号就可以了。

