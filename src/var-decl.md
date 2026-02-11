# Объявления локальных переменных

Короткие объявления переменных (`:=`) должны использоваться, если переменная явно устанавливается в некоторое значение.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
var s = "foo"
```

</td><td>

```go
s := "foo"
```

</td></tr>
</tbody></table>

Однако есть случаи, когда значение по умолчанию яснее, когда используется ключевое слово `var`. [Объявление пустых слайсов], например.

  [Объявление пустых слайсов]: https://go.dev/wiki/CodeReviewComments#declaring-empty-slices

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func f(list []int) {
  filtered := []int{}
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td><td>

```go
func f(list []int) {
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td></tr>
</tbody></table>
