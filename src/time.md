# Используйте `"time"` для работы со временем

Время — это сложно. Неправильные предположения, часто делаемые о времени, включают следующие.

1. День состоит из 24 часов
2. Час состоит из 60 минут
3. Неделя состоит из 7 дней
4. Год состоит из 365 дней
5. [И многое другое](https://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time)

Например, *1* означает, что добавление 24 часов к моменту времени не всегда даст новый календарный день.

Поэтому всегда используйте пакет [`"time"`] при работе со временем, потому что он помогает справляться с этими неправильными предположениями более безопасным и точным образом.

  [`"time"`]: https://pkg.go.dev/time

## Используйте `time.Time` для моментов времени

Используйте [`time.Time`] при работе с моментами времени, и методы `time.Time` при сравнении, сложении или вычитании времени.

  [`time.Time`]: https://pkg.go.dev/time#Time

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func isActive(now, start, stop int) bool {
  return start <= now && now < stop
}
```

</td><td>

```go
func isActive(now, start, stop time.Time) bool {
  return (start.Before(now) || start.Equal(now)) && now.Before(stop)
}
```

</td></tr>
</tbody></table>

## Используйте `time.Duration` для периодов времени

Используйте [`time.Duration`] при работе с периодами времени.

  [`time.Duration`]: https://pkg.go.dev/time#Duration

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func poll(delay int) {
  for {
    // ...
    time.Sleep(time.Duration(delay) * time.Millisecond)
  }
}

poll(10) // это секунды или миллисекунды?
```

</td><td>

```go
func poll(delay time.Duration) {
  for {
    // ...
    time.Sleep(delay)
  }
}

poll(10*time.Second)
```

</td></tr>
</tbody></table>

Возвращаясь к примеру добавления 24 часов к моменту времени, метод, который мы используем для добавления времени, зависит от намерения. Если мы хотим то же время суток, но на следующий календарный день, мы должны использовать [`Time.AddDate`]. Однако, если мы хотим момент времени, гарантированно находящийся через 24 часа после предыдущего времени, мы должны использовать [`Time.Add`].

  [`Time.AddDate`]: https://pkg.go.dev/time#Time.AddDate
  [`Time.Add`]: https://pkg.go.dev/time#Time.Add

```go
newDay := t.AddDate(0 /* years */, 0 /* months */, 1 /* days */)
maybeNewDay := t.Add(24 * time.Hour)
```

## Используйте `time.Time` и `time.Duration` с внешними системами

Используйте `time.Duration` и `time.Time` при взаимодействии с внешними системами, когда это возможно. Например:

- Флаги командной строки: [`flag`] поддерживает `time.Duration` через [`time.ParseDuration`]
- JSON: [`encoding/json`] поддерживает кодирование `time.Time` как строки [RFC 3339] через свой метод [`UnmarshalJSON` method]
- SQL: [`database/sql`] поддерживает преобразование колонок `DATETIME` или `TIMESTAMP` в `time.Time` и обратно, если базовый драйвер поддерживает это
- YAML: [`gopkg.in/yaml.v2`] поддерживает `time.Time` как строку [RFC 3339], и `time.Duration` через [`time.ParseDuration`].

  [`flag`]: https://pkg.go.dev/flag
  [`time.ParseDuration`]: https://pkg.go.dev/time#ParseDuration
  [`encoding/json`]: https://pkg.go.dev/encoding/json
  [RFC 3339]: https://tools.ietf.org/html/rfc3339
  [`UnmarshalJSON` method]: https://pkg.go.dev/time#Time.UnmarshalJSON
  [`database/sql`]: https://pkg.go.dev/database/sql
  [`gopkg.in/yaml.v2`]: https://pkg.go.dev/gopkg.in/yaml.v2

Когда невозможно использовать `time.Duration` в этих взаимодействиях, используйте `int` или `float64` и включите единицу измерения в имя поля.

Например, поскольку `encoding/json` не поддерживает `time.Duration`, единица измерения включается в имя поля.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
// {"interval": 2}
type Config struct {
  Interval int `json:"interval"`
}
```

</td><td>

```go
// {"intervalMillis": 2000}
type Config struct {
  IntervalMillis int `json:"intervalMillis"`
}
```

</td></tr>
</tbody></table>

Когда невозможно использовать `time.Time` в этих взаимодействиях, если не согласована альтернатива, используйте `string` и форматируйте временные метки, как определено в [RFC 3339]. Этот формат используется по умолчанию методом [`Time.UnmarshalText`] и доступен для использования в `Time.Format` и `time.Parse` через [`time.RFC3339`].

  [`Time.UnmarshalText`]: https://pkg.go.dev/time#Time.UnmarshalText
  [`time.RFC3339`]: https://pkg.go.dev/time#RFC3339

Хотя на практике это обычно не проблема, имейте в виду, что пакет `"time"` не поддерживает парсинг временных меток с високосными секундами ([8728]), и также не учитывает високосные секунды в вычислениях ([15190]). Если вы сравниваете два момента времени, разница не будет включать високосные секунды, которые могли произойти между этими двумя моментами.

  [8728]: https://github.com/golang/go/issues/8728
  [15190]: https://github.com/golang/go/issues/15190
