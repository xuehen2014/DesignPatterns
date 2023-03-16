#工厂模式

设计模式里工厂模式一共可以提炼成三类工厂    
* 简单工厂   
* 工厂方法   
* 抽象工厂

---------- 
#### 简单工厂模式-simple-factory

简单工厂模式三要素

产品: 客户端需要的产品    
工厂: 生产产品的工厂     
客户端: 消费产品的客户

```gotemplate
   //以引例的汉堡工厂为例，产品即为汉堡，这个是一个抽象类型，根据产品类型的不同，我们分别定义两个具体产品，即肯德基的汉堡和麦当劳的汉堡
   type Hamburger interface {
       Deliver()
   }
   
   type KfcHamburger struct{}
   
   func (h KfcHamburger) Deliver() {
       fmt.Println("This is a hamburger from KFC.")
   }
   
   type McdonaldsHamburger struct{}
   
   func (h McdonaldsHamburger) Deliver() {
       fmt.Println("This is a hamburger from McDonalds.")
   }
   
   //汉堡工厂则根据需要生产指定类型的汉堡
   type HamburgerFactory struct{}
   
   func (f HamburgerFactory) CreateHamburger(prefer string) Hamburger {
       switch prefer {
       case "KFC":
           return new(KfcHamburger)
       case "McDonalds":
           return new(McdonaldsHamburger)
       default:
           return nil
       }
   }
   
   //客户端
   func main() {
       prefer := getPreferHamburger()
       factory := new(HamburgerFactory)
       hamburger := factory.CreateHamburger(prefer)
       if hamburger == nil {
           fmt.Printf("%s not supported yet.\n", prefer)
           os.Exit(1)
       }
   
       hamburger.Deliver()
   }
   
   //原文链接：https://blog.csdn.net/m0_37554486/article/details/77435677
```

#### 工厂方法模式

工厂方法模式（Factory Method Pattern）又叫作多态性工厂模式，指的是定义一个创建对象的接口，但由实现这个接口的工厂类来决定实例化哪个产品类，工厂方法把类的实例化推迟到子类中进行  

在工厂方法模式中，不再由单一的工厂类生产产品，而是由工厂类的子类实现具体产品的创建。因此，当增加一个产品时，只需增加一个相应的工厂类的子类, 以解决简单工厂生产太多产品时导致其内部代码臃肿（switch … case分支过多）的问题

工厂系列的模式（简单工厂、工厂方法，以及我们马上将要介绍的抽象工厂）都有三要素：
- 产品
- 工厂
- 客户端

```gotemplate
   //抽象产品即为汉堡，我们在此基础上派生出KFC和McDonalds的具体汉堡
   type Hamburger interface {
       Deliver()
   }
   
   type KfcHamburger struct{}
   
   func (h KfcHamburger) Deliver() {
       fmt.Println("This is a hamburger from KFC.\n")
   }
   
   type McdonaldsHamburger struct{}
   
   func (h McdonaldsHamburger) Deliver() {
       fmt.Println("This is a hamburger from McDonalds.\n")
   }
   
   //工厂--与简单工厂模式由一个工厂完成所有产品的创建不同，工厂方法定义了一个抽象的工厂基类，并由派生的具体工厂负责对应产品的生产
   type HamburgerFactory interface {
       Create() Hamburger
   }
   
   type Kfc struct{}
   
   func (f Kfc) Create() Hamburger {
       return new(KfcHamburger)
   }
   
   type Mcdonalds struct{}
   
   func (f Mcdonalds) Create() Hamburger {
       return new(McdonaldsHamburger)
   }
   
   //客户端
   func main() {
       prefer := getPreferHamburger()
   
       var factory HamburgerFactory
       switch prefer {
       case "KFC":
           factory = new(Kfc)
       case "McDonalds":
           factory = new(Mcdonalds)
       default:
           fmt.Printf("%s not supported yet.\n", prefer)
           os.Exit(1)
       }
   
       hamburger := factory.Create()
       hamburger.Deliver()
   }
   
   
   func getPreferHamburger() string {
   
   	var prefer string
   	flag.StringVar(&prefer, "prefer", "KFC", "Preferenced Hamburger")
   	flag.Parse()
   
   	if len(os.Args) < 2 {
   		flag.Usage()
   		os.Exit(1)
   	}
   
   	return prefer
   }
```


