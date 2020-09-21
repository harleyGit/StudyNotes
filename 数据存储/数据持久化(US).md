># App 沙盒目录
![沙盒目录](https://upload-images.jianshu.io/upload_images/2959789-4555d8856d1f4d72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 根目录：`let homeDirectory = NSHomeDirectory()`;
- Documnets: 用户文档目录，苹果建议将程序中建立的或在程序中浏览到的文件数据(程序运行时生成的需要持久化的数据)保存在该目录下，iTunes备份和恢复的时候会包括此目录;
- Library:  该目录下有两个子目录：Caches 和 Preferences
  -  Library/Preferences目录：包含应用程序的偏好设置文件(比如:保存用户名、密码、字体大小等设置)。不应该直接创建偏好设置文件，而是应该使用NSUserDefaults类来取得和设置应用程序的偏好；
   -  Library/Caches目录：保存应用运行时生成的需要持久化的数据，iTunes不会备份此目录。`（曾将用户录制的视频和fmdb数据库存放于此，发现每个一段时间[大约3天左右]，caches目录下的数据都会被自动清空）`。经查询，有[相关资料](https://link.jianshu.com?t=https://my.oschina.net/u/1266799/blog/165631)佐证。简而言之，就是在内存不够的情况下，系统会自动清除caches目录下的数据;
- SystemData：存放系统数据，无对外暴露的接口；
-  tmp：用于存放临时文件，保存应用程序再次启动过程中不需要的信息，重启后清空。


<br/>
**`获取沙盒不同的目录`**
-  获取根目录(整个应用程序所在目录)
`let homedDirectory = NSHomeDirectory()`

- Document 目录
```
//documentDirectory 枚举：搜索路径目录
//allDomainsMask 枚举：限定了文件搜索的区域
let documentPaths = NSSearchPathForDirectoriesInDomains(.documentDirectory, .allDomainsMask, true)
let documentPath = documentPaths.first ?? ""
//方法二
let documentPath2 = NSHomeDirectory() + "/Documents"
print(documentPath2)
// /Users/kehaoran/Library/Developer/CoreSimulator/Devices/62FD8F53-9E45-4714-A7A1-890E85E184CE/data/Containers/Data/Application/A28238F8-2935-4032-9189-C7DDFFD9FDEB/Documents
print(documentPath)
// /Users/kehaoran/Library/Developer/CoreSimulator/Devices/62FD8F53-9E45-4714-A7A1-890E85E184CE/data/Containers/Data/Application/A28238F8-2935-4032-9189-C7DDFFD9FDEB/Documents
```

- Library 目录
```
let documentPaths = NSSearchPathForDirectoriesInDomains(.libraryDirectory, .allDomainsMask, true)
```



<br/>
***
<br/>


># 本地存储
```
- (void)viewDidLoad {
    [super viewDidLoad];
    //获取根目录
    NSString *homeDir = NSHomeDirectory();
    NSLog(@"homeDir = %@",homeDir);

    //获取Documents
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSLog(@"paths = %@",paths);

    //获取caches
    NSArray *cachepaths = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
    NSLog(@"cachepaths = %@",cachepaths);

    NSString *temString = NSTemporaryDirectory();
    NSLog(@"tempaths = %@",temString);

    //有 【学生信息.txt】这个文件就不需要创建，若没有就需要重新创建
    //运行时路径：xxx/Application/(序列号) /tmp/学生信息.txt， 虽然序列号每次运行都在变，但是存储的文件每次都是这个文件
    temString =  [temString stringByAppendingPathComponent:@"学生信息.txt"];
    NSLog(@"tempaths = %@",temString);

    NSDictionary *dic = [NSDictionary dictionaryWithObjectsAndKeys:@"李白",@"name",@"27",@"age", @"2020.3.2", @"时间",nil];
    [dic writeToFile:temString atomically:YES];

    NSDictionary *localDic = [NSDictionary dictionaryWithContentsOfFile:temString];
    NSLog(@"学生信息 = %@",localDic);

}

```
打印：
```
2020-03-03 18:39:32.958441+0800 TestSave[62076:2224259] homeDir = /Users/harleyhuang/Library/Developer/CoreSimulator/Devices/287514A8-73B5-44B5-8565-732BD32F9DA1/data/Containers/Data/Application/321D65B7-E4C0-4A74-AF1B-01A9EFA23C7B
2020-03-03 18:39:32.958607+0800 TestSave[62076:2224259] paths = (
    "/Users/harleyhuang/Library/Developer/CoreSimulator/Devices/287514A8-73B5-44B5-8565-732BD32F9DA1/data/Containers/Data/Application/321D65B7-E4C0-4A74-AF1B-01A9EFA23C7B/Documents"
)
2020-03-03 18:39:32.958722+0800 TestSave[62076:2224259] cachepaths = (
    "/Users/harleyhuang/Library/Developer/CoreSimulator/Devices/287514A8-73B5-44B5-8565-732BD32F9DA1/data/Containers/Data/Application/321D65B7-E4C0-4A74-AF1B-01A9EFA23C7B/Library/Caches"
)
2020-03-03 18:39:32.958826+0800 TestSave[62076:2224259] tempaths = /Users/harleyhuang/Library/Developer/CoreSimulator/Devices/287514A8-73B5-44B5-8565-732BD32F9DA1/data/Containers/Data/Application/321D65B7-E4C0-4A74-AF1B-01A9EFA23C7B/tmp/
2020-03-03 18:39:32.958914+0800 TestSave[62076:2224259] tempaths = /Users/harleyhuang/Library/Developer/CoreSimulator/Devices/287514A8-73B5-44B5-8565-732BD32F9DA1/data/Containers/Data/Application/321D65B7-E4C0-4A74-AF1B-01A9EFA23C7B/tmp/学生信息.txt
2020-03-03 18:39:32.959749+0800 本地存储[62076:2224259] 学生信息 = {
    age = 27;
    name = "\U674e\U767d";
    "\U65f6\U95f4" = "2020.3.2";
}

```
![文本存储字段信息图](https://upload-images.jianshu.io/upload_images/2959789-76bc4257774288eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
***
<br/>
># plist 文件存取
```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    // 存储的路径
    NSString *path = [NSString stringWithFormat:@"%@/Documents/PersonalInformation.plist",NSHomeDirectory()];
    // 设置不备份，否则App Store会拒绝
    [ZJJDontBackUpTool addSkipBackupAttributeToItemAtURL:[NSURL URLWithString:path]];
    // 模拟数据请求到的字典，要存储起来
    // 但是必须要经过筛选，因为里面对象如果是null，就无法存储
    // 判断过程再次省略
    NSDictionary *dic = @{@"name":@"HarleyHuang",@"age":@"28",@"sex":@"male", @"iPhoneNumber(WeiChat)": @"17133853768"};
    [dic writeToFile:path atomically:YES];
    // 存入完成，取出数据
    NSDictionary *dict = [[NSDictionary alloc] initWithContentsOfFile:path];
    
    NSLog(@"文件地址：%@，\n信息：%@", path, dict);
}
```
打印：
```
2020-03-03 18:56:45.671461+0800 TestSave[62370:2242568] 文件地址：/Users/harleyhuang/Library/Developer/CoreSimulator/Devices/287514A8-73B5-44B5-8565-732BD32F9DA1/data/Containers/Data/Application/B72147EF-9BDD-4161-A1F3-D68464B13298/Documents/PersonalInformation.plist，
信息：{
    age = 28;
    "iPhoneNumber(WeiChat)" = 17133853768;
    name = HarleyHuang;
    sex = male;
}
```
![plist 文件内容](https://upload-images.jianshu.io/upload_images/2959789-7a98cf282fe11277.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
***
<br/>
># JSON 文件存储
```
import UIKit

class FileHandle: NSObject {
    
    ///文件写入
    public class func writToFile(fileName: String, _ fileContent: Any) {
        let path = FileHandle.createFoler(folderName: "log/")
        if path.isEmpty {
            return
        }
        print("日志文件夹path: \(path)")
        guard let os = OutputStream(toFileAtPath: "\(path)\(fileName)",
                                    append: false) else { return }
        os.open()
        JSONSerialization.writeJSONObject(fileContent,
                                          to: os,
                                          options: JSONSerialization.WritingOptions.prettyPrinted,
                                          error: NSErrorPointer.none)
        os.close()
    }
    
    /// 文件读取
    public class func readFile(fileName: String) -> [[String: Any]]? {
        let path = FileHandle.folderPath(folderName:"log/")
        guard let data = NSData(contentsOfFile: "\(path)\(fileName)") else {
            return [[String: Any]]()
        }
        let array = try? JSONSerialization.jsonObject(with: data as Data) as? [[String: Any]]
        
        return array
    }
    
    ///获取文件名
    public class func folderPath(folderName: String = "") -> String {
        var folderName = folderName
        if folderName.isEmpty {
            let appName = Bundle.main.infoDictionary?["CFBundleDisplayName"] as? String ?? "HarleyDeFile"
            folderName = appName
        }
        //folderName = "\(Int(NSDate().timeIntervalSince1970))\(folderName)"
        //NSHomeDirectory():获取程序的Home目录
        //方法1获取Cache目录：let cachePath = NSHomeDirectory() + "/Library/Caches"
        let cachePaths = NSSearchPathForDirectoriesInDomains(FileManager.SearchPathDirectory.cachesDirectory,
                                                             FileManager.SearchPathDomainMask.userDomainMask, true)
        let cachePath = cachePaths[0]
        let path = cachePath + "/\(folderName)"
        print("文件路径:\(path)")
        
        return path
    }
    
    /// 判断是不是文件夹
    private class func isFolder (path: String) -> Bool {
        var directoryExists = ObjCBool.init(false)
        let fileExists = FileManager.default.fileExists(atPath: path, isDirectory: &directoryExists)

        return fileExists && directoryExists.boolValue
    }
    
    
    
    /// 创建一个文件夹
    private class func createFoler(folderName: String = "") -> String {
        let path = FileHandle.folderPath(folderName: folderName)
        if !FileManager.default.fileExists(atPath: path) {
            if let _ = try? FileManager.default.createDirectory(atPath: path, withIntermediateDirectories: true, attributes: nil) {
                return path
            }else {
                return ""
            }
        }
        return path
    }
    
    
    /// 获取文件大小
    func fileSize(url: URL) -> UInt64 {
        var fileSize : UInt64 = 0
        do {
            let attr = try FileManager.default.attributesOfItem(atPath: url.path)
            fileSize = attr[FileAttributeKey.size] as! UInt64
             
            let dict = attr as NSDictionary
            fileSize = dict.fileSize()
        } catch {
            print("Error: \(error)")
        }
        
        return fileSize
    }
}
 ```
**`调用案例`**
```
@objc func readJSONFile()  {

        let array = FileHandle.readFile(fileName: "walletAddressLog.json")
        guard let jsonData = array else {
            return
        }
        print("日志内容： \(jsonData)")
        
        /*
        // 读 json：main.json 已拖放至 Xcode 项目 Bundle 里
        let data = NSData(contentsOfFile: "/Users/harleyhuang/Desktop/main.json")
        let array = try? JSONSerialization.jsonObject(with: data! as Data) as? [[String: Any]]

        print("--->> \(String(describing: array))")
         */
    }
    
    @objc func writeJSONFile()  {

        //
        let array = [
            ["companyName": "哔哩哔哩-B站",
            "salary": "25-50K·15薪",
            "financing": "已上市",
            "employee": "1000-9999人",
            "hiringJobs": ["jobName": "iOS Develop",
                           "technologyRequest": ["skil 1": "精通iOS平台开发技术,包括UI、网络、多媒体等方面;熟悉iOS开发调试工具和性能分析工具的使用；", "skil 2": "熟悉OpenGLES/Metal/AVFoundation/ffmpeg等技术框架优先；有音视频相关产品经验优先；"]]],
            ["companyName": "携程旅行网",
                       "salary": "18-22K·15薪",
                       "financing": "已上市",
                       "employee": "10000人以上",
                       "hiringJobs": ["jobName": "iOS Develop",
                                      "technologyRequest": ["skil 1": "精通 Objective-C，熟练掌握React Native等编程语言", "skil 2": "精通多线程开发，深入理解Runtime、RunLoop、内存管理，Block及底层原理", "skil 3": "熟练掌握单例、代理、工厂、观察者、命令等常用的设计模式", "skil 4": "掌握一定的客户端架构设计能力，在MVC、MVP、MVVM、分层架构设计等方面有实践经验","skil 5": "掌握HTTP、HTTPS、TCP、UDP等通讯协议", "skil 6": "对APP保活、省电、性能调优要有一定的经验"]]],
            ["companyName": "爱奇艺(英语流利说)",
             "salary": "18K-36K",
             "financing": "B轮",
             "employee": "1000-9999人",
             "hiringJobs": ["jobName": "iOS Develop",
                            "technologyRequest": ["skil 1": "精通 Objective-C，熟练掌握 iOS SDK", "skil 2": "熟悉网络通信机制及常用数据传输协议，并有成熟的弱网优化方案", "skil 3": "负责过客户端通用底层库和SDK封装", "skil 4": "对稳定性和性能有超乎寻常的关注", "skil 5": "对稳定性和性能有超乎寻常的关注","skil 6": "对音视频，网络，Hybrid 性能优化等领域有更深层次的了解","skil 7": "熟悉 Swift 语言，熟练掌握 Xcode 等相关开发工具，熟悉 iOS SDK，熟悉 SQLite 等数据库；"]]],
            ["companyName": "百度在线",
             "salary": "15K-30K",
            "financing": "xx轮",
            "employee": "10000人以上",
            "hiringJobs": ["jobName": "iOS Develop",
                           "technologyRequest": ["skil 1": "iOS基础知识扎实，熟悉cocoa touch框架和IOS runtime机制", "skil 2": "熟悉oc内存管理机制、并行开发、GUI开发", "skil 3": "精通面向对象编程，熟悉常用设计模式，拥有较好程序设计思想 ", "skil 4": "具有android等多平台客户端开发经验者优先 "]]],
            ["companyName": "京东集团",
             "salary": "20K-35K",
            "financing": "已上市",
            "employee": "10000人以上",
            "hiringJobs": ["jobName": "iOS Develop",
                           "technologyRequest": ["skil 1": "了解基本数据结构，对MVC设计模式能熟练应用", "skil 2": "熟练掌握objective-c 语言及思想，熟悉objective-c内存管理机制", "skil 3": "熟悉http，socket通信原理，对一些主流开源组件能熟练使用", "skil 4": "良好的编成习惯，积极沟通，乐于分享"]]],
            ["companyName": "阅文集团",
             "salary": "15K-30K*13薪",
            "financing": "已上市",
            "employee": "1000-9999人",
            "hiringJobs": ["jobName": "iOS Develop",
                           "technologyRequest": ["skil 1": "3 年以上 iOS开发经验，能独立承担整块功能模块的设计与开发;", "skil 2": "熟练掌握 OC、swift编程语言，拥有良好的编程规范，熟悉常用的数据结构和设计模式；", "skil 3": "熟悉 iOS 应用开发，对 UI 开发、应用框架、网络通信、多线程IO模型和数据库存储等有深入了解；", "skil 4": "对 App 性能优化有实际经验；"]]],
            ["companyName": "阅文集团",
             "salary": "15K-30K*13薪",
            "financing": "已上市",
            "employee": "1000-9999人",
            "hiringJobs": ["jobName": "iOS Develop",
                           "technologyRequest": ["skil 1": "3 年以上 iOS开发经验，能独立承担整块功能模块的设计与开发;", "skil 2": "熟练掌握 OC、swift编程语言，拥有良好的编程规范，熟悉常用的数据结构和设计模式；", "skil 3": "熟悉 iOS 应用开发，对 UI 开发、应用框架、网络通信、多线程IO模型和数据库存储等有深入了解；", "skil 4": "对 App 性能优化有实际经验；"]]],
            ["companyName": "东方头条",
             "salary": "15K-25K",
            "financing": "不需要融资",
            "employee": "100-499人",
            "hiringJobs": ["jobName": "iOS Develop",
                           "technologyRequest": ["skil 1": "年以上iOS开发工作经验，开发过iOS应用扩展（APP Extension）;", "skil 2": "对Objective-C、Cocoa Touch框架和Swift有丰富的经验和深入的理解 ；", "skil 3": "熟练掌握常用开源第三方框架、iOS Runtime、Git、精通iOS下的并行开发、网络、内存管理；", "skil 4": "熟悉基本的数据结构和算法", "skil 5": "悉C、C++、iOS逆向、对代码重构有心得优先", "skil 6": "有输入法开发经验优先"]]],
            ["companyName": "趣头条",
             "salary": "25-40K·16薪",
            "financing": "已上市",
            "employee": "1000-9999人",
            "hiringJobs": ["jobName": "iOS Develop",
                           "technologyRequest": ["skil 1": "本科或以上学历，计算机或相关专业毕业，5-10年iOS开发经验；", "skil 2": "熟悉 C/C++，Objective-C，swift；", "skil 3": "熟练使用Xcode，Instruments等工具；", "skil 4": "熟悉主流开发框架和架构模式；", "skil 5": "熟悉Flutter、RN等主流跨平台方案者优先；"]]]
            
        ]
        
        FileHandle.writToFile(fileName: "JobRequest.json", array)

        /*
        // 写 json 方式一：
        let os = OutputStream(toFileAtPath: "/Users/harleyhuang/Desktop/main.json",
                              append: false)
        os?.open()
        JSONSerialization.writeJSONObject(array,
                                          to: os!,
                                          options: JSONSerialization.WritingOptions.prettyPrinted,
                                          error: NSErrorPointer.none)
        os?.close()
        
        // 写 json 方式二：
        let data = try! JSONSerialization.data(withJSONObject: array,
                                               options: JSONSerialization.WritingOptions.prettyPrinted)
        let url = URL(fileURLWithPath: "/Users/willokyes/Desktop/main.json")
        try! data.write(to: url, options: .atomic)

        */

    }
```
![文件展示](https://upload-images.jianshu.io/upload_images/2959789-480264181e4541c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>
****
<br/>
**`参考资料`**
[NSUserDefaults 文件的查看](https://blog.csdn.net/monkey_cool/article/details/40891967)

[iOS 数据的存储之 Property List (属性列表) 、NSUserDefaults (偏好设置) 、NSKeyedArchiver (归档)【总结 一】](https://www.jianshu.com/p/785028f0e193)
[沙盒路径及其文件夹用法](http://www.cocoachina.com/articles/21461)
[文件夹操作大全](https://www.hangge.com/blog/cache/detail_527.html)
[本地数据存取](https://blog.csdn.net/u013712343/article/details/80886290)
[本地存储](https://www.jianshu.com/p/9e5afdad0511)
