
- [**压缩HTTP请求体**](#压缩HTTP请求体)
- [**上传优化**](https://note.youdao.com/ynoteshare1/index.html?id=a6dd4f1ec0789fa563594ce7a7bfc5ac&type=note)
<br/>

***
<br/>

> <h1 id="压缩HTTP请求体">压缩HTTP请求体</h1>



[使用 zlib 库实现 HTTP 请求数据压缩](https://www.jianshu.com/p/dc76bcfef553)
如何压缩？有什么好处？服务端如何解压缩？
函数：`deflateInit2`


降低请求延迟，使用HTTP管道重用现有TCP连接，使HTTP客户端能够在对第一个请求的响应返回前在相同的TCP Socket 上发送第二个请求。
`NSMutableURLRequest *request =[ [NSMutableURLRequest alloc] initWithURL: url];`
`[request setHTTPShouldUsePipelining: YES];`
这个在现在的Session如何做，


避免网络请求：是要求检查返回数据是否是新的，若是新的返回新数据，若不是没有修改的数据返回 304 响应。
提供了多个控制响应缓存的方式：
```
NSURLRequestUseProtocolCachePolicy

NSURLRequestReloadIgnoringLocalCacheData

```


<br/>

***
<br/>

> <h1 id="">上传优化</h1>

[加载高清大图](https://juejin.cn/post/6844903540578582536#heading-9)
