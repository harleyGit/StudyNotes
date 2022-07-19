


> <h2 id=''></h2>
- [**UIView 响应**](#UIView响应)
- [参考资料](#参考资料)
	- [ReactiveObjc(I)](https://www.jianshu.com/p/2e07b07e5132)
	- [ReactiveCocoa](https://www.jianshu.com/p/0845b1a07bfa)


<br/>

***
<br/>

<h2 id='UIView响应'>UIView 响应</h2>

```

//RAC(_label, text) ,通过RAC的宏来使用，更简化
RAC(_label, text) = _textField.rac_textSignal

//RACObserve 也是一个宏
RACObserve(self.view, frame) subscribeNext:^(id x){
    NSLog(@"");
}

```

 **`ViewController.m`**

```

- （void）viewDidLoad{
      //self 调用者
      //_cmd 方法编号
      @weakify(self);  //使用这个宏，打断循环引用链条
      RACSignal *signal = [RACSignal createSignal: ^RACDisoable * _Nullable (id <RACSubscriber> _Nonull subscriber){
            @strongify(self);
           //这个时候就会发生循环引用，这个block属于Signal，而这个Signal又属于这个控制器ViewController的强引用的。
           //而这个block又做了对self的访问，它会给这个self加个strong，这就是循环引用造成的内存泄露了(堆内存泄露了)
           NSLog(@"%@", self);
           return nill;

}];
     //signal 是在栈中
      _signal = signal;
}

- (void)dealloc{
      NSLog(@"%s", __func__);
}


- (void) dismiss:(id) sender {
      [self dismissViewController:YES completion: nil];
}


```




<br/>

***
<br/>



<br/>
