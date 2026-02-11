# nil является валидным слайсом

`nil` является валидным слайсом длиной 0. Это означает, что:

- Вы не должны возвращать слайс нулевой длины явно. Возвращайте `nil` вместо этого.

  <table>
  <thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  if x == "" {
    return []int{}
  }
  ```

  </td><td>

  ```go
  if x == "" {
    return nil
  }
  ```

  </td></tr>
  </tbody></table>

- Чтобы проверить, пуст ли слайс, всегда используйте `len(s) == 0`. Не проверяйте на `nil`.

  <table>
  <thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  func isEmpty(s []string) bool {
    return s == nil
  }
  ```

  </td><td>

  ```go
  func isEmpty(s []string) bool {
    return len(s) == 0
  }
  ```

  </td></tr>
  </tbody></table>

- Нулевое значение (слайс, объявленный с `var`) можно использовать немедленно без `make()`.

  <table>
  <thead><tr><th>Плохо</th><th>Хорошо</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  nums := []int{}
  // или, nums := make([]int)

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td><td>

  ```go
  var nums []int

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td></tr>
  </tbody></table>

Помните, что хотя это валидный слайс, nil слайс не эквивалентен выделенному слайсу длиной 0 — один является nil, а другой нет — и эти два могут обрабатываться по-разному в различных ситуациях (например, при сериализации).
