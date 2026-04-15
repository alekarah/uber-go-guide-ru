# Тестовые таблицы

Табличные тесты с [подтестами] могут быть полезным паттерном для написания тестов, чтобы избежать дублирования кода, когда основная логика теста повторяется.

Если систему нужно протестировать при _нескольких условиях_, где определённые части входных и выходных данных изменяются, следует использовать табличный тест для уменьшения избыточности и улучшения читаемости.

  [подтестами]: https://go.dev/blog/subtests

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestSplitHostPort(t *testing.T)

host, port, err := net.SplitHostPort("192.0.2.0:8000")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("192.0.2.0:http")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "http", port)

host, port, err = net.SplitHostPort(":8000")
require.NoError(t, err)
assert.Equal(t, "", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("1:8")
require.NoError(t, err)
assert.Equal(t, "1", host)
assert.Equal(t, "8", port)
```

</td><td>

```go
// func TestSplitHostPort(t *testing.T)

tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  {
    give:     "192.0.2.0:8000",
    wantHost: "192.0.2.0",
    wantPort: "8000",
  },
  {
    give:     "192.0.2.0:http",
    wantHost: "192.0.2.0",
    wantPort: "http",
  },
  {
    give:     ":8000",
    wantHost: "",
    wantPort: "8000",
  },
  {
    give:     "1:8",
    wantHost: "1",
    wantPort: "8",
  },
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    host, port, err := net.SplitHostPort(tt.give)
    require.NoError(t, err)
    assert.Equal(t, tt.wantHost, host)
    assert.Equal(t, tt.wantPort, port)
  })
}
```

</td></tr>
</tbody></table>

Тестовые таблицы упрощают добавление контекста к сообщениям об ошибках, уменьшают дублирующую логику и добавляют новые тестовые случаи.

Мы следуем соглашению, что слайс структур называется `tests`, а каждый тестовый случай — `tt`. Кроме того, мы рекомендуем явно указывать входные и выходные значения для каждого тестового случая с префиксами `give` и `want`.

```go
tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  // ...
}

for _, tt := range tests {
  // ...
}
```

## Избегайте излишней сложности в табличных тестах

Табличные тесты могут быть сложны для чтения и поддержки, если подтесты содержат условные утверждения или другую логику ветвления. Табличные тесты **НЕ ДОЛЖНЫ** использоваться, если внутри подтестов нужна сложная или условная логика (т.е. сложная логика внутри цикла `for`).

Большие, сложные табличные тесты вредят читаемости и поддерживаемости, потому что читатели тестов могут испытывать трудности с отладкой сбоев тестов.

Табличные тесты, подобные этому, должны быть разделены либо на несколько тестовых таблиц, либо на несколько отдельных функций `Test...`.

Некоторые идеалы, к которым следует стремиться:

* Фокусируйтесь на самой узкой единице поведения
* Минимизируйте "глубину теста" и избегайте условных утверждений (см. ниже)
* Убедитесь, что все поля таблицы используются во всех тестах
* Убедитесь, что вся логика теста выполняется для всех случаев таблицы

В этом контексте "глубина теста" означает "в рамках данного теста, количество последовательных утверждений, которые требуют выполнения предыдущих утверждений" (аналогично цикломатической сложности). Более узкие тесты имеют меньше зависимостей между утверждениями и, что более важно, эти утверждения с меньшей вероятностью будут условными по умолчанию.

Конкретно, табличные тесты могут стать запутанными и сложными для чтения, если они используют множественные пути ветвления (например, `shouldError`, `expectCall` и т.д.), используют много операторов `if` для конкретных ожиданий моков (например, `shouldCallFoo`) или размещают функции внутри таблицы (например, `setupMocks func(*FooMock)`).

Однако при тестировании поведения, которое изменяется только на основе изменённых входных данных, может быть предпочтительнее группировать похожие случаи вместе в табличном тесте, чтобы лучше проиллюстрировать, как поведение изменяется по всем входным данным, вместо того чтобы разделять сравнимые единицы на отдельные тесты и делать их сложнее для сравнения и противопоставления.

Если тело теста короткое и прямолинейное, допустимо иметь один путь ветвления для случаев успеха против случаев сбоя с полем таблицы вроде `shouldErr` для указания ожиданий ошибок.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
func TestComplicatedTable(t *testing.T) {
  tests := []struct {
    give          string
    want          string
    wantErr       error
    shouldCallX   bool
    shouldCallY   bool
    giveXResponse string
    giveXErr      error
    giveYResponse string
    giveYErr      error
  }{
    // ...
  }

  for _, tt := range tests {
    t.Run(tt.give, func(t *testing.T) {
      // настройка моков
      ctrl := gomock.NewController(t)
      xMock := xmock.NewMockX(ctrl)
      if tt.shouldCallX {
        xMock.EXPECT().Call().Return(
          tt.giveXResponse, tt.giveXErr,
        )
      }
      yMock := ymock.NewMockY(ctrl)
      if tt.shouldCallY {
        yMock.EXPECT().Call().Return(
          tt.giveYResponse, tt.giveYErr,
        )
      }

      got, err := DoComplexThing(tt.give, xMock, yMock)

      // проверка результатов
      if tt.wantErr != nil {
        require.EqualError(t, err, tt.wantErr)
        return
      }
      require.NoError(t, err)
      assert.Equal(t, want, got)
    })
  }
}
```

</td><td>

```go
func TestShouldCallX(t *testing.T) {
  // настройка моков
  ctrl := gomock.NewController(t)
  xMock := xmock.NewMockX(ctrl)
  xMock.EXPECT().Call().Return("XResponse", nil)

  yMock := ymock.NewMockY(ctrl)

  got, err := DoComplexThing("inputX", xMock, yMock)

  require.NoError(t, err)
  assert.Equal(t, "want", got)
}

func TestShouldCallYAndFail(t *testing.T) {
  // настройка моков
  ctrl := gomock.NewController(t)
  xMock := xmock.NewMockX(ctrl)

  yMock := ymock.NewMockY(ctrl)
  yMock.EXPECT().Call().Return("YResponse", nil)

  _, err := DoComplexThing("inputY", xMock, yMock)
  assert.EqualError(t, err, "Y failed")
}
```
</td></tr>
</tbody></table>

Эта сложность затрудняет изменение, понимание и доказательство корректности теста.

Хотя строгих рекомендаций нет, читаемость и поддерживаемость всегда должны быть в приоритете при выборе между табличными тестами и отдельными тестами для множественных входов/выходов системы.

## Параллельные тесты

Параллельные тесты, как и некоторые специализированные циклы (например, те, которые порождают горутины или захватывают ссылки как часть тела цикла), должны явно присваивать переменные цикла в области видимости цикла, чтобы гарантировать, что они содержат ожидаемые значения.

```go
tests := []struct{
  give string
  // ...
}{
  // ...
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    t.Parallel()
    // ...
  })
}
```

В примере выше мы должны объявить переменную `tt` с областью видимости итерации цикла из-за использования `t.Parallel()` ниже. Если мы этого не сделаем, большинство или все тесты получат неожиданное значение для `tt` или значение, которое изменяется во время их выполнения.

<!-- TODO: Explain how to use _test packages. -->
