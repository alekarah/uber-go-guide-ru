# Выход из программы в Main

Go-программы используют [`os.Exit`] или [`log.Fatal*`] для немедленного выхода. (Panic — это не хороший способ выхода из программ, пожалуйста [не используйте panic](panic.md).)

  [`os.Exit`]: https://pkg.go.dev/os#Exit
  [`log.Fatal*`]: https://pkg.go.dev/log#Fatal

Вызывайте `os.Exit` или `log.Fatal*` **только в `main()`**. Все остальные функции должны возвращать ошибки для сигнализации о сбое.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func main() {
  body := readFile(path)
  fmt.Println(body)
}

func readFile(path string) string {
  f, err := os.Open(path)
  if err != nil {
    log.Fatal(err)
  }

  b, err := io.ReadAll(f)
  if err != nil {
    log.Fatal(err)
  }

  return string(b)
}
```

</td><td>

```go
func main() {
  body, err := readFile(path)
  if err != nil {
    log.Fatal(err)
  }
  fmt.Println(body)
}

func readFile(path string) (string, error) {
  f, err := os.Open(path)
  if err != nil {
    return "", err
  }

  b, err := io.ReadAll(f)
  if err != nil {
    return "", err
  }

  return string(b), nil
}
```

</td></tr>
</tbody></table>

Обоснование: Программы с несколькими функциями, которые выходят из программы, представляют несколько проблем:

- Неочевидный поток управления: Любая функция может выйти из программы, поэтому становится сложно рассуждать о потоке управления.
- Сложность тестирования: Функция, которая выходит из программы, также выйдет из теста, вызывающего её. Это усложняет тестирование функции и создаёт риск пропуска других тестов, которые ещё не были запущены `go test`.
- Пропущенная очистка: Когда функция выходит из программы, она пропускает вызовы функций, поставленные в очередь с помощью выражений `defer`. Это добавляет риск пропуска важных задач очистки.
