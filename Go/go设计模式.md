> <h1 id=""></h1>
- [**工厂模式**](#工厂模式)




<br/><br/><br/>

***
<br/>

> <h1 id="工厂模式">工厂模式</h1>

**说明：**

Golang中的结构体没有**构造函数**，通常可以使用工厂模式来解决问题。

```go
pacage model

type Student struct {
	Name string...
}
```

因为Student的S是大写可以暴露给其他包使用，但是若是`type student struct{}`,该如何做呢？这个时候可以使用**工厂模式了**。

**如下：**

**model文件下有：**

```go
pacage model

type student struct {
	Name string
	score float64
}


// 因为student结构体首字母是小写，因此是只能在model使用
// 可以通过工厂模式来解决

func NewStudent(n string, s float64) *student {
	return &student {
		Name: n,
		score: s,
	}
}

// 如果score也是小写，我们可以提供一个get方法
func (s *student) getScore() {
	return s.score //ok
}
```

<br/>

**main.go文件：**

```go
import "go_code/factory/model"

func main() {

	// 定义studnet结构体是小写字母，我们可以通过工厂模式来解决
	var stu  = model.NewStudent("tom~", 88.8)
	
	fmt.Println(*stu) //&{...}
	fmt.Println("name=", stu.Name, "score=", &stu.getScore() cxxdcxZ)
}
```









