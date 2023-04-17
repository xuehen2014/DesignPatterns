# 备忘录模式

#### 什么是备忘录模式
备忘录模式（Memento Pattern）又叫作快照模式（Snapshot Pattern）, 或令牌模式（Token Pattern）， 指在不破坏封装的前提下， 捕获一个对象的内部状态， 并在对象之外保存这个状态。 这样以后就可将该对象恢复到原先保存的状态， 属于行为型设计模式

#### 适用场景
需要保存历史快照的场景  
希望在对象之外保存状态，且除了自己，其他类对象无法访问状态保存的具体内容

```gotemplate
   package main
   
   import (
   	"errors"
   	"fmt"
   	"time"
   )
   
   // 编辑器接口定义
   type IEditor interface {
   	Title(title string)
   	Content(content string)
   	Save()
   	Undo() error
   	Redo() error
   	Show()
   }
   // 定义编辑器的备忘录, 也就是编辑器的内部状态数据模型, 同时也对应一个历史版本
   type EditorMemento struct {
   	title string
   	content string
   	createTime int64
   }
   
   func newEditorMemento(title string, content string) *EditorMemento {
   	return &EditorMemento{
   		title, content, time.Now().Unix(),
   	}
   }
   
   // 编辑器类, 实现IEditor接口
   type Editor struct {
   	title string
   	content string
   	versions []*EditorMemento
   	index int
   }
   
   func NewEditor() IEditor {
   	return &Editor{
   		"", "", make([]*EditorMemento, 0), 0,
   	}
   }
   
   func (editor *Editor) Title(title string) {
   	editor.title = title
   }
   
   func (editor *Editor) Content(content string) {
   	editor.content = content
   }
   
   func (editor *Editor) Save() {
   	it := newEditorMemento(editor.title, editor.content)
   	editor.versions = append(editor.versions, it)
   	editor.index = len(editor.versions) - 1
   }
   
   func (editor *Editor) Undo() error {
   	return editor.load(editor.index - 1)
   }
   
   func (editor *Editor) load(i int) error {
   	size := len(editor.versions)
   	if size <= 0 {
   		return errors.New("no history versions")
   	}
   
   	if i < 0 || i >= size {
   		return errors.New("no more history versions")
   	}
   
   	it := editor.versions[i]
   	editor.title = it.title
   	editor.content = it.content
   	editor.index = i
   	return nil
   }
   
   func (editor *Editor) Redo() error {
   	return editor.load(editor.index + 1)
   }
   
   func (editor *Editor) Show() {
   	fmt.Printf("tMockEditor.Show, title=%s, content=%s\n", editor.title, editor.content)
   }
   
   func main() {
   	editor := NewEditor()
   
   	// test save()
   	editor.Title("唐诗")
   	editor.Content("白日依山尽")
   	editor.Save()
   
   	editor.Title("唐诗 登鹳雀楼")
   	editor.Content("白日依山尽, 黄河入海流. ")
   	editor.Save()
   
   	editor.Title("唐诗 登鹳雀楼 王之涣")
   	editor.Content("白日依山尽, 黄河入海流。欲穷千里目, 更上一层楼。")
   	editor.Save()
   
   	// test show()
   	fmt.Println("-------------Editor 当前内容-----------")
   	editor.Show()
   
   	fmt.Println("-------------Editor 回退内容-----------")
   	for {
   		e := editor.Undo()
   		if e != nil {
   			break
   		} else {
   			editor.Show()
   		}
   	}
   
   	fmt.Println("-------------Editor 前进内容-----------")
   	for {
   		e := editor.Redo()
   		if e != nil {
   			break
   		} else {
   			editor.Show()
   		}
   	}
   }
```