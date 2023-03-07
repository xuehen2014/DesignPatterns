#代理模式

#### 什么是代理模式
代理模式是一种结构型设计模式。 其中代理控制着对于原对象的访问， 并允许在将请求提交给原对象的前后进行一些处理，从而增强原对象的逻辑处理。

上面的代理者我们一般叫做代理对象或者直接叫做代理-- Proxy，进行逻辑处理的原对象通常被称作服务对象，代理要跟服务对象实现相同的接口，才能让客户端傻傻分不清自己使用的到底是代理还是真正的服务对象，这样一来代理就能在客户端察觉不到的情况下对服务对象的处理逻辑进行增强。

什么叫对处理逻辑进行增强？或者换一种说法，叫对核心功能添加增强功能？举个例子来说，处理客户端查询用户订单信息的 API Handler 就是核心处理逻辑，增强逻辑就是我们需要在查询订单信息之前，验证请求是否是有效用户、记录请求的参数和返回的响应数据等等  

在代理模式中，通过让代理类实现跟服务类相同的接口，从而把代理类伪装成了服务类，客户端请求代理时，代理再把请求委派给其持有的真实服务类，在委派的过程中我们就可以添加增强逻辑。
***
相关链接：   
- 代理模式 [资料1](https://mp.weixin.qq.com/s?__biz=MzUzNTY5MzU2MA==&mid=2247497211&idx=1&sn=81cd8898a26d2c54d7dee6d8cb478bb0&chksm=fa83246ccdf4ad7abe6e4f30c3a92aaa734d0ab1ac5431b05712a947ad89834729922db76039&scene=178&cur_album_id=2531498848431669249#rd)
- 代理模式 [资料2](https://lailin.xyz/post/proxy.html)

*** 

```gotemplate
    //交通工具接口, 有一个Drive()方法
    type Vehicle interface {
        Drive()
    }
    
    //具体的交通工具Car, 实现了Vehicle接口
    type Car struct{}
    
    func (c *Car) Drive() {
        fmt.Println("Car is being driven")
    }
    
    type Driver struct {
        Age int
    }
    
    //代理-同样也实现了Vehicle接口
    type CarProxy struct {
        vehicle    Vehicle
        driver *Driver
    }
    
    func NewCarProxy(driver *Driver) *CarProxy {
        return &CarProxy{&Car{}, driver}
    }
    
    func (c *CarProxy) Drive() {
        if c.driver.Age >= 16 {
            c.vehicle.Drive()
        } else {
            fmt.Println("Driver too young!")
        }
    }
    
    func main() {
        car := NewCarProxy(&Driver{12})
        car.Drive()
        car2 := NewCarProxy(&Driver{22})
        car2.Drive()
    }

```

##### 其它实现

```gotemplate
    // IUser IUser
    type IUser interface {
        Login(username, password string) error
    }
    
    // User 用户
    type User struct {
    }
    
    // Login 用户登录
    func (u *User) Login(username, password string) error {
        // 不实现细节
        return nil
    }
    
    // UserProxy 代理类
    type UserProxy struct {
        user *User
    }
    
    // NewUserProxy NewUserProxy
    func NewUserProxy(user *User) *UserProxy {
        return &UserProxy{
            user: user,
        }
    }
    
    // Login 登录，和 user 实现相同的接口
    func (p *UserProxy) Login(username, password string) error {
        // before 这里可能会有一些统计的逻辑
        start := time.Now()
        // 这里是原有的业务逻辑
        if err := p.user.Login(username, password); err != nil {
            return err
        }
        // after 这里可能也有一些监控统计的逻辑
        log.Printf("user login cost time: %s", time.Now().Sub(start))
        return nil
    }

```
