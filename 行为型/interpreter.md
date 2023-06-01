# 解释器模式

#### 什么是解释器模式
解释器模式，用于解决需要解释语言中的句子或表达式的问题。以下是一些可以在 程序中使用解释器模式的真实场景：

处理配置文件
许多应用程序使用配置文件来指定应用程序的行为方式。这些配置文件可以用 YAML 或 JSON 等 DSL 编写。解释器可用于解析这些配置文件并以应用编程语言对象的形式向应用程序提供配置信息。
模板引擎
模板引擎处理模板和一组变量以产生输出。模板是DSL的一个例子，可以使用Interpreter来解析和处理模板。
数学表达式计算器
数学表达式是我们日常都能接触到的，使用了一种特定领域语言语法书写语句或者叫表达式的实例
这些表达式在程序里可以使用解释器模式进行解析和解释。例如，计算器应用程序可以使用解释器来解析和评估用户输入的数学表达式。
自然语言处理
在更高级的情况下，解释器模式可用于解析和解释自然语言，不过这通常会涉及想机器学习这样的更复杂的技术。
虽然解释器模式可以用来解决这些问题，但它并不总是最好的解决方案。对于复杂的语言，使用特定的解析库或工具或其他设计模式可能更有效

```gotemplate
   package main
   
   import (
   	"fmt"
   	"strings"
   )
   
   type Expression interface {
   	Interpret(variables map[string]Expression) int
   }
   
   type Number struct {
   	number int
   }
   
   func (n *Number) Interpret(variables map[string]Expression) int {
   	return n.number
   }
   
   type Plus struct {
   	leftOperand  Expression
   	rightOperand Expression
   }
   
   func (p *Plus) Interpret(variables map[string]Expression) int {
   	return p.leftOperand.Interpret(variables) + p.rightOperand.Interpret(variables)
   }
   
   type Minus struct {
   	leftOperand  Expression
   	rightOperand Expression
   }
   
   func (m *Minus) Interpret(variables map[string]Expression) int {
   	return m.leftOperand.Interpret(variables) - m.rightOperand.Interpret(variables)
   }
   
   type Variable struct {
   	name string
   }
   
   func (v *Variable) Interpret(variables map[string]Expression) int {
   	if variables[v.name] == nil {
   		return 0
   	}
   	return variables[v.name].Interpret(variables)
   }
   
   type Evaluator struct {
   	syntaxTree Expression
   }
   
   func NewEvaluator(expression string) *Evaluator {
   	expressionStack := new(Stack)
   	for _, token := range strings.Split(expression, " ") {
   		if token == "+" {
   			right := expressionStack.Pop().(Expression)
   			left := expressionStack.Pop().(Expression)
   			subExpression := &Plus{left, right}
   			expressionStack.Push(subExpression)
   		} else if token == "-" {
   			right := expressionStack.Pop().(Expression)
   			left := expressionStack.Pop().(Expression)
   			subExpression := &Minus{left, right}
   			expressionStack.Push(subExpression)
   		} else {
   			expressionStack.Push(&Variable{token})
   		}
   	}
   	syntaxTree := expressionStack.Pop().(Expression)
   	return &Evaluator{syntaxTree}
   }
   
   func (e *Evaluator) Interpret(context map[string]Expression) int {
   	return e.syntaxTree.Interpret(context)
   }
   
   type Element struct {
   	value interface{}
   	next  *Element
   }
   
   type Stack struct {
   	top  *Element
   	size int
   }
   
   func (s *Stack) Push(value interface{}) {
   	s.top = &Element{value, s.top}
   	s.size++
   }
   
   func (s *Stack) Pop() interface{} {
   	if s.size == 0 {
   		return nil
   	}
   	value := s.top.value
   	s.top = s.top.next
   	s.size--
   	return value
   }
   
   func main() {
   	expression := "w x z - +"
   	sentence := NewEvaluator(expression)
   	variables := make(map[string]Expression)
   	variables["w"] = &Number{5}
   	variables["x"] = &Number{10}
   	variables["z"] = &Number{42}
   	result := sentence.Interpret(variables)
   	fmt.Println(result)
   }
```