># @_exported
#`实现PCH的功能`

```
@_exported用于在自己模块中导出其他模块。

比如，自己定义了一个 A 模块。
在 A 模块中定义：

@_exported import UIKit

当我们引入 A 模块（import A）时，可以使用 UIKit 中的符号。

````
