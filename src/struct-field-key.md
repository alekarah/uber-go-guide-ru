# Используйте имена полей для инициализации структур

Вы почти всегда должны указывать имена полей при инициализации структур. Это теперь контролируется [`go vet`].

  [`go vet`]: https://pkg.go.dev/cmd/vet

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

Исключение: Имена полей *могут* быть опущены в тестовых таблицах, когда полей 3 или меньше.

```go
tests := []struct{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```
