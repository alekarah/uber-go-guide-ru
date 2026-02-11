# Используйте go.uber.org/atomic

Атомарные операции с пакетом [sync/atomic] работают с сырыми типами (`int32`, `int64` и т.д.), поэтому легко забыть использовать атомарную операцию для чтения или изменения переменных.

[go.uber.org/atomic] добавляет типобезопасность этим операциям, скрывая базовый тип. Кроме того, он включает удобный тип `atomic.Bool`.

  [go.uber.org/atomic]: https://pkg.go.dev/go.uber.org/atomic
  [sync/atomic]: https://pkg.go.dev/sync/atomic

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
type foo struct {
  running int32  // atomic
}

func (f* foo) start() {
  if atomic.SwapInt32(&f.running, 1) == 1 {
     // уже запущен…
     return
  }
  // запускаем Foo
}

func (f *foo) isRunning() bool {
  return f.running == 1  // гонка!
}
```

</td><td>

```go
type foo struct {
  running atomic.Bool
}

func (f *foo) start() {
  if f.running.Swap(true) {
     // уже запущен…
     return
  }
  // запускаем Foo
}

func (f *foo) isRunning() bool {
  return f.running.Load()
}
```

</td></tr>
</tbody></table>
