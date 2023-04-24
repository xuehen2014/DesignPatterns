# 模板模式

#### 什么是模板模式    
模板模式在一个方法中定义一个算法骨架，并将某些步骤推迟到子类中实现   
模板模式可以在不改变算法整体结构的情况下，重新定义算法中的某些步骤   
这里的算法可以理解为业务逻辑

- 代码实现 (1)
```gotemplate
  package main
  
  import (
  	"fmt"
  	"math/rand"
  	"strconv"
  	"time"
  )
  
  type BankBusinessHandler interface {
  	// 排队拿号
  	TakeRowNumber()
  	// 等位
  	WaitInHead()
  	// 处理具体业务
  	HandleBusiness()
  	// 对服务作出评价
  	Commentate()
  	// 钩子方法，判断是不是VIP， VIP不用等位
  	CheckVipIdentity() bool
  }
  
  //实现 BankBusinessHandler 接口 --- 类似一个抽象类
  type DefaultBusinessHandler struct {
  }
  
  func (dbh *DefaultBusinessHandler) TakeRowNumber() {
  	fmt.Println("请拿好您的取件码：" + strconv.Itoa(rand.Intn(100)) + " ，注意排队情况，过号后顺延三个安排")
  }
  
  func (dbh *DefaultBusinessHandler) WaitInHead() {
  	fmt.Println("排队等号中...")
  	time.Sleep(5 * time.Second)
  	fmt.Println("请去窗口xxx...")
  }
  
  func (dbh *DefaultBusinessHandler) HandleBusiness() {
  	//留给具体实现类
  }
  
  func (dbh *DefaultBusinessHandler) Commentate() {
  	fmt.Println("请对我的服务作出评价，满意请按0，满意请按0，(～￣▽￣)～")
  }
  
  func (dbh *DefaultBusinessHandler) CheckVipIdentity() bool {
  	// 留给具体实现类实现
  	return false
  }
  
  type BankBusinessExecutor struct {
  	handler BankBusinessHandler
  }
  
  // 模板方法，处理银行业务 --封装下具体的业务实现步骤
  func (b *BankBusinessExecutor) ExecuteBankBusiness() {
  	// 适用于与客户端单次交互的流程
  	// 如果需要与客户端多次交互才能完成整个流程，每次交互的操作去对应模板里定义的方法就好，并不需要一个调用所有方法的模板方法
  	b.handler.TakeRowNumber()
  	if !b.handler.CheckVipIdentity() {
  		b.handler.WaitInHead()
  	}
  	b.handler.HandleBusiness()
  	b.handler.Commentate()
  }
  
  func NewBankBusinessExecutor(businessHandler BankBusinessHandler) *BankBusinessExecutor {
  	return &BankBusinessExecutor{handler: businessHandler}
  }
  
  //存款业务
  type DepositBusinessHandler struct {
  	*DefaultBusinessHandler
  	userVip bool
  }
  
  func (dh *DepositBusinessHandler) HandleBusiness() {
  	fmt.Println("账户存储很多万人民币...")
  }
  
  func (dh *DepositBusinessHandler) CheckVipIdentity() bool {
  	return dh.userVip
  }
  
  //取款业务
  type WithdrawalBusinessHandler struct {
  	*DefaultBusinessHandler
  	WithdrawalAmount int //取款金额
  }
  
  func (wh *WithdrawalBusinessHandler) HandleBusiness() {
  	fmt.Println("账户取款取款取款很多万人民币...")
  }
  
  func (wh *WithdrawalBusinessHandler) CheckVipIdentity() bool {
  	if wh.WithdrawalAmount > 1000 {
  		return true
  	}
  	return false
  }
  
  func main() {
  	dh := &DepositBusinessHandler{userVip: false}
  	bbe := NewBankBusinessExecutor(dh)
  	bbe.ExecuteBankBusiness()
  
  	wh := &WithdrawalBusinessHandler{WithdrawalAmount: 2000}
  	aaq := NewBankBusinessExecutor(wh)
  	aaq.ExecuteBankBusiness()
  }

```

- 代码实现（2）

```gotemplate
   package template
   
   import "fmt"
   
   type Cooker interface {
       open()
       fire()
       cooke()
       outfire()
       close()
   }
   
   // 类似于一个抽象类
   type CookMenu struct {
   }
   
   func (CookMenu) open() {
       fmt.Println("打开开关")
   }
   
   func (CookMenu) fire() {
       fmt.Println("开火")
   }
   
   // 做菜，交给具体的子类实现
   func (CookMenu) cooke() {
   }
   
   func (CookMenu) outfire() {
       fmt.Println("关火")
   }
   
   func (CookMenu) close() {
       fmt.Println("关闭开关")
   }
   
   // 封装具体步骤
   func doCook(cook Cooker) {
       cook.open()
       cook.fire()
       cook.cooke()
       cook.outfire()
       cook.close()
   }
   
   type XiHongShi struct {
       CookMenu
   }
   
   func (*XiHongShi) cooke() {
       fmt.Println("做西红柿")
   }
   
   type ChaoJiDan struct {
       CookMenu
   }
   
   func (ChaoJiDan) cooke() {
       fmt.Println("做炒鸡蛋")
   }
   
   func main() {
       // 做西红柿
       xihongshi := &XiHongShi{}
       doCook(xihongshi)
       
       // 做炒鸡蛋
       chaojidan := &ChaoJiDan{}
       doCook(chaojidan)
   }

```