# Option选项模式

#### 什么是Option选项模式  

Option模式的专业术语为：Functional Options Pattern（函数式选项模式）
Option模式为golang的开发者提供了将一个函数的参数设置为可选的功能，也就是说我们可以选择参数中的某几个，并且可以按任意顺序传入参数。
比如针对特殊场景需要不同参数的情况，C++可以直接用重载来写出任意个同名函数，在任意场景调用的时候使用同一个函数名即可；但同样情况下，在golang中我们就必须在不同的场景使用不同的函数，并且参数传递方式可能不同的人写出来是不同的样子，这将导致代码可读性差，维护性差

```gotemplate
   package main
   
   import "fmt"
   
   type Message struct {
   	id      int
   	name    string
   	address string
   	phone   int
   }
   
   func (msg Message) String() {
   	fmt.Printf("ID:%d \n- Name:%s \n- Address:%s \n- phone:%d\n", msg.id, msg.name, msg.address, msg.phone)
   }
   
   func New(id, phone int, name, addr string) Message {
   	return Message{
   		id:      id,
   		name:    name,
   		address: addr,
   		phone:   phone,
   	}
   }
   
   type Option func(msg *Message)
   
   var DEFAULT_MESSAGE = Message{id: -1, name: "-1", address: "-1", phone: -1}
   
   func WithID(id int) Option {
   	return func(m *Message) {
   		m.id = id
   	}
   }
   
   func WithName(name string) Option {
   	return func(m *Message) {
   		m.name = name
   	}
   }
   
   func WithAddress(addr string) Option {
   	return func(m *Message) {
   		m.address = addr
   	}
   }
   
   func WithPhone(phone int) Option {
   	return func(m *Message) {
   		m.phone = phone
   	}
   }
   
   func NewByOption(opts ...Option) Message {
   	msg := DEFAULT_MESSAGE
   	for _, o := range opts {
   		o(&msg)
   	}
   	return msg
   }
   
   func NewByOptionWithoutID(id int, opts ...Option) Message {
   	msg := DEFAULT_MESSAGE
   	msg.id = id
   	for _, o := range opts {
   		o(&msg)
   	}
   	return msg
   }
   
   func NewByOptionFunc(id int, optionFunc Option) Message {
     msg := DEFAULT_MESSAGE
     msg.id = id
     optionFunc(&msg)
     return msg
   }
   
   func main() {
   	message1 := New(1, 123, "message1", "cache1")
   	message1.String()
   	message2 := NewByOption(WithID(2), WithName("message2"), WithAddress("cache2"), WithPhone(456))
   	message2.String()
   	message3 := NewByOptionWithoutID(3, WithAddress("cache3"), WithPhone(789), WithName("message3"))
   	message3.String()
   	message :=  NewByOptionFunc(4, func(msg *Message){
   	    msg.name = "zhangsan"
   	    msg.address = "xxx"
   	}) 
   	message.String()
   }

```