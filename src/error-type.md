# Типы ошибок

Существует несколько вариантов объявления ошибок.
Прежде чем выбрать наиболее подходящий вариант для вашего случая, рассмотрите следующее.

- Нужно ли вызывающей стороне сопоставлять ошибку, чтобы обработать её?
  Если да, мы должны поддержать функции [`errors.Is`] или [`errors.As`],
  объявив переменную ошибки верхнего уровня или пользовательский тип.
- Является ли сообщение об ошибке статической строкой
  или динамической строкой, требующей контекстной информации?
  В первом случае мы можем использовать [`errors.New`], но во втором случае мы должны
  использовать [`fmt.Errorf`] или пользовательский тип ошибки.
- Передаём ли мы дальше новую ошибку, возвращённую нижележащей функцией?
  Если да, см. [раздел об оборачивании ошибок](error-wrap.md).

[`errors.Is`]: https://pkg.go.dev/errors#Is
[`errors.As`]: https://pkg.go.dev/errors#As

| Сопоставление? | Сообщение | Рекомендация                        |
|----------------|-----------|-------------------------------------|
| Нет            | static    | [`errors.New`]                      |
| Нет            | dynamic   | [`fmt.Errorf`]                      |
| Да             | static    | `var` верхнего уровня с [`errors.New`] |
| Да             | dynamic   | пользовательский тип `error`        |

[`errors.New`]: https://pkg.go.dev/errors#New
[`fmt.Errorf`]: https://pkg.go.dev/fmt#Errorf

Например,
используйте [`errors.New`] для ошибки со статической строкой.
Экспортируйте эту ошибку как переменную для поддержки сопоставления с помощью `errors.Is`,
если вызывающей стороне нужно сопоставить и обработать эту ошибку.

<table>
<thead><tr><th>Без сопоставления</th><th>С сопоставлением</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

if err := foo.Open(); err != nil {
  // Не можем обработать ошибку.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if errors.Is(err, foo.ErrCouldNotOpen) {
    // обрабатываем ошибку
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

Для ошибки с динамической строкой
используйте [`fmt.Errorf`], если вызывающей стороне не нужно сопоставлять её,
и пользовательский тип `error`, если вызывающей стороне нужно сопоставлять её.

<table>
<thead><tr><th>Без сопоставления</th><th>С сопоставлением</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

// package bar

if err := foo.Open("testfile.txt"); err != nil {
  // Не можем обработать ошибку.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

func Open(file string) error {
  return &NotFoundError{File: file}
}


// package bar

if err := foo.Open("testfile.txt"); err != nil {
  var notFound *NotFoundError
  if errors.As(err, &notFound) {
    // обрабатываем ошибку
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

Обратите внимание, что если вы экспортируете переменные или типы ошибок из пакета,
они станут частью публичного API пакета.
