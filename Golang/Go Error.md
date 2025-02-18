https://code-basics.com/ru/languages/go/lessons/errors

Что бы создать свою ошибку (просто текст) можно использовать этот код
```go
import "errors"

err := errors.New("my error description")
```

Ошибки в Go тесно связаны с понятием интерфейсов([[Go Interface]]), ведь тип `error` это интерфейс, который реализует 1 метод. 
```go
type error interface{
	Error() string
}
```

Поэтому ваша структура может легко подойти под ошибку
```go
import "fmt"

type TimeoutErr struct {
	msg       string
	threshold int
}

func (err TimeoutErr) Error() string {
	return fmt.Sprintf("%s: %d second", err.msg, err.threshold)
}

func spawnError(i int) (int, err){
	if i <= 10 {
		return i*i, nil
	}
	return -1, TimeoutErr{msg: "TIMEOUT!! OMG", threshold: 30}
}

func main() {
	result, err := spawnError(5)
	if err != nil {
		fmt.Println("Жить в кайф!")
	} else {
		fmt.Println("Что то пошло не так")
		panic(tErr)
	}	
}
```

Если из функции предполагается возврат ошибки, то она должна возвращаться последним аргументом, если ошибки не возникло, то возвращать следует `nil`
Благодаря `err != nil` можно поверить произошла ли ошибка.

Функция `fmt.Errorf` позволяет создавать новые ошибка из форматированной строки и возвращает структуру, которая реализует интерфейс `error`
`%w` - 
```go
type wrapError struct {
	msg string
	err error
}

formatErr := fmt.Errorf("failed to open file: %w", someErr)
// formatErr - структура wrapError, реализующая интерфейс error
```
https://github.com/golang/go/blob/master/src/fmt/errors.go#L22
## Функция `errors.Is`
**Семантика функции**: `func Is(err, target error) bool`
Функция проходит по цепочке обертывания ошибок, ищет целевую ошибку и возвращает `true` или `false` в зависимости от успеха
```go
package main

import (
	"errors"
	"fmt"
)

var ErrNotFound = errors.New("not found")

type MyError struct {
	msg       string
	baseError error
}

func (err MyError) Error() string {
	return err.msg
}

// При вызове этого метода у MyError он знает в каком атрибуте
// находится обернутая ошибка и вызывает уже для него Is метод
func (err MyError) Is(target error) bool {
	return errors.Is(err.baseError, target)
}

func main() {
	err := fmt.Errorf("error occurred: %w", ErrNotFound)
	myErr := MyError{msg: "Hello", baseError: err}

	// Проверяем, есть ли в цепочке оберток myErr ошибка ErrNotFound
	if errors.Is(myErr, ErrNotFound) {
		fmt.Println("Error is 'not found'")
	} else {
		fmt.Println("Error is something else")
	}
}
```
В примере выше ошибка обернуты так: `ErrNotFound` -> `err` -> `myErr`
Для использования `errors.Is` для кастомной ошибки нужно реализовать интерфейс из одного метода `Is(target error) bool` ну или метод `UnWrap`
https://github.com/golang/go/blob/master/src/errors/wrap.go#L44
## Функция `errors.As`
**Семантика функции**: `func As(err error, target interface{}) bool`
Функция проходит по цепочке обертывания ошибок (если они есть) и пытается преобразовать ошибку к указанному типу. 
Аргумент `target` являться non-nil указателем на тип, реализующий интерфейс `error`
Если преобразование успешно, функция возвращает `true` и записывает значение ошибки в целевую переменную. В противном случае возвращается `false`.
Алгоритм работы:
- Сначала `errors.As()` проверяет, соответствует ли текущая ошибка указанному типу.
- Если текущая ошибка реализует интерфейс `Unwrap() error`, то `errors.As()` вызывает метод `Unwrap()` для получения следующей ошибки в цепочке. Затем процесс повторяется рекурсивно для развернутой ошибки.
Процесс продолжается до тех пор, пока не будет найдена совпадающая ошибка или пока не закончится цепочка обертывания (т.е. пока `Unwrap()` не вернет `nil`).
```go
package main

import (
	"errors"
	"fmt"
)

var ErrNotFound = errors.New("not found")

type MyError struct {
	msg       string
	someInt   int
	baseError error
}

func (err MyError) Error() string {
	return err.msg
}

func main() {

	myErr := MyError{msg: "Hello", someInt: 666, baseError: ErrNotFound}
	err := fmt.Errorf("error occurred: %w", myErr)

	var myErrNew MyError
	
	// fmt.Println(err.someInt) - ошибка компиляции, тк err не имеет такого поля
	// Ошибку err приводит у указаному типу

	// по цепочку переменную err буду "развертывать" до момента встречи ошибки
	// с типом MyError, и запишут найденную структуру по укателю &myErrNew в myErrNew
	fmt.Println(errors.As(err, &myErrNew)) // true
	// и можно будет воспользоваться полем someInt
	fmt.Println(myErrNew.someInt) // 666

}
```
Благодаря `reflectlite.TypeOf(err).AssignableTo(targetType)` на строке 118 идёт проверка того, можно ли тип значения переменной `err` присвоить в `target`
https://github.com/golang/go/blob/master/src/errors/wrap.go#L97


## Функция `errors.Unwrap`
**Семантика функции**: `func Unwrap(err error) error`
Используется для получения обернутой (wrapped) ошибки из текущей ошибки.
```go
package main

import (
	"errors"
	"fmt"
)

var ErrNotFound = errors.New("not found")

type MyError struct {
	msg       string
	baseError error
}

func (err MyError) Error() string {
	return err.msg
}

func (err MyError) Unwrap() error {
	return err.baseError
}

func main() {
	err := fmt.Errorf("[I am err (%w) in me]", ErrNotFound)
	myErr := MyError{msg: "Hello", baseError: err}

	fmt.Printf("wrapped error: %s\n", errors.Unwrap(myErr))
	// wrapped error: error occurred: not found
}
```
Для корректной работы с вашими кастомными ошибками следует описать метод `Unwrap` для вашей ошибки, что будет возвращать ошибку, которую вы обернули своей ошибкой.
https://github.com/golang/go/blob/master/src/errors/wrap.go#L17

```
myErr := errors.New("[MyError]")
fmtErr := fmt.Errorf("[FMT error inside:%w]", myErr)

// fmt.Println(fmtErr.Unwrap()) - ошибка компиляции, тк fmt.Errorf возвращает тип error
// этот тип - интерфейс только с методом Error

fmt.Println(fmtErr.Error()) // [FMT error inside:[MyError]]

// Поэтому что бы можно было вызвать Unwrap, то приведем к другому интерфейсу
unwrapErr := fmtErr.(interface{ Unwrap() error })
fmt.Println(unwrapErr.Unwrap()) // [MyError]
fmt.Println(unwrapErr.Error()) // Ошибка компиляции
```


https://habr.com/ru/companies/otus/articles/453806/

## Panic & Recover

Функция `panic` вызывает немедленное прекращение выполнения программы, а `recover` позволяет перехватить панику и продолжить выполнение.
```go
package main

import (
	"errors"
	"fmt"
)

func riskyOperation() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()
    // panic("something went wrong") - тоже можно
    myerr := errors.New("[MyError]")
    panic(myerr)
}

func main() {
	/*
	// здесь тоже может быть определен defer с recover
	defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()
    */
	fmt.Println("Hello, World")
	riskyOperation()
}
```
Важно заметить, что `recover()` должен быть в `defer` функции([[Go Defer]]), тк после паники это единственный код, что сможет гарантированно выполниться и соответственно перехватить панику