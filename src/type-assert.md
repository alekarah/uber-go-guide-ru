# Обрабатывайте ошибки приведения типов

Однозначная форма возврата [приведения типов] вызовет panic при неправильном типе. Поэтому всегда используйте идиому "comma ok".

  [приведения типов]: https://go.dev/ref/spec#Type_assertions

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // корректно обрабатываем ошибку
}
```

</td></tr>
</tbody></table>

<!-- TODO: There are a few situations where the single assignment form is
fine. -->
