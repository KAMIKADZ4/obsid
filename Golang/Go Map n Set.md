## Создание map
```go
// создание пустой мапы
var m map[int]string 
fmt.Println(m, m == nil) // map[] true
// m[1] = "one" ошибка будет

// сокращенное создание пустой мапы 
m := map[int]string{} 
// m == nil false

// рекомендуемое создание с обозначением размера 
m := make(map[int]string, 10) 
// создание мапы с элементами 
m := map[int]string{1: "hello", 2: "world"}
```

## Чтение из map
При чтении можно получить 2 значения (1 - само значение, 2 - есть ли ключ в мапе). Если ключа в мапе нет, то ошибки не будет, вернется в качестве значения zero value
```go
elements := map[string]int{"Mike": 10, "Alice": 0}

elem, exists := elements["Mike"] // 10, true
elem, exists := elements["Alice"] // 0, true

elem, exists := elements["Bob"] // 0, false
```

## Добавление в map
Добавляется обычным присваиванием, также можно сразу выполнять действия надо ключами что ещё не созданы, в качестве начального значения будет принять zero value
```go
elements := map[string]int{"Mike": 10}
elements["Alice"] = 100
// map[Mike:10 Alice:100]

elements["Bob"] += 99
// map[Mike:10 Alice:100 Bob:99]
```

## Удаление в map
Удаление осуществляется с помощью функции `detele`, если ключа не существует, то ошибки не возникнет (просто ничего не будет удалено)
```go
delete(elements, "Bob")
```

## Примечание
Мапы всегда передаются по ссылке, поэтому следует быть осторожнее
```go
func main() { 
	m := map[int]string{1: "hello", 2: "world"} 
	modifyMap(m) 
	fmt.Println(m) // вывод: map[1:changed 2:world 200:added] 
}

func modifyMap(m map[int]string) { 
	m[200] = "added" 
	m[1] = "changed" 
}
```

Также ничего не мешает в map класть объекты разных типов данных
```go
mapa := make(map[int]any) // значение любого типа

mapa := make(map[any]int) // ключ любого типа

mapa := make(map[any]any) // и ключ и значение любого типа

mapa[10] = "Mike"
mapa["Bob"] = "Alice"
mapa[true] = 10

```

# Множества

 в Go нет отдельного типа данных для множества, тут главное знать небольшую хитрость, мы может определить структуру `struct{}` - пустая структура, которая после инициализации не будет занимать места
```
mapa := make(map[int]struct{})

mapa[10] = struct{}{}


```