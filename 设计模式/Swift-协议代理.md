># Swift 实现可选协议方法
```
protocol  MiningHeaderViewDelegate {
    //可选协议的实现
    func setUpContent(_ titleArray: [String]?, completeHandler:@escaping (_ index: Int) -> Void)
    
    //必选协议实现
    func setTitles(_ titleArray: [String])
}

extension MiningHeaderViewDelegate {
    
    func setUpContent(_ titleArray: [String]?, completeHandler:@escaping (_ index: Int) -> Void) {}
}
```
