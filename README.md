# autodochttprouter

## Основаня задача

Данный модуль предназначен для роутинга http запросов и автодокументирования кода.

Он, помимо основной работы он создает вызов /mgmt/help[/txt] и /mgmt/html/json, который выведет документацию по вызовам. При каждой компиляции роутер генерирует документацию по текущему состоянию структру.

## Как это все работает.

```go
    type AgeType struct {
        Date string `json:"bd" comment:"день рождения"`
        Col  int    `json:"col_of_year" comment:"кол-во полных лет"`
    }

    type A struct {
        ID   int64              `json:"id" comment:"уникальный номер"`
        Name string             `json:"name" comment:"имя"`
        Age  AgeType            `json:"age" comment:"возраст"`
        NonDocumentedField bool `json:"secret" comment:"-"`
    }

	router := autodochttprouter.NewResolver()
	router.Add("GET", "/test", func(w http.ResponseWriter, r *http.Request) { srvTestOkHandler(&w, r, db, cfg) }, "Тестовый вызов", nil, nil)
    router.Add("POST", "/test", func(w http.ResponseWriter, r *http.Request) { srvTestOkHandler(&w, r, db, cfg) }, "Тестовый вызов", []interface{}{A{}}, []interface{A{}})
    router.Add("DELETE", "/test", func(w http.ResponseWriter, r *http.Request) { srvTestOkHandler(&w, r, db, cfg) }, "-", nil, nil)
```

В данном примере мы:

1. Описываем структуры запросов.
1. Создаем новый роутер.
1. Добавляем в него тестовые вызовы.

Функция добавления пути имеет следующие параметры:

* **Метод** - м.б. любым, включая *. В этом случае по указанному пути будут обрабатываться все методы
* **Путь** - адрес вызова. Может содержать регулярные выражения, такие как **\w+** или **\d+**. 
* **htttp.HandlerFunc** функция, которая обрабатывает вызов по этому пути и с этим методом
* **комментарий** - строка, содержащая описание вызова. 
    * В случае GET запросов в ней же имеет смысл описать и параметры вызова
    * Если данный параметр установить в "-" (минус), то информация о вызове не появится в документации (см. вызов DELETE)
    * Это же правило работает для аттрибута comment в описании структур (см. поле NonDocumentedField)
* **массив пустых элементов запроса** в случае если входящий запрос имеет тип JSON, то можно передать набор пустых экземпляров стуктур, участвующих в запросе. и тогда автодокументация покажет их.
    * поле может быть **nil**
* **массив пустых элементов ответа** аналогично предыдущему

**Замечание**: любой элемент в аннотации структур м.б. опущен. Тогда будет выведено имя поля структуры.