# 建造者模式

造者模式（Builder Pattern）使用多个简单的对象一步一步构建成一个复杂的对象。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

一个 Builder 类会一步一步构造最终的对象。该 Builder 类是独立于其他对象的。

意图：将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。

主要解决：主要解决在软件系统中，有时候面临着"一个复杂对象"的创建工作，其通常由各个部分的子对象用一定的算法构成；由于需求的变化，这个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的算法却相对稳定。

何时使用：一些基本部件不会变，而其组合经常变化的时候。


```gotemplate
   type DBPool struct {
   	dsn             string
   	maxOpenConn     int
   	maxIdleConn     int
   	maxConnLifeTime time.Duration
   }
   
   type DBPoolBuilder struct {
   	DBPool
   	err error
   }
   
   func Builder () *DBPoolBuilder {
   	b := new(DBPoolBuilder)
   	// 设置 DBPool 属性的默认值
   	b.DBPool.dsn = "127.0.0.1:3306"
   	b.DBPool.maxConnLifeTime = 1 * time.Second
   	b.DBPool.maxOpenConn = 30
   	return b
   }
   
   func (b *DBPoolBuilder) DSN(dsn string) *DBPoolBuilder {
   	if b.err != nil {
   		return b
   	}
   	if dsn == "" {
   		b.err = fmt.Errorf("invalid dsn, current is %s", dsn)
   	}
   
   	b.DBPool.dsn = dsn
   	return b
   }
   
   func (b *DBPoolBuilder) MaxOpenConn(connNum int) *DBPoolBuilder {
   	if b.err != nil {
   		return b
   	}
   	if connNum < 1 {
   		b.err = fmt.Errorf("invalid MaxOpenConn, current is %d", connNum)
   	}
   
   	b.DBPool.maxOpenConn = connNum
   	return b
   }
   
   func (b *DBPoolBuilder) MaxConnLifeTime(lifeTime time.Duration) *DBPoolBuilder {
   	if b.err != nil {
   		return b
   	}
   	if lifeTime < 1  * time.Second {
   		b.err = fmt.Errorf("connection max life time can not litte than 1 second, current is %v", lifeTime)
   	}
   
   	b.DBPool.maxConnLifeTime = lifeTime
   	return b
   }
   
   func (b *DBPoolBuilder) Build() (*DBPool, error) {
   	if b.err != nil {
   		return nil, b.err
   	}
   	if b.DBPool.maxOpenConn < b.DBPool.maxIdleConn {
   		return nil, fmt.Errorf("max total(%d) cannot < max idle(%d)", b.DBPool.maxOpenConn, b.DBPool.maxIdleConn)
   	}
   	return &b.DBPool, nil
   }
   
   func main() {
   	dbPool, err := Builder().DSN("localhost:3306").MaxOpenConn(50).MaxConnLifeTime(10 * time.Second).Build()
   	if err != nil {
   		fmt.Println(err)
   	}
   	fmt.Println(dbPool)
   }
```