**Цель:** Проверить глубокое понимание работы со структурами в Go, включая их организацию в памяти, копирование, сравнение, использование тегов и выбор типа ресивера для методов.

**Ключевые аспекты для проверки:**

1.  **Копирование структур:** Понимание семантики присваивания (value type), как копируются поля (включая слайсы, карты, указатели).
2.  **Встраивание (Embedding) vs Композиция:** Разница в доступе к полям/методам, продвижение методов, разрешение конфликтов имен.
3.  **Сравнение структур:** Когда структуры сравнимы (`==`), а когда нет.
4.  **Теги полей:** Использование для сериализации/десериализации (`json`, `db`), именование, опции (`omitempty`, `-`).
5.  **Методы с value/pointer receiver:** Разница в поведении (модификация), производительность, критерии выбора, работа с `nil` ресивером, влияние на реализацию интерфейсов.

**План задач (14 штук):**

1.  **Задача 1 (Копирование):** Что выведет код? Фокус на копировании простой структуры.
2.  **Задача 2 (Копирование):** Что выведет код? Фокус на копировании структуры с полем-слайсом.
3.  **Задача 3 (Копирование):** Что выведет код? Фокус на копировании структуры с полем-указателем.
4.  **Задача 4 (Методы):** Что выведет код? Использование value receiver.
5.  **Задача 5 (Методы):** Что выведет код? Использование pointer receiver.
6.  **Задача 6 (Методы):** Что выведет код? Вызов метода с pointer receiver на `nil` указателе.
7.  **Задача 7 (Методы):** Будет ли код компилироваться и работать? Если да, что выведет? Вызов методов с разными типами ресиверов на значениях и указателях.
8.  **Задача 8 (Встраивание):** Что выведет код? Демонстрация продвижения методов.
9.  **Задача 9 (Встраивание):** Что выведет код? Конфликт имен полей при встраивании.
10. **Задача 10 (Встраивание vs Композиция):** Что выведет код? В чем разница между `OuterEmbedding` и `OuterComposition` при доступе к полям/методам `Inner`?
11. **Задача 11 (Сравнение):** Можно ли сравнить переменные `p1` и `p2`? Почему? Что выведет код?
12. **Задача 12 (Сравнение):** Можно ли сравнить переменные `c1` и `c2`? Почему?
13. **Задача 13 (Теги):** Какой JSON будет сгенерирован для `c1` и `c2`?
14. **Задача 14 (Теги):** Какой JSON будет сгенерирован для `u`? Какие поля будут включены?

---

**Задачи:**

**Задача 1: Копирование простой структуры**

```go
package main

import "fmt"

type Point struct {
	X int
	Y int
}

func main() {
	p1 := Point{X: 10, Y: 20}
	p2 := p1 // Копирование структуры

	p2.X = 15

	fmt.Println(p1)
	fmt.Println(p2)
}

// Что выведет программа?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод:**
  ```
  {10 20}
  {15 20}
  ```
  **Объяснение:**
  Структуры в Go являются типами-значениями (value types). При присваивании `p2 := p1` создается полная копия структуры `p1`. Изменение поля `X` в копии `p2` не затрагивает оригинальную структуру `p1`.
</details>

---

**Задача 2: Копирование структуры со слайсом**

```go
package main

import "fmt"

type Data struct {
	ID      int
	Payload []string
}

func main() {
	d1 := Data{
		ID:      1,
		Payload: []string{"a", "b"},
	}
	d2 := d1 // Копирование структуры

	d2.ID = 2
	d2.Payload[0] = "x" // Изменение элемента слайса

	fmt.Println(d1)
	fmt.Println(d2)
	fmt.Printf("d1.Payload address: %p\n", d1.Payload)
	fmt.Printf("d2.Payload address: %p\n", d2.Payload)
}

// Что выведет программа?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод (адреса могут отличаться):**
  ```
  {1 [x b]}
  {2 [x b]}
  d1.Payload address: 0x...
  d2.Payload address: 0x...
  ```
  (Адреса будут одинаковыми)

  **Объяснение:**
  При копировании структуры `d1` в `d2` поле `ID` (тип `int`) копируется по значению. Однако поле `Payload` (тип `[]string`, слайс) копируется как *заголовок* слайса. Заголовок слайса содержит указатель на базовый массив, длину и емкость. Обе переменные `d1.Payload` и `d2.Payload` после копирования указывают на *один и тот же* базовый массив в памяти. Поэтому изменение элемента через `d2.Payload[0]` видно и через `d1.Payload`. Это пример *мелкого копирования* (shallow copy). Адреса самих слайсов (их заголовков) будут одинаковыми, так как они указывают на один массив.
</details>

---

**Задача 3: Копирование структуры с указателем**

```go
package main

import "fmt"

type Config struct {
	Name  string
	Value *int
}

func main() {
	v := 100
	c1 := Config{Name: "timeout", Value: &v}
	c2 := c1 // Копирование структуры

	c2.Name = "retries"
	*c2.Value = 200 // Изменение значения по указателю

	fmt.Printf("c1: %+v, c1.Value: %d\n", c1, *c1.Value)
	fmt.Printf("c2: %+v, c2.Value: %d\n", c2, *c2.Value)
}

// Что выведет программа?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод:**
  ```
  c1: {Name:timeout Value:0x...}, c1.Value: 200
  c2: {Name:retries Value:0x...}, c2.Value: 200
  ```
  (Адреса `Value` будут одинаковыми)

  **Объяснение:**
  Поле `Name` (тип `string`) копируется по значению. Поле `Value` (тип `*int`, указатель) также копируется по значению, то есть копируется сам *адрес*, на который указывает `c1.Value`. В результате `c1.Value` и `c2.Value` указывают на одну и ту же переменную `v` в памяти. Изменение значения по этому адресу через `*c2.Value = 200` изменяет значение переменной `v`, что отражается при доступе через `*c1.Value`.
</details>

---

**Задача 4: Метод с value receiver**

```go
package main

import "fmt"

type Counter struct {
	Value int
}

// Метод с value receiver
func (c Counter) Increment() {
	c.Value++
	fmt.Printf("Inside Increment (value receiver): %d\n", c.Value)
}

func main() {
	counter := Counter{Value: 5}
	counter.Increment()
	fmt.Printf("Outside: %d\n", counter.Value)
}

// Что выведет программа?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод:**
  ```
  Inside Increment (value receiver): 6
  Outside: 5
  ```
  **Объяснение:**
  Метод `Increment` имеет value receiver (`c Counter`). Это означает, что метод работает с *копией* структуры `counter`. Изменение `c.Value` внутри метода затрагивает только эту копию. Оригинальная структура `counter` в функции `main` остается неизменной.
</details>

---

**Задача 5: Метод с pointer receiver**

```go
package main

import "fmt"

type Counter struct {
	Value int
}

// Метод с pointer receiver
func (c *Counter) Increment() {
	c.Value++
	fmt.Printf("Inside Increment (pointer receiver): %d\n", c.Value)
}

func main() {
	counter := Counter{Value: 5}
	counter.Increment()
	fmt.Printf("Outside: %d\n", counter.Value)

	pCounter := &Counter{Value: 10}
	pCounter.Increment()
	fmt.Printf("Outside (pointer): %d\n", pCounter.Value)
}

// Что выведет программа?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод:**
  ```
  Inside Increment (pointer receiver): 6
  Outside: 6
  Inside Increment (pointer receiver): 11
  Outside (pointer): 11
  ```
  **Объяснение:**
  Метод `Increment` имеет pointer receiver (`c *Counter`). Это означает, что метод работает с *указателем* на оригинальную структуру. Изменения `c.Value` (что эквивалентно `(*c).Value++`) происходят непосредственно в оригинальной структуре, на которую указывает `c`.
  Go предоставляет синтаксический сахар: можно вызывать метод с pointer receiver на значении (`counter.Increment()`), и Go автоматически возьмет адрес (`(&counter).Increment()`), если значение адресуемо. Аналогично, можно вызвать метод с value receiver на указателе (`pCounter.ValueMethod()`), и Go автоматически разыменует указатель (`(*pCounter).ValueMethod()`).
</details>

---

**Задача 6: Вызов метода с pointer receiver на nil указателе**

```go
package main

import "fmt"

type Greeter struct {
	Message string
}

func (g *Greeter) Greet() {
	if g == nil {
		fmt.Println("Greeter is nil")
		return
	}
	fmt.Println(g.Message)
}

func main() {
	var g1 *Greeter // g1 is nil
	g1.Greet()

	g2 := &Greeter{Message: "Hello"}
	g2.Greet()
}

// Что выведет программа? Не будет ли паники?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод:**
  ```
  Greeter is nil
  Hello
  ```
  **Объяснение:**
  Вызов метода с pointer receiver на `nil` указателе *не* вызывает панику немедленно. Метод вызывается, и внутри него ресивер `g` будет иметь значение `nil`. Паника произойдет только если внутри метода попытаться получить доступ к полям `nil` ресивера (например, `g.Message`) без предварительной проверки на `nil`. В данном коде проверка `if g == nil` корректно обрабатывает этот случай. Это полезно для реализации методов с "нулевым" поведением по умолчанию.
</details>

---

**Задача 7: Неявное взятие адреса/разыменование при вызове методов**

```go
package main

import "fmt"

type Data struct {
	Value int
}

func (d Data) GetValue() int {
	return d.Value
}

func (d *Data) SetValue(v int) {
	d.Value = v
}

func main() {
	data := Data{Value: 1}
	pData := &Data{Value: 2}

	fmt.Println(data.GetValue())
	data.SetValue(10)
	fmt.Println(pData.GetValue())
	pData.SetValue(20)

	fmt.Println(data.Value)
	fmt.Println(pData.Value)
}

// Будет ли код компилироваться и работать? Если да, что он выведет?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод:**
  ```
  1
  2
  10
  20
  ```
  **Объяснение:**
  Да, код будет компилироваться и работать. Go делает удобные преобразования при вызове методов:
  1.  `data.GetValue()`: Вызов метода с value receiver на значении - работает как есть.
  2.  `data.SetValue(10)`: Вызов метода с pointer receiver на значении. Так как `data` - это адресуемая переменная, Go автоматически берет её адрес: `(&data).SetValue(10)`.
  3.  `pData.GetValue()`: Вызов метода с value receiver на указателе. Go автоматически разыменовывает указатель: `(*pData).GetValue()`.
  4.  `pData.SetValue(20)`: Вызов метода с pointer receiver на указателе - работает как есть.

  Это упрощает работу, но важно понимать, что происходит "под капотом", особенно когда речь идет об изменении состояния (`SetValue`).
</details>

---

**Задача 8: Встраивание (Embedding) - Продвижение методов**

```go
package main

import "fmt"

type Logger struct {
	Level string
}

func (l *Logger) Log(message string) {
	fmt.Printf("[%s] %s\n", l.Level, message)
}

type Service struct {
	Name string
	*Logger // Встраивание указателя на Logger
}

func main() {
	logger := &Logger{Level: "INFO"}
	s := Service{
		Name:   "MyService",
		Logger: logger,
	}

	s.Log("Service started")
}

// Что выведет программа?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод:**
  ```
  [INFO] Service started
  ```
  **Объяснение:**
  При встраивании типа (`*Logger` в `Service`) его поля и методы становятся доступными так, как если бы они были объявлены непосредственно во внешней структуре (`Service`). Это называется "продвижением" (promotion). Вызов `s.Log(...)` эквивалентен `s.Logger.Log(...)`. Обратите внимание, что ресивером для метода `Log` остается `*Logger`, поэтому он получает правильный `Level`.
</details>

---

**Задача 9: Встраивание - Конфликт имен**

```go
package main

import "fmt"

type Base struct {
	ID string
}

func (b Base) GetID() string {
	return "BaseID: " + b.ID
}

type Derived struct {
	Base // Встраивание Base
	ID   int
}

func main() {
	d := Derived{
		Base: Base{ID: "abc"},
		ID:   123,
	}

	fmt.Println(d.ID)
	fmt.Println(d.Base.ID)
	// fmt.Println(d.GetID())
}

// Что выведет программа? Что произойдет, если раскомментировать последнюю строку?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод:**
  ```
  123
  abc
  ```
  **Объяснение:**
  При конфликте имен между полем внешней структуры (`Derived.ID`) и полем встроенной структуры (`Base.ID`), приоритет имеет поле внешней структуры. Поэтому `d.ID` обращается к полю `ID` типа `int` структуры `Derived`. Чтобы получить доступ к полю `ID` встроенной структуры `Base`, нужно использовать явное указание: `d.Base.ID`.

  **Если раскомментировать `fmt.Println(d.GetID())`:**
  Эта строка скомпилируется и выполнится. Вывод будет:
  ```
  BaseID: abc
  ```
  Метод `GetID` принадлежит `Base`, и он был "продвинут" до `Derived`. Он по-прежнему оперирует полями `Base`, поэтому использует `d.Base.ID`. Конфликта имен методов здесь нет.
</details>

---

**Задача 10: Встраивание vs Композиция**

```go
package main

import "fmt"

type Inner struct {
	Value string
}

func (i Inner) Get() string {
	return i.Value
}

type OuterEmbedding struct {
	Inner // Встраивание
}

type OuterComposition struct {
	InnerComp Inner // Композиция
}

func main() {
	oe := OuterEmbedding{Inner: Inner{Value: "Embedded"}}
	oc := OuterComposition{InnerComp: Inner{Value: "Composed"}}

	fmt.Println("Embedding:")
	fmt.Println(oe.Value)
	fmt.Println(oe.Get())

	fmt.Println("\nComposition:")
	fmt.Println(oc.InnerComp.Value)
	fmt.Println(oc.InnerComp.Get())
}

// Что выведет код? В чем принципиальная разница в доступе к полям и методам Inner для oe и oc?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод:**
  ```
  Embedding:
  Embedded
  Embedded

  Composition:
  Composed
  Composed
  ```
  **Объяснение:**
  *   **Встраивание (Embedding):** Поля и методы встроенного типа (`Inner`) "продвигаются" во внешнюю структуру (`OuterEmbedding`). К ним можно обращаться напрямую `oe.Value` и `oe.Get()`, как если бы они были объявлены в `OuterEmbedding`. Это похоже на наследование (хотя и не является им в классическом ООП смысле).
  *   **Композиция (Composition):** Внешняя структура (`OuterComposition`) содержит поле типа `Inner` (`InnerComp`). Для доступа к полям и методам `Inner` необходимо явно указывать имя поля: `oc.InnerComp.Value` и `oc.InnerComp.Get()`. Это более явный способ указать, что `OuterComposition` *содержит* `Inner`, но не *является* им.

  Встраивание удобно для расширения функциональности и следования принципу DRY, но может привести к менее явным зависимостям. Композиция обычно считается более предпочтительной из-за явности ("предпочитайте композицию наследованию/встраиванию").
</details>

---

**Задача 11: Сравнение структур (Comparable)**

```go
package main

import "fmt"

type Point struct {
	X, Y int
}

func main() {
	p1 := Point{X: 10, Y: 20}
	p2 := Point{X: 10, Y: 20}
	p3 := Point{X: 5, Y: 5}

	fmt.Println("p1 == p2:", p1 == p2)
	fmt.Println("p1 == p3:", p1 == p3)
}

// Можно ли сравнивать p1 и p2? Что выведет код?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод:**
  ```
  p1 == p2: true
  p1 == p3: false
  ```
  **Объяснение:**
  Да, структуры `p1` и `p2` можно сравнивать с помощью оператора `==` (и `!=`). Структуры сравнимы, если *все* их поля сравнимы. Тип `int` является сравнимым. Сравнение происходит по-полюсно: сравниваются значения `X`, затем значения `Y`. Структуры равны, если все их соответствующие поля равны.
</details>

---

**Задача 12: Сравнение структур (Non-Comparable)**

```go
package main

// import "fmt"

type DataContainer struct {
	ID   int
	Data []string
}

func main() {
	// c1 := DataContainer{ID: 1, Data: []string{"a"}}
	// c2 := DataContainer{ID: 1, Data: []string{"a"}}

	// fmt.Println(c1 == c2) // Будет ли эта строка компилироваться?
}

// Можно ли сравнивать переменные типа DataContainer с помощью оператора ==? Почему?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Ответ:**
  Нет, сравнивать переменные типа `DataContainer` с помощью `==` **нельзя**.

  **Объяснение:**
  Если раскомментировать строки, код **не скомпилируется**. Ошибка будет примерно такой: `invalid operation: c1 == c2 (struct containing []string cannot be compared)`.
  Причина в том, что структуры сравнимы только если *все* их поля сравнимы. Типы `slice`, `map` и `function` не являются сравнимыми в Go. Поскольку `DataContainer` содержит поле `Data` типа `[]string` (слайс), вся структура становится не сравнимой.
</details>

---

**Задача 13: Теги полей для JSON**

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Config struct {
	ServerAddress string `json:"server_addr"`
	Timeout       int    `json:"timeout_ms,omitempty"`
	MaxConns      int    `json:",omitempty"`
	DebugMode     bool
}

func main() {
	c1 := Config{
		ServerAddress: "localhost:8080",
		Timeout:       5000,
		DebugMode:     true,
	}

	c2 := Config{
		ServerAddress: "192.168.1.100:9000",
		Timeout:       0,
		MaxConns:      100,
		DebugMode:     false,
	}

	encoder := json.NewEncoder(os.Stdout)
	encoder.SetIndent("", "  ") // Для красивого вывода

	fmt.Println("Config 1:")
	encoder.Encode(c1)

	fmt.Println("\nConfig 2:")
	encoder.Encode(c2)
}

// Какой JSON будет сгенерирован для c1 и c2?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод JSON:**
  ```json
  Config 1:
  {
    "server_addr": "localhost:8080",
    "timeout_ms": 5000,
    "DebugMode": true
  }

  Config 2:
  {
    "server_addr": "192.168.1.100:9000",
    "MaxConns": 100,
    "DebugMode": false
  }
  ```
  **Объяснение:**
  *   Тег `json:"server_addr"` переименовывает поле `ServerAddress` в `server_addr` в JSON.
  *   Тег `json:"timeout_ms,omitempty"` переименовывает `Timeout` в `timeout_ms`. Опция `omitempty` означает, что поле будет включено в JSON, только если его значение *не* является нулевым для его типа (0 для `int`).
      *   В `c1`, `Timeout` равен 5000 (не 0), поэтому поле `timeout_ms` включается.
      *   В `c2`, `Timeout` равен 0, поэтому поле `timeout_ms` *опускается*.
  *   Тег `json:",omitempty"` для `MaxConns` не меняет имя поля, но добавляет опцию `omitempty`.
      *   В `c1`, `MaxConns` равен 0 (нулевое значение для int), поэтому поле `MaxConns` *опускается*.
      *   В `c2`, `MaxConns` равен 100 (не 0), поэтому поле `MaxConns` включается.
  *   Поле `DebugMode` не имеет тега `json`, поэтому используется имя поля как есть. Опция `omitempty` здесь не указана, поэтому поле включается всегда (даже если значение `false`, что является нулевым для `bool`).
</details>

---

**Задача 14: Теги полей для JSON - Игнорирование**

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type User struct {
	ID       int    `json:"id"`
	Username string `json:"username"`
	Password string `json:"-"`
	apiKey   string // неэкспортируемое поле
}

func main() {
	u := User{
		ID:       123,
		Username: "alice",
		Password: "secret_password",
		apiKey:   "internal_key",
	}

	encoder := json.NewEncoder(os.Stdout)
	encoder.SetIndent("", "  ")
	encoder.Encode(u)
}

// Какой JSON будет сгенерирован? Какие поля будут включены?
```

<details>
  <summary>Спойлер (Объяснение)</summary>

  **Вывод JSON:**
  ```json
  {
    "id": 123,
    "username": "alice"
  }
  ```
  **Объяснение:**
  *   Поля `ID` и `Username` включаются в JSON с именами, указанными в тегах (`id` и `username`).
  *   Поле `Password` имеет тег `json:"-"`. Это специальное значение указывает пакету `encoding/json` полностью игнорировать это поле при маршалинге (и анмаршалинге). Оно не попадет в итоговый JSON.
  *   Поле `apiKey` написано с маленькой буквы, что делает его *неэкспортируемым* (приватным для пакета). Пакет `encoding/json` (как и другие пакеты, использующие рефлексию) по умолчанию игнорирует неэкспортируемые поля. Поэтому `apiKey` также не включается в JSON.
</details>

---
