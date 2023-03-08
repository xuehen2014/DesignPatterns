# 观察者模式

#### 什么是观察者模式   
观察者模式也经常被叫做发布 - 订阅（Publish/Subscribe）模式 

观察者模式 (Observer Pattern)，定义对象间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知，依赖对象在收到通知后，可自行调用自身的处理程序，实现想要干的事情，比如更新自己的状态。

发布者对观察者唯一了解的是它实现了某个接口（观察者接口）。这种松散耦合的设计最大限度地减少了对象之间的相互依赖，因此使我们能够构建灵活的系统

***

```gotemplate
   package observer
   
   import "fmt"
   
   // ISubject subject--相当于是发布者的定义
   type ISubject interface {
   	Register(observer IObserver)
   	Remove(observer IObserver)
   	Notify(msg string)
   }
   
   // IObserver 观察者
   type IObserver interface {
   	Update(msg string)
   }
   
   // Subject Subject
   type Subject struct {
   	observers []IObserver
   }
   
   // Register 注册
   func (sub *Subject) Register(observer IObserver) {
   	sub.observers = append(sub.observers, observer)
   }
   
   // Remove 移除观察者
   func (sub *Subject) Remove(observer IObserver) {
   	for i, ob := range sub.observers {
   		if ob == observer {
   			sub.observers = append(sub.observers[:i], sub.observers[i+1:]...)
   		}
   	}
   }
   
   // Notify 通知
   func (sub *Subject) Notify(msg string) {
   	for _, o := range sub.observers {
   		o.Update(msg)
   	}
   }
   
   // Observer1 Observer1
   type Observer1 struct{}
   
   // Update 实现观察者接口
   func (Observer1) Update(msg string) {
   	fmt.Printf("Observer1: %s", msg)
   }
   
   // Observer2 Observer2
   type Observer2 struct{}
   
   // Update 实现观察者接口
   func (Observer2) Update(msg string) {
   	fmt.Printf("Observer2: %s", msg)
   }
   
   func main() {
        sub := &Subject{}
        sub.Register(&Observer1{})
        sub.Register(&Observer2{})
        sub.Notify("hi")
   }
```