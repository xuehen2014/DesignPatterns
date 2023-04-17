# 门面模式

#### 什么是门面模式
定义: 门面模式为子系统提供一组统一的接口，定义一组高层接口让子系统更易调用

门面模式又称为外观模式，它是一种结构型模式。引入外观模式后调用方与多个子系统的通信必须通过一个统一的外观对象进行，外观模式为子系统中的功能接口提供一个一致的界面，此模式定义了一个高层接口，这个接口使得这些子系统更加容易使用

```gotemplate
   package main
   
   import (
   	"fmt"
   )
   
   const (
   	BOOT_ADDRESS = 0
   	BOOT_SECTOR  = 0
   	SECTOR_SIZE  = 0
   )
   
   type CPU struct{}
   
   func (c *CPU) Freeze() {
   	    fmt.Println("CPU.Freeze()")
   }
   
   func (c *CPU) Jump(position int) {
   	    fmt.Println("CPU.Jump()")
   }
   
   func (c *CPU) Execute() {
   	    fmt.Println("CPU.Execute()")
   }
   
   type Memory struct{}
   
   func (m *Memory) Load(position int, data []byte) {
   	    fmt.Println("Memory.Load()")
   }
   
   type HardDrive struct{}
   
   func (hd *HardDrive) Read(lba int, size int) []byte {
        fmt.Println("HardDrive.Read()")
        return make([]byte, 0)
   }
   
   type ComputerFacade struct {
        processor *CPU
        ram       *Memory
        hd        *HardDrive
   }
   
   func NewComputerFacade() *ComputerFacade {
   	    return &ComputerFacade{new(CPU), new(Memory), new(HardDrive)}
   }
   
   //各子系统对外统一入口
   func (c *ComputerFacade) start() {
        c.processor.Freeze()
        c.ram.Load(BOOT_ADDRESS, c.hd.Read(BOOT_SECTOR, SECTOR_SIZE))
        c.processor.Jump(BOOT_ADDRESS)
        c.processor.Execute()
   }
   
   func main() {
        computer := NewComputerFacade()
        computer.start()
   }
```
