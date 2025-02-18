
Если у нескольких аргументов одинаковый тип данных, то можно указать тип данных только для последнего аргумента этого типа
```go
func multiply(x int, y int, z int, name string) int {
    // code
}

// упрощенный вариант
func multiply(x, y, z int, name string) int {
    // code
}
```

Используете `...` перед типом для аргумент был вариативным и принимал переменное количество аргументов в себя
```go
package main 

import "fmt"

func main() { 
	// кол-во аргументов может быть любым 
	PrintNums(1, 2, 3)
	x := []int{4, 5, 6}
	PrintNums(x...) // распаковка
} 

func PrintNums(nums ...int) { 
	// nums - []int
	for _, n := range nums { 
		fmt.Println(n) 
	} 
}
```

Возвращаемое значение может быть одно
```go
func greet() string {
    return "Hello, World!"
}
```
А может возвращаемых значений быть больше
```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("деление на ноль")
    }
    return a / b, nil
}
```

Также возвращаемое значение может быть именнованное
```go
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return
}
```
Здесь `x` и `y` объявлены как возвращаемые значения и инициализированы с `zero value` , и оператор `return` без аргументов вернёт их текущие значения.

Функции могут быть анонимные
```go
func main() {
    add := func(x, y int) int {
        return x + y
    }
    fmt.Println(add(2, 3)) // 5
	
	fmt.Println(func(x, y int) int {
        return x * y
    }(2, 3)) // 6
}
```

Функции так же можно передавать как аргументы функций
```go
func apply(op func(int, int) int, a, b int) int {
    return op(a, b)
}

func mul(x, y int) int {
	return x * y
}

func main() {
    add := func(x, y int) int { return x + y }
    fmt.Println(apply(add, 2, 3)) // 5
	fmt.Println(apply(mul, 2, 3)) // 6
}
```