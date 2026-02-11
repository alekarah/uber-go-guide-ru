# Избегайте голых параметров

Голые параметры в вызовах функций могут ухудшить читаемость. Добавляйте комментарии в стиле C (`/* ... */`) для имён параметров, когда их значение не очевидно.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)
```

</td><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)
```

</td></tr>
</tbody></table>

Ещё лучше — замените голые типы `bool` на пользовательские типы для более читаемого и типобезопасного кода. Это позволит иметь больше, чем просто два состояния (true/false) для этого параметра в будущем.

```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady Status = iota + 1
  StatusDone
  // Возможно, у нас будет StatusInProgress в будущем.
)

func printInfo(name string, region Region, status Status)
```
