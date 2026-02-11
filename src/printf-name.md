# Именование функций в стиле Printf

Когда вы объявляете функцию в стиле `Printf`, убедитесь, что `go vet` может её обнаружить и проверить форматную строку.

Это означает, что вы должны использовать предопределённые имена функций в стиле `Printf`, если возможно. `go vet` будет проверять их по умолчанию. См. [Printf family] для дополнительной информации.

  [Printf family]: https://pkg.go.dev/cmd/vet#hdr-Printf_family

Если использование предопределённых имён невозможно, завершайте выбранное имя буквой f: `Wrapf`, а не `Wrap`. `go vet` можно попросить проверить конкретные имена в стиле `Printf`, но они должны заканчиваться на f.

```shell
go vet -printfuncs=wrapf,statusf
```

См. также [go vet: Printf family check].

  [go vet: Printf family check]: https://kuzminva.wordpress.com/2017/11/07/go-vet-printf-family-check/
