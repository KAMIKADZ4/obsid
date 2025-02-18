Интерфейсы в Go - это тип, определяющий набор методов (определяют поведение). Если какой то тип (структура, срез, мапа) имеет все методы, указанные в интерфейсе, то считается, что тип реализуется интерфейс. 
```go
type MyInterfacer interface {
    Method1() int
    Method2(string) bool
}
```
В название интерфейсов принято использовать постфикс `-er` (Reader, Writer и тп)

Пример структуры, реализующий интерфейс
```go
type MyStruct struct {
    Value int
}

func (m MyStruct) Method1() int {
    return m.Value
}

func (m MyStruct) Method2(s string) bool {
    return s == "true"
}
```
Теперь `MyStruct` автоматически реализует `MyInterface`, потому что у него есть оба метода, указанные в интерфейсе.
> Если это выглядит как утка, плавает как утка и крякает как утка, то это, вероятно, утка и есть.
## Пустой интерфейс
В Go можно определить пустой интерфейс `interface{}`, который не содержит никаких методов. И каждый тип автоматически реализует этот интерфейс. Это часто используется для создания функций, которые могут принимать значения любого типа.
```go
var i interface{}
i = 42
i = "hello"
i = []int{1, 2, 3}
```

## Типовые утверждения и преобразования
Чтобы узнать, какой конкретный тип находится за интерфейсом, можно использовать **типовые утверждения** (`type assertion`)
```go
func describe(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Integer: %v\n", v)
    case string:
        fmt.Printf("String: %s\n", v)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}

func main() {
    describe(42)       // Integer: 42
    describe("hello")  // String: hello
    describe([]int{})  // Unknown type: []int
}
```
И выполнить преобразование типа напрямую
```go
func isString(i interface{}) bool {
    _, ok := i.(string)
    return ok
}
```
## Объединение интерфейса
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type ReadWriter interface {
    Reader
    Writer
}
```
В данном случае `ReadWriter` включает в себя все методы из `Reader` и `Writer`.
https://habr.com/ru/articles/276981/
## Интерфейсы под капотом

Интерфейс представляет собой структуру с указателем на `Interface Table` и на сам тип. В `Interface Table` хранится 
```go
type iface struct {
	tab  *itab          // указатель на таблицу методов (итаб)
	data unsafe.Pointer // указатель на данные
}
```
https://github.com/golang/go/blob/master/src/runtime/runtime2.go#L179

Структура `itab`
```go
type ITab struct {
    Inter *InterfaceType  // указатель на тип интерфейса
    Type  *Type          // указатель на конкретный тип, который реализует интерфейс
    Hash  uint32          // хеш типа для быстрого сравнения
    Fun   [1]uintptr      // массив указателей на функции (методы)
}
```
https://github.com/golang/go/blob/master/src/internal/abi/iface.go#L14



https://github.com/golang/go/blob/master/src/runtime/iface.go

https://habr.com/ru/articles/276981/