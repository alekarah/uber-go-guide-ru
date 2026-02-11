# Группировка и порядок функций

- Функции должны быть отсортированы примерно в порядке вызова.
- Функции в файле должны быть сгруппированы по приёмнику.

Следовательно, экспортируемые функции должны появляться первыми в файле, после определений `struct`, `const`, `var`.

`newXYZ()`/`NewXYZ()` может появиться после определения типа, но до остальных методов приёмника.

Поскольку функции сгруппированы по приёмнику, простые утилитарные функции должны появляться ближе к концу файла.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>
