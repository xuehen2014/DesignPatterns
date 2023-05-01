# 状态模式

#### 什么是状态模式
状态模式通过将事件触发的状态转移和动作执行拆分到不同的状态类中，来避免分支判断逻辑

状态模式用于解决系统中复杂对象的状态转换以及不同状态下行为的封装问题。当系统中某个对象存在多个状态，这些状态之间可以进行转换，而且对象在不同状态下行为不相同时可以使用状态模式，把特定于状态的代码抽象到一组独立的状态类中避免过多的状态条件判断，减少维护成本

#### 构成
- Context（环境类）：环境类又称为上下文类，它定义客户端需要的接口，内部维护一个当前状态实例，并负责具体状态的切换。
- State（抽象状态）：定义状态下的行为，可以有一个或多个行为。
- ConcreteState（具体状态）：每一个具体状态类对应环境的一个具体状态，不同的具体状态类其行为有所不同


```gotemplate
// State Pattern
package main

import (
	"fmt"
	"time"
)

// State interface
type LightState interface {
	// 亮起当前状态的交通灯
	Light()
	// 转换到新状态的时候，调用的方法
	EnterState()
	// 设置一个状态要转变的状态
	NextLight(light *TrafficLight)
	// 检测车速
	CarPassingSpeed(*TrafficLight, int, string)
}

// Context
type TrafficLight struct {
	State LightState
	SpeedLimit int
}

func NewSimpleTrafficLight(speedLimit int) *TrafficLight {
	return &TrafficLight{
		SpeedLimit: speedLimit,
		State: NewRedState(),
	}
}

func (tl *TrafficLight) TransitionState(newState LightState) {
	tl.State = newState
	tl.State.EnterState()
}

// 用于让具体LightState 嵌套实现类似继承的效果，减少公用法在每个具体 LightState 实现类中的重复实现
// 它只实现LightState里的通用方法的默认版，不能实现所有的方法，那样的话他也就算一个 LightState 具体实现了，
// 而这不是我们想要的，每个类型逻辑不同的关键方法以及覆盖默认版的通用方法的工作，留给具体类型去实现。
type DefaultLightState struct {
	StateName string
}

// Draw 和 EnterState 的逻辑交给每个 LightState 的实现类来实现
func (state *DefaultLightState) CarPassingSpeed(road *TrafficLight, speed int, licensePlate string) {
	if speed > road.SpeedLimit {
		fmt.Printf("Car with license %s was speeding\n", licensePlate)
	}
}

func (state *DefaultLightState) EnterState(){
	fmt.Println("changed state to:", state.StateName)
}


// 红灯状态
type redState struct {
	DefaultLightState
}

func NewRedState() *redState {
	state := &redState{}
	state.StateName = "RED"
	return state
}

func (state *redState) Light() {
	fmt.Println("红灯亮起，不可行驶")
}

func (state *redState) CarPassingSpeed(light *TrafficLight, speed int, licensePlate string) {
	// 红灯时不能行驶， 所以这里要重写覆盖 DefaultLightState 里定义的这个方法
	if speed > 0 {
		fmt.Printf("Car with license \"%s\" ran a red light!\n", licensePlate)
	}
}

func (state *redState) NextLight(light *TrafficLight){
	light.TransitionState(NewGreenState())
}

// 绿灯状态
type greenState struct{
	DefaultLightState
}

func NewGreenState() *greenState{
	state :=  &greenState{}
	state.StateName = "GREEN"
	return state
}

func (state *greenState) Light(){
	fmt.Println("绿灯亮起，请行驶")
}

func (state *greenState) NextLight(light *TrafficLight){
	light.TransitionState(NewAmberState())
}

// 黄灯状态
type amberState struct {
	DefaultLightState
}

func NewAmberState() *amberState{
	state :=  &amberState{}
	state.StateName = "AMBER"
	return state
}

func (state *amberState) Light(){
	fmt.Println("黄灯亮起，请注意")
}

func (state *amberState) NextLight(light *TrafficLight){
	light.TransitionState(NewRedState())
}

func main() {
	trafficLight := NewSimpleTrafficLight(500)

	interval := time.NewTicker(5 * time.Second)
	for {
		select {
			case <- interval.C:
				trafficLight.State.Light()
				trafficLight.State.CarPassingSpeed(trafficLight, 25, "CN1024")
				trafficLight.State.NextLight(trafficLight)
		default:
		}
	}
}

```