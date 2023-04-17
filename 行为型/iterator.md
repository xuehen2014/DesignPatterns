# 迭代器模式

#### 什么是迭代器模式
迭代器模式（Iterator Design Pattern），也叫作游标模式（Cursor Design Pattern）。提供了一种方法顺序地访问一个聚合对象中的元素，而不是暴露该对象的内部表示

```gotemplate
   package main
   
   import "fmt"
   
   type Collection interface {
   	CreateIterator() Iterator
   }
   
   type Iterator interface {
   	HasNext() bool
   	Next()
   	CurrentItem() interface{}
   }
   
   type User struct {
   	name string
   	age  int
   }
   
   type UserCollection struct {
   	Users []*User
   }
   
   func (u *UserCollection) CreateIterator() Iterator {
   	return &UserIterator{
   		users: u.Users,
   		index: 0,
   	}
   }
   
   type UserIterator struct {
   	index int
   	users []*User
   }
   
   func (ui *UserIterator) HasNext() bool {
   	return ui.index < len(ui.users)
   }
   
   func (ui *UserIterator) Next() {
   	ui.index++
   }
   
   func (ui *UserIterator) CurrentItem() interface{} {
   	return ui.users[ui.index]
   }
   
   func main() {
   	userK := &User{
   		name: "Kevin",
   		age:  30,
   	}
   	userD := &User{
   		name: "Diamond",
   		age:  25,
   	}
   
   	userCollection := &UserCollection{
   		Users: []*User{userK, userD},
   	}
   
   	iterator := userCollection.CreateIterator()
   	for iterator.HasNext() {
   		user := iterator.CurrentItem()
   		fmt.Printf("User is %v\n", user)
   		iterator.Next()
   	}
   }

```

