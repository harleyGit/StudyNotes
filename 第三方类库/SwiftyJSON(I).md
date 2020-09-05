`public var arrayObject: [Any]?`


```
let data = [T].deserialize(from: json["data"].arrayObject)
//arrayObject 如何使用？
//JSON转化为Array数组（[AnyObject]?）


let data = T.deserialize(from: json["data"].dictionaryObject)
//dictionaryObject 如何使用？
//JSON转化为Dictionary字典（[String: AnyObject]?）

```


<br/>
***
<br/>
[SwiftyJSON的使用详解](https://www.hangge.com/blog/cache/detail_968.html)
