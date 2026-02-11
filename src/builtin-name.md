# Избегайте использования встроенных имён

[Спецификация языка] Go описывает несколько встроенных [предопределённых идентификаторов], которые не должны использоваться в качестве имён в программах на Go.

В зависимости от контекста, повторное использование этих идентификаторов в качестве имён либо затенит оригинал в текущей лексической области (и во всех вложенных областях), либо сделает затронутый код запутанным. В лучшем случае компилятор выдаст ошибку; в худшем случае такой код может внести скрытые, трудно обнаруживаемые баги.

  [Спецификация языка]: https://go.dev/ref/spec
  [предопределённых идентификаторов]: https://go.dev/ref/spec#Predeclared_identifiers

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
var error string
// `error` затеняет встроенный

// или

func handleErrorMessage(error string) {
    // `error` затеняет встроенный
}
```

</td><td>

```go
var errorMessage string
// `error` ссылается на встроенный

// или

func handleErrorMessage(msg string) {
    // `error` ссылается на встроенный
}
```

</td></tr>
<tr><td>

```go
type Foo struct {
    // Хотя эти поля технически не
    // создают затенение, grep поиск
    // строк `error` или `string` теперь
    // неоднозначен.
    error  error
    string string
}

func (f Foo) Error() error {
    // `error` и `f.error` визуально
    // похожи
    return f.error
}

func (f Foo) String() string {
    // `string` и `f.string` визуально
    // похожи
    return f.string
}
```

</td><td>

```go
type Foo struct {
    // Строки `error` и `string` теперь
    // однозначны.
    err error
    str string
}

func (f Foo) Error() error {
    return f.err
}

func (f Foo) String() string {
    return f.str
}
```

</td></tr>
</tbody></table>

Обратите внимание, что компилятор не выдаст ошибки при использовании предопределённых идентификаторов, но такие инструменты, как `go vet`, должны правильно указывать на эти и другие случаи затенения.
