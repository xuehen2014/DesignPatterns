# 策略模式

#### Tips
模板、策略和职责链三个设计模式是解决业务系统流程复杂多变这个痛点的利器

#### 什么是策略模式    
定义一类算法族，将每个算法分别封装起来，让他们可以互相替换，此模式让算法的变化独立于使用算法的客户端
策略模式要解决的问题是，让使用客户端跟具体执行任务的策略解耦，不管使用哪种策略完成任务，不需要更改客户端使用策略的方式

```gotemplate
   package main
   
   import (
   	"fmt"
   	"io/ioutil"
   	"os"
   )
   
   // StorageStrategy 存储策略
   type StorageStrategy interface {
   	Save(name string, data []byte) error
   }
   
   var strategys = map[string]StorageStrategy{
   	"file":         &fileStorage{},
   	"encrypt_file": &encryptFileStorage{},
   }
   
   // NewStorageStrategy NewStorageStrategy
   func NewStorageStrategy(t string) (StorageStrategy, error) {
   	s, ok := strategys[t]
   	if !ok {
   		return nil, fmt.Errorf("not found StorageStrategy: %s", t)
   	}
   
   	return s, nil
   }
   
   // FileStorage 保存到文件
   type fileStorage struct{}
   
   // Save Save
   func (s *fileStorage) Save(name string, data []byte) error {
   	return ioutil.WriteFile(name, data, os.ModeAppend)
   }
   
   // encryptFileStorage 加密保存到文件
   type encryptFileStorage struct{}
   
   // Save Save
   func (s *encryptFileStorage) Save(name string, data []byte) error {
   	// 加密
   	data, err := encrypt(data)
   	if err != nil {
   		return err
   	}
   
   	return ioutil.WriteFile(name, data, os.ModeAppend)
   }
   
   func encrypt(data []byte) ([]byte, error) {
   	// 这里实现加密算法
   	return data, nil
   }
   
   func main() {
     data, sensitive := getData()
     strategyType := "file"
     if sensitive {
     	strategyType = "encrypt_file"
     }
     
     storage, err := NewStorageStrategy(strategyType)
     storage.Save("./test.txt", data)
   }
   
   // getData 获取数据的方法
   // 返回数据，以及数据是否敏感
   func getData() ([]byte, bool) {
   	return []byte("test data"), false
   }
```

#### 其它实现
```gotemplate
   package main
   
   import "fmt"
   
   type PayBehavior interface {
   	OrderPay(px *PayCtx)
   }
   
   // 具体支付策略实现
   // 微信支付
   type WxPay struct {}
   func(*WxPay) OrderPay(px *PayCtx) {
   	fmt.Printf("Wx支付加工支付请求 %v\n", px.payParams)
   	fmt.Println("正在使用Wx支付进行支付")
   }
   
   // 三方支付
   type ThirdPay struct {}
   func(*ThirdPay) OrderPay(px *PayCtx) {
   	fmt.Printf("三方支付加工支付请求 %v\n", px.payParams)
   	fmt.Println("正在使用三方支付进行支付")
   }
   
   type PayCtx struct {
   	// 提供支付能力的接口实现
   	payBehavior PayBehavior
   	// 支付参数
   	payParams map[string]interface{}
   }
   
   func (px *PayCtx) setPayBehavior(p PayBehavior) {
   	px.payBehavior = p
   }
   
   func (px *PayCtx) Pay() {
   	px.payBehavior.OrderPay(px)
   }
   
   func NewPayCtx(p PayBehavior) *PayCtx {
   	// 支付参数，Mock数据
   	params := map[string]interface{} {
   		"appId": "234fdfdngj4",
   		"mchId": 123456,
   	}
   	return &PayCtx{
   		payBehavior: p,
   		payParams: params,
   	}
   }
   
   func main() {
   	wxPay := &WxPay{}
   	px := NewPayCtx(wxPay)
   	px.Pay()
   	// 假设现在发现微信支付没钱，改用三方支付进行支付
   	thPay := &ThirdPay{}
   	px.setPayBehavior(thPay)
   	px.Pay()
   }
```

