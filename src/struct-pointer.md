# Инициализация ссылок на структуры

Используйте `&T{}` вместо `new(T)` при инициализации ссылок на структуры, чтобы это было согласовано с инициализацией структур.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
sval := T{Name: "foo"}

// несогласованно
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>
