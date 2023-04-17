# 桥接模式

#### 什么是桥接模式
桥接模式（Bridge Pattern）又叫作桥梁模式、接口模式或柄体（Handle and Body）模式，指将抽象部分与具体实现部分分离，使它们都可以独立地变化，属于结构型设计模式

#### 适用场景
- 在抽象和具体实现之间需要增加更多灵活性的场景  
- 一个负责某块逻辑的类存在两个或多个独立变化的维度，而这些维度都需要独立进行扩展 
- 不希望使用继承，或因为多层继承导致系统类的个数剧增

```gotemplate

   package main
   
   import (
   	"bytes"
   	"fmt"
   	"io"
   	"math/rand"
   )
   
   // 数据导出器
   type IDataExporter interface {
        Fetcher(fetcher IDataFetcher)
        Export(sql string, writer io.Writer) error
   }
   
   // 数据查询器
   type IDataFetcher interface {
   	    Fetch(sql string) []interface{}
   }
   
   //mysql数据查询器
   type MysqlDataFetcher struct {
   	    Config string
   }
   
   func (mf *MysqlDataFetcher) Fetch(sql string) []interface{} {
        fmt.Println("Fetch data from mysql source: " + mf.Config)
        rows := make([]interface{}, 0)
        rows = append(rows, rand.Perm(10), rand.Perm(10))
        return rows
   }
   
   func NewMysqlDataFetcher(configStr string) IDataFetcher {
        return &MysqlDataFetcher{
            Config: configStr,
        }
   }
   
   //oracle数据查询器
   type OracleDataFetcher struct {
   	    Config string
   }
   
   func (of *OracleDataFetcher) Fetch(sql string) []interface{} {
        fmt.Println("Fetch data from oracle source: " + of.Config)
        rows := make([]interface{}, 0)
        rows = append(rows, rand.Perm(10), rand.Perm(10))
        return rows
   }
   
   func NewOracleDataFetcher(configStr string) IDataFetcher {
      	return &OracleDataFetcher{
      		configStr,
      	}
   }
   
   
   //CSV数据导出器
   type CsvExporter struct {
   	    mFetcher IDataFetcher
   }
   
   func NewCsvExporter(fetcher IDataFetcher) IDataExporter {
        return &CsvExporter{
            fetcher,
        }
   }
   
   func (ce *CsvExporter) Fetcher(fetcher IDataFetcher) {
   	    ce.mFetcher = fetcher
   }
   
   func (ce *CsvExporter) Export(sql string, writer io.Writer) error {
        rows := ce.mFetcher.Fetch(sql)
        fmt.Printf("CsvExporter.Export, got %v rows\n", len(rows))
        for i, v:= range rows {
            fmt.Printf("  行号: %d 值: %s\n", i + 1, v)
        }
        return nil
   }
   
   //JSON数据导出器
   type JsonExporter struct {
   	    mFetcher IDataFetcher
   }
   
   func NewJsonExporter(fetcher IDataFetcher) IDataExporter {
        return &JsonExporter{
            fetcher,
        }
   }
   
   func (je *JsonExporter) Fetcher(fetcher IDataFetcher) {
   	    je.mFetcher = fetcher
   }
   
   func (je *JsonExporter) Export(sql string, writer io.Writer) error {
        rows := je.mFetcher.Fetch(sql)
        fmt.Printf("JsonExporter.Export, got %v rows\n", len(rows))
        for i, v:= range rows {
            fmt.Printf("  行号: %d 值: %s\n", i + 1, v)
        }
        return nil
   }
   
   
   func main() {
        mFetcher := NewMysqlDataFetcher("mysql://127.0.0.1:3306")
        csvExporter := NewCsvExporter(mFetcher)
        var writer bytes.Buffer
        // 从MySQL数据源导出 CSV
        csvExporter.Export("select * from xxx", &writer)
       
        oFetcher := NewOracleDataFetcher("mysql://127.0.0.1:1001")
        csvExporter.Fetcher(oFetcher)
        // 从 Oracle 数据源导出 CSV
        csvExporter.Export("select * from xxx", &writer)
       
        // 从 MySQL 数据源导出 JSON
        jsonExporter := NewJsonExporter(mFetcher)
        jsonExporter.Export("select * from xxx", &writer)
   }
```