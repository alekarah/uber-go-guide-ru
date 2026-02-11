# Именование ошибок

Для значений ошибок, хранящихся как глобальные переменные,
используйте префикс `Err` или `err` в зависимости от того, экспортируются ли они.
Эта рекомендация имеет приоритет над [Префикс _ для неэкспортируемых глобальных переменных](global-name.md).

```go
var (
  // Следующие две ошибки экспортируются,
  // чтобы пользователи этого пакета могли сопоставить их
  // с помощью errors.Is.

  ErrBrokenLink = errors.New("link is broken")
  ErrCouldNotOpen = errors.New("could not open")

  // Эта ошибка не экспортируется, потому что
  // мы не хотим делать её частью нашего публичного API.
  // Мы всё ещё можем использовать её внутри пакета
  // с помощью errors.Is.

  errNotFound = errors.New("not found")
)
```

Для пользовательских типов ошибок используйте суффикс `Error`.

```go
// Аналогично, эта ошибка экспортируется,
// чтобы пользователи этого пакета могли сопоставить её
// с помощью errors.As.

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

// А эта ошибка не экспортируется, потому что
// мы не хотим делать её частью публичного API.
// Мы всё ещё можем использовать её внутри пакета
// с помощью errors.As.

type resolveError struct {
  Path string
}

func (e *resolveError) Error() string {
  return fmt.Sprintf("resolve %q", e.Path)
}
```
