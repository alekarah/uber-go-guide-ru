# Объявления переменных верхнего уровня

На верхнем уровне используйте стандартное ключевое слово `var`. Не указывайте тип, если он не отличается от типа выражения.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
var _s string = F()

func F() string { return "A" }
```

</td><td>

```go
var _s = F()
// Поскольку F уже указывает, что возвращает строку, нам не нужно
// указывать тип снова.

func F() string { return "A" }
```

</td></tr>
</tbody></table>

Указывайте тип, если тип выражения не совпадает точно с желаемым типом.

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F возвращает объект типа myError, но мы хотим error.
```
