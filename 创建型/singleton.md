# 单例模式

---
单例模式，也叫单子模式，是一种常用的软件设计模式。 
在应用这个模式时，单例对象的类必须保证只有一个实例存在。许多时候整个系统只需要拥有一个的全局对象，这样有利于我们协调系统整体的行为。比如在某个服务器程序中，该服务器的配置信息存放在一个文件中，这些配置数据由一个单例对象统一读取，然后服务进程中的其他对象再通过这个单例对象获取这些配置信息。这种方式简化了在复杂环境下的配置管理

***

**注意：**

1、单例类只能有一个实例  
2、单例类必须自己创建自己的唯一实例  
3、单例类必须给所有其他对象提供这一实例

***

1.懒汉式  
   懒汉模式是开源项目中使用最多的一种，最大的缺点是非线程安全的

```gotemplate
   type singleton struct {
   }
   
   // private
   var instance *singleton
   
   // public
   func GetInstance() *singleton {
       if instance == nil {
           instance = &singleton{}     // not thread safe
       }
       return instance
   }
```

2.懒汉式（加锁）   
   不论对象是否创建，获取对象都需要加锁

```gotemplate
   type singleton struct {
   }
   
   var instance *singleton
   var mu sync.Mutex
   
   func GetInstance() *singleton {
       mu.Lock()
       defer mu.Unlock()
   
       if instance == nil {
           instance = &singleton{}     // unnecessary locking if instance already created
       }
       return instance
   }
```

3.懒汉式（双重检测+加锁）  
这是一个不错的方法，但是还并不是很完美。因为编译器优化没有检查实例存储状态。如果使用sync/atomic包的话 就可以自动帮我们加载和设置标记

```gotemplate
   type singleton struct {
   }
   
   var instance *singleton
   var mu sync.Mutex
   
   func GetInstance() *singleton {
      if instance == nil {     // <-- Not yet perfect. since it's not fully atomic
       mu.Lock()
       defer mu.Unlock()
   
       if instance == nil {
           instance = &singleton{}     // unnecessary locking if instance already created
       }
      }
       return instance
   }
```

4.使用原子方式检测

```gotemplate
   import "sync"
   import "sync/atomic"
   
   var initialized uint32
   var instance *singleton
   
   func GetInstance() *singleton {
       if atomic.LoadUInt32(&initialized) == 1 {
           return instance
       }
       mu.Lock()
       defer mu.Unlock()
   
       if initialized == 0 {
            instance = &singleton{}
            atomic.StoreUint32(&initialized, 1)
       }
   
       return instance
   }
```

5.Golang的方式

```gotemplate
   import (
       "sync"
   )
    
   type singleton struct {
   }
    
   var instance *singleton
   var once sync.Once
    
   func GetInstance() *singleton {
       once.Do(func() {
           instance = &singleton{}
       })
       return instance
   }
```





