В Go `defer` позволяет отложить выполнение функции до завершения текущей функции. Даже если функция завершается с помощью return или panic, то defer гарантирует, что все отложенные функции будут выполнены. Часто используется для освобождения ресурсов (закрытие файла или соединения с бд, освобождение mutex'ов)

Когда вы используете `defer` перед вызовом функции, этот вызов помещается в стек отложенных вызовов. Все отложенные вызовы выполняются в обратном порядке (LIFO — Last In, First Out)
```go
func main() {
	defer fmt.Println(1)
	defer fmt.Println(2)
	defer fmt.Println(3)
	fmt.Println("Some work")
}
// Some work
// 3
// 2
// 1
```

Даже вызов паники в `defer` не помещает остальным `defer` функция
```go
func main() {
	defer fmt.Println(1)
	defer panic("OH NOO")
	defer fmt.Println(3)
	fmt.Println("Some work")
}
// Some work
// 3
// 1
// panic: OH NOO
```

Учитывайте, что какие переменные передаются по значению, а какие то по ссылке.
```go
func main() {
	x := 10
	defer fmt.Println("Defer X:", x)
	x = 100

	m := map[string]int{"Alice": 5}
	defer fmt.Println("Defer m:", m)
	m["Mike"] = 55
	m["Alice"]++

	fmt.Println("Result x:", x)
	fmt.Println("Result m:", m)
}
```
Вывод
```
Result x: 100
Result m: map[Alice:6 Mike:55]
Defer m: map[Alice:6 Mike:55]
Defer X: 10
```
