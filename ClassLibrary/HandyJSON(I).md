**`JSON转model`**
```
//方法一
static func deserialize(from array: [Any]?) -> [Element?]?

// 反序列化
    fileprivate func test1() {

        let jsonString = "{\"age\":24,\"name\":\"Micheal\",\"sex\":\"男\"}"
        guard let model = TestModel.deserialize(from: jsonString) else {return}

        print(model.name)/// Micheal
        print(model.age!)/// 24
        print(model.sex!)/// 男
    }
```


<br/>
***
<br/>
># HandyJSONEnum
**`Model`**
```
enum AnimalType: String, HandyJSONEnum {
    case Cat = "Cat"
    
    case Dog = "Dog"
    
    case Bird = "Bird"
}

struct Animal: HandyJSON {
    var name: String?
    var type: AnimalType?
}
```

**`Controller`**
```

override func viewDidLoad() {
        super.viewDidLoad()
        
        //HandyJSON
        self.handyJSONTest()
}

func handyJSONTest() {
        let jsonString = "{\"type\":\"Cat\",\"name\":\"Tom\"}"
        
        if let animal = Animal.deserialize(from: jsonString) {
            print(animal.type?.rawValue)
        }
    }


```

效果图：
![控制台打印](https://upload-images.jianshu.io/upload_images/2959789-438468e5aeb94019.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





<br/>
***
<br/>


参考资料：
[HandyJSON使用详解 US](https://www.jianshu.com/p/e26e9d8dc86f)
[HandyJSON使用讲解](https://www.cnblogs.com/yajunLi/p/7121950.html)
