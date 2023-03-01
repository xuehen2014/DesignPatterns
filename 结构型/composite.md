# 组合模式

#### 什么是组合模式    
组合模式（Composite Pattern）又叫作部分-整体（Part-Whole）模式，它的宗旨是通过将单个对象（叶子节点）和组合对象（树枝节点）用相同的接口进行表示，使得客户对单个对象和组合对象的使用具有一致性，属于结构型设计模式  

#### 应用场景   
组合模式的使用要求业务场景中的实体必须能够表示成树形结构才行，由组合模式将一组对象组织成树形结构，客户端（代码的使用者）可以将单个对象和组合对象都看做树中的节点，以统一处理逻辑，并且利用树形结构的特点，将对树、子树的处理转化成叶节点的递归处理，依次简化代码实现  
通过上边的描述我们可以马上想到文件系统、公司组织架构这些有层级结构的事物的操作会更适合应用组合模式

#### 组合模式由以下几个角色构成

- 组件 （Component）： 组件是一个接口，描述了树中单个对象和组合对象都要实现的的操作。
- 叶节点 （Leaf） ：即单个对象节点，是树的基本结构， 它不包含子节点，因此也就无法将工作指派给下去，叶节点最终会完成大部分的实际工作。
- 组合对象 （Composite）”——是包含叶节点或其他组合对象等子项目的符合对象。 组合对象不知道其子项目所属的具体类， 它只通过通用的组件接口与其子项目交互。
- 客户端 （Client）： 通过组件接口与所有项目交互。 因此， 客户端能以相同方式与树状结构中的简单或复杂对象进行交互

```gotemplate
   // 表示组织机构的接口
   type Organization interface {
       display()
       duty()
   }
   
   //组合对象
   type CompositeOrganization struct {
       orgName string
       depth   int
       list    []Organization
   }
   
   func NewCompositeOrganization(name string, depth int) *CompositeOrganization {
       return &CompositeOrganization{name, depth, []Organization{}}
   }
   
   func (c *CompositeOrganization) add(org Organization) {
       if c == nil {
           return
       }
       c.list = append(c.list, org)
   }
   
   func (c *CompositeOrganization) remove(org Organization) {
       if c == nil {
           return
       }
       for i, val := range c.list {
           if val == org {
               c.list = append(c.list[:i], c.list[i+1:]...)
               return
           }
       }
       return
   }
   
   func (c *CompositeOrganization) display() {
       if c == nil {
           return
       }
       fmt.Println(strings.Repeat("-", c.depth * 2), " ", c.orgName)
       for _, val := range c.list {
           val.display()
       }
   }
   
   func (c *CompositeOrganization) duty() {
       if c == nil {
           return
       }
   
       for _, val := range c.list {
           val.duty()
       }
   }
   
   // Leaf对象--人力资源部门
   type HRDOrg struct {
       orgName string
       depth   int
   }
   
   func (o *HRDOrg) display() {
       if o == nil {
           return
       }
       fmt.Println(strings.Repeat("-", o.depth * 2), " ", o.orgName)
   }
   
   func (o *HRDOrg) duty() {
       if o == nil {
           return
       }
       fmt.Println(o.orgName, "员工招聘培训管理")
   }
   
   // Leaf对象--财务部门
   type FinanceOrg struct {
       orgName string
       depth   int
   }
   
   func (f *FinanceOrg) display() {
       if f == nil {
           return
       }
       fmt.Println(strings.Repeat("-", f.depth * 2), " ", f.orgName)
   }
   
   func (f *FinanceOrg) duty() {
       if f == nil {
           return
       }
       fmt.Println(f.orgName, "员工招聘培训管理")
   }
   
   //具体调用
   func main() {
       root := NewCompositeOrganization("北京总公司", 1)
       root.add(&HRDOrg{orgName: "总公司人力资源部", depth: 2})
       root.add(&FinanceOrg{orgName: "总公司财务部", depth: 2})
   
       compSh := NewCompositeOrganization("上海分公司", 2)
       compSh.add(&HRDOrg{orgName: "上海分公司人力资源部", depth: 3})
       compSh.add(&FinanceOrg{orgName: "上海分公司财务部", depth: 3})
       root.add(compSh)
   
       compGd := NewCompositeOrganization("广东分公司", 2)
       compGd.add(&HRDOrg{orgName: "广东分公司人力资源部", depth: 3})
       compGd.add(&FinanceOrg{orgName: "南京办事处财务部", depth: 3})
       root.add(compGd)
   
       fmt.Println("公司组织架构：")
       root.display()
   
       fmt.Println("各组织的职责：")
       root.duty()
   }

```