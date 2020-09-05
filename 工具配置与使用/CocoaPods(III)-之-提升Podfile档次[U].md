># 建立一个Podfile 文件
-  在终端定位到某个文件夹
`$ cd /Users/harleyhuang/Documents/GitHub/我的项目`

- 初始化一个 Podfile 文件夹
`pod init`
这样就自动生成了一个 `Podfile` 的文件夹，一应格式都俱全了，wonderful！！

- 在 Podfile 文件夹下开始写需要的三方类库
```
#下面一行是指明依赖库的来源地址
source 'https://github.com/CocoaPods/Specs.git'

#说明平台是ios，版本是12.0
platform :ios, '12.0'
# 忽略引入库的所有警告（强迫症者的福音啊）
inhibit_all_warnings!

install! 'cocoapods', 
  :deterministic_uuids => false, 
  :integrate_targets => false

# 针对 '我的项目' 引入各种类库
target '我的项目' do  
  use_frameworks!
  
  pod 'SnapKit', '~> 5.0.1'
  pod 'Moya', '~> 13.0'
#  pod 'Moya/RxSwift', '~> 13.0'
  pod 'SwiftyJSON', '~> 5.0.0'
  pod 'HandyJSON', '~> 5.0.1'
#  pod 'ImageSlideshow', '~> 1.7'
  pod 'ImageSlideshow/Kingfisher', '~> 1.7.0'
  pod 'Kingfisher', '~> 4.10.0'
#  pod 'CryptoSwift', '~> 1.2'
#  pod "CKMnemonic"

#  pod 'DZNEmptyDataSet', '~> 1.8.1'  # https://github.com/dzenbot/DZNEmptyDataSet
  
#  pod 'RxSwift',    '~> 4.0'
#  pod 'RxCocoa',    '~> 4.0'
#  pod 'RxDataSources', '~> 3.1.0'
  
#  pod 'Spring', :git => 'https://github.com/MengTo/Spring.git'
  pod 'R.swift', '~> 5.1.0'
  pod 'IQKeyboardManagerSwift', '~> 6.5.4'
#  pod 'SwiftLint'
  pod 'Bugly', '~> 2.5.0'
#   chain
#  pod 'BitconchIo', '~> 1.0.3'

  pod 'EosioSwift', '~> 0.1.1'
  pod 'EosioSwiftAbieosSerializationProvider', '~> 0.1.1'
  pod 'EosioSwiftSoftkeySignatureProvider', '~> 0.1.1'
  pod 'EosioSwiftEcc', '~> 0.1.3'
#  pod "EosioSwiftVault", "~> 0.2.1"

  #友盟+
  pod 'UMCCommon', '~> 2.1.1'
  pod 'UMCAnalytics', '~> 6.0.5'
  

  #Objective-C
  pod 'MBProgressHUD', '~> 1.1.0'
#  pod 'IQKeyboardManager'
  pod 'BlocksKit', '~>2.2.5'
  pod 'Masonry', '~> 1.0.2'
  pod 'MJRefresh', '~> 3.1.12'
  
#  install_all_flutter_pods(flutter_application_path)

  #针对我得项目Tests引入OCMock
  target '我得项目Tests' do
      inherit! :search_paths
      pod 'OCMock', '~> 2.0.1'
  end

end



# 这个是cocoapods的一些配置,官网并没有太详细的说明,一般采取默认就好了,也就是不写.
post_install do |installer|       
   installer.pods_project.targets.each do |target| 
     puts target.name 
   end
end

```


[](https://www.jianshu.com/p/b8b889610b7e)
[](https://www.jianshu.com/p/f18d9f1a1477?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)

