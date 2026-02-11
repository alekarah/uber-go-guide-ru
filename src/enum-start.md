# Начинайте enum с единицы

Стандартный способ введения перечислений в Go — объявить пользовательский тип и группу `const` с `iota`. Поскольку переменные имеют нулевое значение по умолчанию, вы обычно должны начинать ваши enum с ненулевого значения.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

</td></tr>
</tbody></table>

Существуют случаи, когда использование нулевого значения имеет смысл, например, когда нулевое значение является желаемым поведением по умолчанию.

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

<!-- TODO: section on String methods for enums -->
