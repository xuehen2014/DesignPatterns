# 装饰器模式

#### 什么是装饰器模式

装饰器模式（Decorator Pattern）也叫作包装器模式（Wrapper Pattern），指在不改变原有对象的基础上，动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活，属于结构型设计模式

使用装饰器模式，我们通过将现有对象放置在实现了相同一套接口的包装器对象中来动态地向现有对象添加新行为。在包装器中进行我们代码的扩展，有助于重用功能并且不会修改现有对象的代码，符合“开闭原则”

这里被放置在包装对象的“现有对象”通常会被叫做“组件”（Component），而包装组件的包装器对象就是我们常说的“装饰器”（Decorator），因为装饰器会组件实现相同接口，故客户端无法识别两者的差异，也就不需要在增加装饰器时对客户端调用代码进行修改了

```gotemplate
   package main
   
   import "fmt"
   
   // PS5 产品接口
   type PS5 interface {
        StartGPUEngine()
        GetPrice() int64
   }
   
   // CD 版 PS5主机
   type PS5WithCD struct{}
   
   func (p PS5WithCD) StartGPUEngine() {
   	    fmt.Println("start engine")
   }
   func (p PS5WithCD) GetPrice() int64 {
   	    return 5000
   }
   
   // PS5 数字版主机
   type PS5WithDigital struct{}
   
   func (p PS5WithDigital) StartGPUEngine() {
   	    fmt.Println("start normal gpu engine")
   }
   
   func (p PS5WithDigital) GetPrice() int64 {
   	    return 3600
   }
   
   //Plus版 PS5 主机
   type PS5MachinePlus struct {
   	    ps5Machine PS5
   }
   
   func (p *PS5MachinePlus) SetPS5Machine(ps5 PS5) {
   	    p.ps5Machine = ps5
   }
   
   func (p PS5MachinePlus) StartGPUEngine() {
        p.ps5Machine.StartGPUEngine()
        fmt.Println("start plus plugin")
   }
   
   func (p PS5MachinePlus) GetPrice() int64 {
   	    return p.ps5Machine.GetPrice() + 500
   }
   
   // 主题色版 PS5 主机
   type PS5WithTopicColor struct {
   	    ps5Machine PS5
   }
   
   func (p *PS5WithTopicColor) SetPS5Machine(ps5 PS5) {
   	    p.ps5Machine = ps5
   }
   
   func (p PS5WithTopicColor) StartGPUEngine() {
        p.ps5Machine.StartGPUEngine()
        fmt.Println("尊贵的主题色主机，GPU启动")
   }
   func (p PS5WithTopicColor) GetPrice() int64 {
   	    return p.ps5Machine.GetPrice() + 200
   }
   
   func main() {
        ps5MachinePlus := PS5MachinePlus{}
        ps5MachinePlus.SetPS5Machine(PS5WithCD{})
        // ps5MachinePlus.SetPS5Machine(PS5WithDigital{}) // 可以在更换主机
        ps5MachinePlus.StartGPUEngine()
        price := ps5MachinePlus.GetPrice()
        fmt.Printf("PS5 CD 豪华Plus版，价格: %d 元\n\n", price)
       
        ps5WithTopicColor := PS5WithTopicColor{}
        ps5WithTopicColor.SetPS5Machine(ps5MachinePlus)
        ps5WithTopicColor.StartGPUEngine()
        price = ps5WithTopicColor.GetPrice()
        fmt.Printf("PS5 CD 豪华Plus 经典主题配色版，价格: %d 元\n", price)
   }
```