# Копирование слайсов и map на границах

Слайсы и map содержат указатели на лежащие в основе данные, поэтому будьте осторожны в сценариях, когда их нужно копировать.

## Получение слайсов и map

Имейте в виду, что пользователи могут изменить map или слайс, который вы получили в качестве аргумента, если вы сохраните ссылку на него.

<table>
<thead><tr><th>Плохо</th> <th>Хорошо</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Вы действительно хотели изменить d1.trips?
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// Теперь мы можем изменить trips[0], не затрагивая d1.trips.
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

## Возврат слайсов и map

Аналогично, будьте осторожны с изменениями пользователями map или слайсов, раскрывающими внутреннее состояние.

<table>
<thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Snapshot возвращает текущую статистику.
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot больше не защищён мьютексом, поэтому любой
// доступ к snapshot подвержен гонкам данных.
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Snapshot теперь является копией.
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>


