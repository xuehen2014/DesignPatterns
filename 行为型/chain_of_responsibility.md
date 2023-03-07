# 职责链模式

#### 什么是职责链模式   
顾名思义，责任链模式为请求创建了一个接受者对象的链，这种模式给予请求的类型，对请求的发送者和接受者进行解偶。这种类型的设计模式属于行为模式。

在这种模式中，通常每个接受者都包含对另一个接受者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传递给下一个接收者，以此类推

意图：避免请求发送者与接受者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止。

主要解决：职责链上的处理者负责处理请求，客户只需要将请求发送到职责链上即可，无须关心请求的处理细节和请求的传递，所以职责链将请求的发送者和请求的处理者解偶了。

何时使用：在处理消息的时候以过滤很多道。

如何解决：拦截的类都实现统一接口

***

* 具体实现1
```gotemplate
   type PatientHandler interface {
   	Execute(*patient) error
   	SetNext(PatientHandler) PatientHandler
   	Do(*patient) error
   }
   
   type Next struct {
   	nextHandler PatientHandler
   }
   
   func (n *Next) SetNext(handler PatientHandler) PatientHandler {
   	n.nextHandler = handler
   	return handler
   }
   
   func (n *Next) Execute(patient *patient) (err error) {
   	// 由于go无继承的概念, 只能用组合，组合跟继承不一样，这里如果Next 实现了 Do 方法，那么匿名组合它的具体处理类型，执行Execute的时候，调用的还是内部Next对象的Do方法
   	// 调用不到外部类型的 Do 方法，所以 Next 不能实现 Do 方法
   	if n.nextHandler != nil {
   		if err = n.nextHandler.Do(patient); err != nil {
   			return
   		}
   
   		return n.nextHandler.Execute(patient)
   	}
   
   	return
   }
   
   // Pharmacy 药房处理器
   type Pharmacy struct {
   	Next
   }
   
   
   func (m *Pharmacy) Do (p *patient) (err error) {
   	if p.MedicineDone {
   		fmt.Println("Medicine already given to patient")
   		return
   	}
   	fmt.Println("Pharmacy giving medicine to patient")
   	p.MedicineDone = true
   	return
   }
   
   // Cashier 收费处处理器
   type Cashier struct {
   	Next
   }
   
   func (c *Cashier) Do(p *patient) (err error) {
   	if p.PaymentDone {
   		fmt.Println("Payment Done")
   		return
   	}
   	fmt.Println("Cashier getting money from patient patient")
   	p.PaymentDone = true
   	return
   }
   
   // Clinic 诊室处理器--用于医生给病人看病
   type Clinic struct {
   	Next
   }
   
   func (d *Clinic) Do(p *patient) (err error) {
   	if p.DoctorCheckUpDone {
   		fmt.Println("Doctor checkup already done")
   		return
   	}
   	fmt.Println("Doctor checking patient")
   	p.DoctorCheckUpDone = true
   	return
   }
   
   // Reception 挂号处处理器
   type Reception struct {
   	Next
   }
   
   func (r *Reception) Do(p *patient) (err error) {
   	if p.RegistrationDone {
   		fmt.Println("Patient registration already done")
   		return
   	}
   	fmt.Println("Reception registering patient")
   	p.RegistrationDone = true
   	return
   }
   
   // StartHandler 不做操作，作为第一个Handler向下转发请求
   // Go 语法限制，抽象公共逻辑到通用Handler后，并不能跟继承一样让公共方法调用不通子类的实现
   type StartHandler struct {
   	Next
   }
   
   // Do 空Handler的Do
   func (h *StartHandler) Do(c *patient) (err error) {
   	// 空Handler 这里什么也不做 只是载体 do nothing...
   	return
   }
   
   type patient struct {
   	Name              string
   	RegistrationDone  bool
   	DoctorCheckUpDone bool
   	MedicineDone      bool
   	PaymentDone       bool
   }
   func main() {
   	patientHealthHandler := StartHandler{}
   	//
   	patient := &patient{Name: "abc"}
   	// 设置病人看病的链路
   	patientHealthHandler.SetNext(&Reception{}).// 挂号
   		SetNext(&Clinic{}). // 诊室看病
   		SetNext(&Cashier{}). // 收费处交钱
   		SetNext(&Pharmacy{}) // 药房拿药
   	// 还可以无效扩展，比如中间加入化验科化验，图像科拍片等等
   
   	// 执行上面设置好的业务流程
   	if err := patientHealthHandler.Execute(patient); err != nil {
   		// 异常
   		fmt.Println("Fail | Error:" + err.Error())
   		return
   	}
   	// 成功
   	fmt.Println("Success")
   }
```

* 具体实现2

```gotemplate
   // Package chain 职责链模式
   // 🌰 假设我们现在有个校园论坛，由于社区规章制度、广告、法律法规的原因需要对用户的发言进行敏感词过滤
   //    如果被判定为敏感词，那么这篇帖子将会被封禁
   package main
   
   import (
   	"fmt"
   )
   
   // SensitiveWordFilter 敏感词过滤器，判定是否是敏感词
   type SensitiveWordFilter interface {
   	Filter(content string) bool
   }
   
   // SensitiveWordFilterChain 职责链
   type SensitiveWordFilterChain struct {
   	filters []SensitiveWordFilter
   }
   
   // AddFilter 添加一个过滤器
   func (c *SensitiveWordFilterChain) AddFilter(filter SensitiveWordFilter) {
   	c.filters = append(c.filters, filter)
   }
   
   // Filter 执行过滤
   func (c *SensitiveWordFilterChain) Filter(content string) bool {
   	for _, filter := range c.filters {
   		// 如果发现敏感直接返回结果
   		if filter.Filter(content) {
   			return true
   		}
   	}
   	return false
   }
   
   // AdSensitiveWordFilter 广告
   type AdSensitiveWordFilter struct{}
   
   // Filter 实现过滤算法
   func (f *AdSensitiveWordFilter) Filter(content string) bool {
   	// TODO: 实现算法
   	return false
   }
   
   // PoliticalWordFilter 政治敏感
   type PoliticalWordFilter struct{}
   
   // Filter 实现过滤算法
   func (f *PoliticalWordFilter) Filter(content string) bool {
   	// TODO: 实现算法
   	return true
   }
   
   func main() {
   	chain := &SensitiveWordFilterChain{}
   	chain.AddFilter(&AdSensitiveWordFilter{})
   	chain.AddFilter(&PoliticalWordFilter{})
   	fmt.Println(chain.Filter("test"))
   }
```