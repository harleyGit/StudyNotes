># Podfile 基本格式
```
# 指明依赖库的来源地址
source 'https://github.com/CocoaPods/Specs.git'
# 说明平台是ios，版本是12.0
platform :ios, '12.0'
# 忽略引入库的所有警告（强迫症者的福音啊）
inhibit_all_warnings!

target 'AFNetworkInterpret' do
  use_frameworks!
  
  # 针对AFNetworkInterpret target引入AFNetworking
	pod 'AFNetworking', '~> 4.0.1'
	

  # Pods for AFNetworkInterpret
  
  # target 'AFNetworkInterpretTests' do
    # inherit! :search_paths
    
    # 针对AFNetworkInterpretTests target引入OCMock
    # pod 'OCMock', '~> 2.0.1'
  # end
  

end


# 这个是cocoapods的一些配置,官网并没有太详细的说明,一般采取默认就好了,也就是不写.
post_install do |installer|
   installer.pods_project.targets.each do |target|
     puts target.name
   end
end

```


<br/>
`参考资料：`[](https://www.jianshu.com/p/b8b889610b7e)
