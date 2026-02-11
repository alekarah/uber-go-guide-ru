# Форматные строки вне Printf

Если вы объявляете форматные строки для функций в стиле `Printf` вне строкового литерала, сделайте их значениями `const`.

Это помогает `go vet` выполнять статический анализ форматной строки.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td><td>

```go
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td></tr>
</tbody></table>
