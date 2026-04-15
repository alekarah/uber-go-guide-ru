# Функциональные опции

Функциональные опции — это паттерн, в котором вы объявляете непрозрачный тип `Option`, который записывает информацию в некоторую внутреннюю структуру. Вы принимаете вариативное количество этих опций и действуете на основе полной информации, записанной опциями во внутренней структуре.

Используйте этот паттерн для необязательных аргументов в конструкторах и других публичных API, которые, как вы предполагаете, потребуют расширения, особенно если у вас уже есть три или более аргументов в этих функциях.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
// package db

func Open(
  addr string,
  cache bool,
  logger *zap.Logger
) (*Connection, error) {
  // ...
}
```

</td><td>

```go
// package db

type Option interface {
  // ...
}

func WithCache(c bool) Option {
  // ...
}

func WithLogger(log *zap.Logger) Option {
  // ...
}

// Open создаёт соединение.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  // ...
}
```

</td></tr>
<tr><td>

Параметры cache и logger всегда должны быть предоставлены, даже если пользователь хочет использовать значения по умолчанию.

```go
db.Open(addr, db.DefaultCache, zap.NewNop())
db.Open(addr, db.DefaultCache, log)
db.Open(addr, false /* cache */, zap.NewNop())
db.Open(addr, false /* cache */, log)
```

</td><td>

Опции предоставляются только при необходимости.

```go
db.Open(addr)
db.Open(addr, db.WithLogger(log))
db.Open(addr, db.WithCache(false))
db.Open(
  addr,
  db.WithCache(false),
  db.WithLogger(log),
)
```

</td></tr>
</tbody></table>

Наш предлагаемый способ реализации этого паттерна — с интерфейсом `Option`, который содержит неэкспортируемый метод, записывающий опции в неэкспортируемую структуру `options`.

```go
type options struct {
  cache  bool
  logger *zap.Logger
}

type Option interface {
  apply(*options)
}

type cacheOption bool

func (c cacheOption) apply(opts *options) {
  opts.cache = bool(c)
}

func WithCache(c bool) Option {
  return cacheOption(c)
}

type loggerOption struct {
  Log *zap.Logger
}

func (l loggerOption) apply(opts *options) {
  opts.logger = l.Log
}

func WithLogger(log *zap.Logger) Option {
  return loggerOption{Log: log}
}

// Open создаёт соединение.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    cache:  defaultCache,
    logger: zap.NewNop(),
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}
```

Обратите внимание, что существует способ реализации этого паттерна с замыканиями, но мы считаем, что паттерн выше более гибкий для авторов и проще в отладке и тестировании. В частности, он позволяет сравнивать опции друг с другом в тестах и моках, в отличие от замыканий, где это невозможно. Кроме того, он позволяет опциям реализовывать другие интерфейсы, включая `fmt.Stringer`, который позволяет получить читаемые для пользователя строковые представления опций.

См. также:

- [Self-referential functions and the design of options]
- [Functional options for friendly APIs]

  [Self-referential functions and the design of options]: https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html
  [Functional options for friendly APIs]: https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis

<!-- TODO: replace this with parameter structs and functional options, when to
use one vs other -->
