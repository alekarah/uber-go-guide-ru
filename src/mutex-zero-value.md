# Мьютексы с нулевым значением валидны

Нулевое значение `sync.Mutex` и `sync.RWMutex` является валидным, поэтому вам практически никогда не нужен указатель на мьютекс.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
mu := new(sync.Mutex)
mu.Lock()
```

</td><td>

```go
var mu sync.Mutex
mu.Lock()
```

</td></tr>
</tbody></table>

Если вы используете структуру через указатель, то мьютекс должен быть полем-значением (не указателем) в ней. Не встраивайте мьютекс в структуру, даже если структура не экспортируется.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
type SMap struct {
  sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

</td><td>

```go
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

</td></tr>

<tr><td>

Поле `Mutex`, а также методы `Lock` и `Unlock` непреднамеренно становятся частью экспортируемого API `SMap`.

</td><td>

Мьютекс и его методы являются деталями реализации `SMap`, скрытыми от вызывающих сторон.

</td></tr>
</tbody></table>
