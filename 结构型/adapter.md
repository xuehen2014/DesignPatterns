# 适配器模式

#### 什么是适配器模式   
适配器模式的英文翻译是 Adapter Design Pattern。顾名思义，这个模式就是用来做适配的，它将不兼容的接口转换为可兼容的接口，让原本由于接口不兼容而不能一起工作的类可以一起工作。对于这个模式，有一个经常被拿来解释它的例子，就是 USB 转接头充当适配器，把两种不兼容的接口，通过转接变得可以一起工作

```gotemplate
    type Target interface {
        RequiredMethod()
    }
    
    type Adaptee struct{}
    
    func (a *Adaptee) OldMethod() {
        fmt.Println("Adaptee.OldMethod()")
    }
    
    type Adapter struct {
        adaptee *Adaptee
    }
    
    func NewAdapter() *Adapter {
        return &Adapter{new(Adaptee)}
    }
    
    func (a *Adapter) RequiredMethod() {
        fmt.Println("Adapter.RequiredMethod()")
        a.adaptee.OldMethod()
    }
    
    func main() {
        target := NewAdapter()
        target.RequiredMethod()
    }
```

#### 适配器和代理模式的区别    
适配器模式和代理模式同属于结构型的设计模式，他们两个在类结构上也非常相似，都是由一个包装对象持有原对象，把客户端对包装对象的请求转发到原对象上。那么这两个模式有什么不同呢？我们怎么区分自己使用的是适配器还是代理模式？

适配器和代理模式的区别：

适配器与原对象（被适配对象）实现不同的接口，适配器的特点在于兼容，客户端通过适配器的接口完成跟自己不兼容的原对象的访问交互。
代理与原对象（被代理对象）实现相同的接口，代理模式的特点在于隔离和控制，代理直接转发原对象的返回给客户端，但是可以在调用原始对象接口的前后做一些额外的辅助工作，AOP编程的实现也是利用这个原理