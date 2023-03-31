# 原型模式

#### 什么是原型模式    
通过复制、拷贝或者叫克隆已有对象的方式来创建新对象的设计模式叫做原型模式，被拷贝的对象也被称作原型对象。

原型对象按照惯例，会暴露出一个 Clone 方法，给外部调用者一个机会来从自己这里“零成本”的克隆出一个新对象。

这里的“零成本”说的是，调用者啥都不用干，干等着，原型对象在 Clone 方法里自己克隆出自己，给到调用者，所以按照这个约定所有原型对象都要实现一个 Clone 方法

```gotemplate
   // Keyword 搜索关键字
   type Keyword struct {
   	word      string
   	visit     int
   	UpdatedAt *time.Time
   }
   
   // Clone 这里使用序列化与反序列化的方式深拷贝
   func (k *Keyword) Clone() *Keyword {
   	var newKeyword Keyword
   	b, _ := json.Marshal(k)
   	json.Unmarshal(b, &newKeyword)
   	return &newKeyword
   }
   
   // Keywords 关键字 map
   type Keywords map[string]*Keyword
   
   // Clone 复制一个新的 keywords
   // updatedWords: 需要更新的关键词列表，由于从数据库中获取数据常常是数组的方式
   func (words Keywords) Clone(updatedWords []*Keyword) Keywords {
   	newKeywords := Keywords{}
   
   	for k, v := range words {
   		// 这里是浅拷贝，直接拷贝了地址
   		newKeywords[k] = v
   	}
   
   	// 替换掉需要更新的字段，这里用的是深拷贝
   	for _, word := range updatedWords {
   		newKeywords[word.word] = word.Clone()
   	}
   
   	return newKeywords
   }

```