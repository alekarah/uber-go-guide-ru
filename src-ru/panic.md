# Не используйте panic

Код, работающий в production, должен избегать panic. Panic — это основная причина [каскадных сбоев]. Если возникает ошибка, функция должна вернуть ошибку и позволить вызывающей стороне решить, как её обработать.

  [каскадных сбоев]: https://en.wikipedia.org/wiki/Cascading_failure

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func run(args []string) {
  if len(args) == 0 {
    panic("an argument is required")
  }
  // ...
}

func main() {
  run(os.Args[1:])
}
```

</td><td>

```go
func run(args []string) error {
  if len(args) == 0 {
    return errors.New("an argument is required")
  }
  // ...
  return nil
}

func main() {
  if err := run(os.Args[1:]); err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
  }
}
```

</td></tr>
</tbody></table>

Panic/recover — это не стратегия обработки ошибок. Программа должна вызывать panic только когда происходит что-то неисправимое, например разыменование nil. Исключением является инициализация программы: критические проблемы при запуске, которые должны прервать выполнение программы, могут вызывать panic.

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

Даже в тестах предпочитайте `t.Fatal` или `t.FailNow` вместо panic, чтобы гарантировать, что тест будет помечен как неуспешный.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
  panic("failed to set up test")
}
```

</td><td>

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

</td></tr>
</tbody></table>
