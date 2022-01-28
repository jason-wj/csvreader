# csvreader
Jason <idea_wj@163.com>
简单的csv格式文件解析到`struct`工具

2022-01-27 注：为满足自身需要，在 [原作者：zhnxin 项目](https://github.com/zhnxin/csvreader) 基础上做了改动：
1. 当header中字段首尾存在空格时，去除空格
2. value值，去除开头和结尾的空格
3. header中的字段必须全为小写，结构体随意，但名称必须和header中字段名称保持一致

## 安装
```shell
go get github.com/jason-wj/csvreader
```

## 简单用法

NOTE: 默认情况下，*csv* 文件的首行会被当作header处理。

```csv
hosname,ip
redis,172.17.0.2
mariadb,172.17.0.3
```

```go
type Info struct{
    Hostname string
    IP string
}

//struct slice
infos := []Info{}
_ = csvreader.New().UnMarshalFile("file.csv",&infos)
body,_ := json.Marshal(infos)
fmt.Println(string(body))

//point slice
infos = []*Info{}
_ = csvreader.New().UnMarshalFile("file.csv",&infos)
body,_ := json.Marshal(infos)
fmt.Println(string(body))
```

NOTE: 如果 *csv* 文件首行不包含header，可以使用 *WithHeader([]string)* 来指定header。

```go
_ = csvreader.New().WithHeader([]string{"hostname","ip"}).UnMarshalFile("file.csv",&infos)
```

## 自定义parster

就像枚举类型(enum),偶尔会遇到这种需要实现自定义转换过程的情况。例子如下

```go
type NetProtocol uint32
const(
    NetProtocol_TCP NetProtocol = iota
    NetProtocol_UDP
    NetProtocol_DCCP
    NetProtocol_SCTP
)

type ServiceInfo struct{
    Host string
    Port string
    Protocol NetProtocol
}
```

直接使用原始的类型来编辑csv文件，十分不便。这时就需要实现自定义parser。

```go
type CsvMarshal interface {
    FromString(string) error
}
```

```go
func (p *NetProtocol)FromString(protocol string) error{
switch strings.ToLower(protocol){
    case "tcp":
        *p = NetProtocol_TCP
    case "udp":
        *p = NetProtocol_UDP
    case "dccp":
        *p = NetProtocol_DCCP
    case "sctp":
        *p = NetProtocol_SCTP
    default:
        return fmt.Errorf("unknown protocoal:%s",protocol)
    }
    return nil
}
```
另外一个例子 reader_test.go TestCustom