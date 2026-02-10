# Проверка соответствия интерфейсу

Проверяйте соответствие интерфейсу во время компиляции, где это уместно. Это включает:

- Экспортируемые типы, которые должны реализовывать определённые интерфейсы как часть их API-контракта
- Экспортируемые или неэкспортируемые типы, которые являются частью коллекции типов, реализующих один и тот же интерфейс
- Другие случаи, когда нарушение интерфейса приведёт к проблемам у пользователей

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
type Handler struct {
  // ...
}



func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  ...
}
```

</td><td>

```go
type Handler struct {
  // ...
}

var _ http.Handler = (*Handler)(nil)

func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

</td></tr>
</tbody></table>

Выражение `var _ http.Handler = (*Handler)(nil)` не скомпилируется, если `*Handler` когда-либо перестанет соответствовать интерфейсу `http.Handler`.

Правая часть присваивания должна быть нулевым значением проверяемого типа. Это `nil` для типов-указателей (таких как `*Handler`), слайсов и map, и пустая структура для типов-структур.

```go
type LogHandler struct {
  h   http.Handler
  log *zap.Logger
}

var _ http.Handler = LogHandler{}

func (h LogHandler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```
