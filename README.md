# Модуль для работы с путями

**Структура данных** - это некоторый набор данных представленный в xml или json формате.

```xml
<Файл>
    <Документ>
        <Доверенность Номер="23787263565">
            <ФИО Фамилия="Ростова" Имя="Наталья" Отчество="Ильинична"/>
        </Доверенность>
        <Сотрудник ИНН="6449013711" СНИЛС="12345678901">
            <ФИО Фамилия="Иванов" Имя="Иван" Отчество="Иванович"/>
            <Телефон Номер="89045634525"/>
            <Телефон Номер="+73432673489"/>
        </Сотрудник>
        <Сотрудник ИНН="3664069397" СНИЛС="12345678902">
            <ФИО Фамилия="Петров" Имя="Петр" Отчество="Петрович"/>
            <Телефон Номер="89049565496"/>
        </Сотрудник>
    </Документ>
</Файл>
```


**Путь в структуре данных** - это последовательность идентификаторов разделенных точками,
которые указывают адресацию до конкретного узла или атрибута.  
Например путь `Файл.Документ.Доверенность.Номер` указывает адресацию до номера доверенности.
Путь может быть множественным, тогда для указания конкретного множественного узла используются 
квадратные скобки. Например: `Файл.Документ.Сотрудник[0].ИНН`.

**Спецификация структуры данных** - это описание возможных узлов и атрибутов в структуре данных.
Пример спецификации можно посмотреть в файле `spec.json`.

Пример использования:

```javascript
// pathlib - экспортируемая функция из pathlib.js
// spec - содержимое из файла spec.json

var path = pathlib(spec);

path.isValid('Файл.Документ.Квитанция'); // false

var p1 = path('Файл.Документ.Доверенность');
p1.toArray(); // [{name: 'Файл', index: null}, {name: 'Документ', index: null}, {name: 'Доверенность', index: null}]
p1.length; // 3

```

## API

### pathlib([Object]) -> [Function]
Принимает спецификацию и возвращает функцию для создания объектов pathObject.

### path([String]|[Array]) -> [pathObject]
Создает объект pathObject из строки или массива.

```javascript
var p0 = path('');
var p1 = path('Файл.Документ.Доверенность');
var p2 = path([{name: 'Файл', index: null}, {name: 'Документ', index: null}, {name: 'Доверенность', index: null}])
```

### path.pathObject -> [Array]
Седержит массив узлов в виде объектов с именами и индексами множественности. 
Если множественность в спецификации не определена, то значение null.

```javascript
var p0 = path('Файл.Документ.Сотрудник[0].Телефон[1].Номер');
p0.pathInArray;

/**
 * [ { name: 'Файл', index: null },
 * { name: 'Документ', index: null },
 * { name: 'Сотрудник', index: '0' },
 * { name: 'Телефон', index: '1' },
 * { name: 'Номер', index: null } ]
 */
```

### path.isValid([String]|[Array]) -> [Boolean]
Возвращает истину если путь является корректным по спецификации.

```javascript
path.isValid(''); // false
path.isValid('Файл.Документ.Квитанция'); // false
path.isValid('Файл.Документ.Доверенность'); // true
```

### pathObject.fillMultiple(value='0') -> [pathObject]
Заполняет множественность в пути по данным спецификации, параметр `value` указывает символ заполнитель. 
Не заполняет множественность если она уже указана в пути.

```javascript
path('Файл.Документ.Сотрудник.ФИО.Фамилия').fillMultiple(); // path('Файл.Документ.Сотрудник[0].ФИО.Фамилия')
path('Файл.Документ.Сотрудник.ФИО.Фамилия').fillMultiple('1'); // path('Файл.Документ.Сотрудник[1].ФИО.Фамилия')
path('Файл.Документ.Сотрудник.Телефон[1].Номер').fillMultiple(); // path('Файл.Документ.Сотрудник[0].Телефон[1].Номер')
```

### pathObject.dropMultiple(index|null) -> [pathObject]
Удаляет множественность из пути по данным спецификации.  
Параметр `index` указывает из какого множественного элемента следует удалить множественность.

```javascript
path('Файл.Документ.Сотрудник[0].Телефон[1].Номер').dropMultiple(); // path('Файл.Документ.Сотрудник.Телефон.Номер')
path('Файл.Документ.Сотрудник[0].Телефон.Номер').dropMultiple(); // path('Файл.Документ.Сотрудник.Телефон.Номер')
path('Файл.Документ.Сотрудник[0].Телефон[1].Номер').dropMultiple(-1); // path('Файл.Документ.Сотрудник[0].Телефон.Номер')
path('Файл.Документ.Сотрудник[0].Телефон[1].Номер').dropMultiple(0); // path('Файл.Документ.Сотрудник.Телефон[1].Номер')
```

### pathObject.each(callback)
Выполняет обход по пути от корневого элемента.  
Функция `callback(obj, index)` выполняется над каждым узлом или атрибутом.  
Если функция `callback` возвращает `false` итерации останавливаются. 

```javascript
var p1 = path('Файл.Документ.Сотрудник[0].Телефон[1].Номер');

p1.each(function(obj, index) {
   // obj.name - название узла или атрибута.
   // obj.index - порядковый номер множественного элемента (null для немнож. или атрибутов).
   if (obj.name === 'Телефон') {
      return false; // обход прекращается.
   }
});
```

### pathObject.toArray() -> [Array]
Возвращает путь в виде массива.

```javascript
var p1 = path('Файл.Документ.Доверенность');
p1.toArray(); // [{name: 'Файл', index: null}, {name: 'Документ', index: null}, {name: 'Доверенность', index: null}]
```

### pathObject.toString() -> [String]
Возвращает путь в виде строки.

```javascript
var arr = [{name: 'Файл', index: null}, {name: 'Документ', index: null}, {name: 'Доверенность', index: null}];
var p1 = path(arr);

p1 + ''; // 'Файл.Документ.Доверенность'
p1.toString(); // 'Файл.Документ.Доверенность'
```

### pathObject.length -> [Number]
Возвращает количество элементов в пути.

```javascript
var p1 = path('Файл.Документ.Доверенность');
p1.length; // 3
```

### pathObject.inColumn() -> [String]
Выводит узлы пути с их индексами в столбик.

```javascript
var p1 = path('Файл.Документ.Сотрудник[0].Телефон[0].Номер');
p1.inColumn(); 
/**
 * Файл
 * Документ
 * Сотрудник[0]
 * Телефон[0]
 * Номер
 /*
 ```
 ### pathObject.getMultiple() -> [Array]
 Возвращает массив индексов множественных узлов объекта и индексов узлов.
 ```javascript
 var p1 = path('Файл.Документ.Сотрудник[0].Телефон[0].Номер');
p1.getMultiple(); // [ [ 2, '0' ], [ 3, '0' ] ]
```

## Запуск тестов

Для запуска тестов необходимо установить [Node.js](https://nodejs.org).    
Для запуска тестов выполнить `npm test`.
