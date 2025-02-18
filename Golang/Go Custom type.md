https://code-basics.com/ru/languages/go/lessons/custom-types

Объявление кастояных типов
```go
type NumCount int

func (c *NumCount) inc() { *c++ }

func main() { 
	nc := NumCount(len([]int{1, 2, 3})) 
	fmt.Println(nc) // 3 
}
```