# Методы RegExp и String

В этой главе мы рассмотрим все детали методов для работы с регулярными выражениями.

## str.match(regexp)

Метод `str.match(regexp)` ищет совпадения с `regexp` в строке `str`.

У него есть три режима работы:


1. Если у регулярного выражения нет флага `pattern:g`, то он возвращает первое совпадение в виде массива со скобочными группами и индексом:

    ```js run
    let str = "I love JavaScript";

    let result = str.match(/Java(Script)/);

    alert( result[0] );     // JavaScript (всё совпадение)
    alert( result[1] );     // Script (первые собки)
    alert( result.length ); // 2

    // Дополнительная информация:
    alert( result.index );  // 0 (позиция совпадения)
    alert( result.input );  // "I love JavaScript (исходная строка)
    ```

2. Если у регулярного выражения есть флаг `pattern:g`, то он возвращает массив всех совпадений, без скобочных групп и других деталей.
    ```js run
    let str = "I love JavaScript";

    let result = str.match(/Java(Script)/g);

    alert( result[0] ); // JavaScript
    alert( result.length ); // 1
    ```

3. Если совпадений нет, то, вне зависимости от наличия флага `pattern:g`, возвращается `null`.

    Это очень важный нюанс. При отсутствии совпадений возвращается не пустой массив, а именно `null`. Если об этом забыть, можно легко допустить ошибку, например:

    ```js run
    let str = "I love JavaScript";

    let result = str.match(/HTML/);

    alert(result); // null
    alert(result.length); // Ошибка: у null нет свойства length
    ```

    Если хочется, чтобы результатом всегда был массив, можно написать так:

    ```js
    let result = str.match(regexp) || [];
    ```


## str.matchAll(regexp)

[recent browser="new"]

Метод `str.matchAll(regexp)` - "новый, улучшенный" вариант метода `str.match`.

Он используется, в первую очередь, для поиска всех совпадений вместе со скобочными группами.

1. Он возвращает не массив, а перебираемый объект с результатами, обычный массив можно сделать при помощи `Array.from`.
2. Каждое совпадение возвращается в виде массива со скобочными группами (как `str.match` без флага `pattern:g`).

Например:

```js run
let str = '<h1>Hello, world!</h1>';
let reg = /<(.*?)>/g;

let matchAll = str.matchAll(reg);

alert(matchAll); // [object RegExp String Iterator], не массив, а перебираемый объект

matchAll = Array.from(matchAll); // теперь массив

let firstMatch = matchAll[0];
alert( firstMatch[0] );  // <h1>
alert( firstMatch[1] );  // h1
alert( firstMatch.index );  // 0
alert( firstMatch.input );  // <h1>Hello, world!</h1>
```

При переборе результатов `matchAll` в цикле `for..of` вызов `Array.from`, разумеется, не нужен.

## str.split(reg|substr, limit)

Разбивает строку в массив по разделителю – регулярному выражению reg или подстроке substr.

Обычно мы используем метод `split` со строками, вот так:

```js run
alert('12-34-56'.split('-')) // массив [12, 34, 56]
```

Но мы можем разделить по регулярному выражению аналогичным образом:

```js run
alert('12, 34, 56'.split(/,\s*/)) // массив [12, 34, 56]
```

## str.searсh(regexp)

Метод `str.search(regexp)` возвращает позицию первого совпадения с `regexp` в строке `str` или `-1`, если совпадения нет.

Например:

```js run
let str = "Я люблю JavaScript!";

*!*
let regexp = /Java.+/;
*/!*

alert( str.search(regexp) ); // 8
```

## str.replace(str|reg, str|func)

Это универсальный метод поиска-и-замены, один из самых полезных. Этакий швейцарский армейский нож для поиска и замены в строке.

Мы можем использовать его и без регулярных выражений, для поиска-и-замены подстроки:

```js run
// заменить тире двоеточием
alert('12-34-56'.replace("-", ":")) // 12:34-56
```

Хотя есть подводный камень.

**Когда первый аргумент `replace` является строкой, он ищет только первое совпадение.**

Вы можете видеть это в приведённом выше примере: только первый `"-"` заменяется на `":"`.

Чтобы найти все тире, нам нужно использовать не строку `"-"`, а регулярное выражение `/-/g` с обязательным флагом `pattern:g`:

```js run
// заменить все тире двоеточием
alert( '12-34-56'.replace( *!*/-/g*/!*, ":" ))  // 12:34:56
```

Второй аргумент - строка замены. Мы можем использовать специальные символы в нем:

| Спецсимволы | Действие в строке замены |
|--------|--------|
|`$$`|вставляет `"$"` |
|`$&`|вставляет всё найденное совпадение|
|<code>$&#096;</code>|вставляет часть строки до совпадения|
|`$'`|вставляет часть строки после совпадения|
|`$n`|если `n` это 1-2 значное число, то вставляет содержимое n-й скобки| считается слева направо, в противном случае это означает круглую скобку с указанным именем|
|`$<имя>`|вставояет содержимое скобки с указанным именем|

Например:

```js run
let str = "John Smith";

// поменять местами имя и фамилию
alert(str.replace(/(\w+) (\w+)/i, '$2, $1')) // Smith, John
```

**Для ситуаций, которые требуют "умных" замен, вторым аргументом может быть функция.**

Она будет вызываться для каждого совпадения, и её результат будет вставлен в качестве замены.

Функция вызывается с аргументами `func(match, p1, p2, ..., pn, offset, input, groups)`:

1. `match` -- найденное совпадение,
2. `p1, p2, ..., pn` -- содержимое скобок (см. главу [](info:regexp-groups)).
3. `offset` -- позиция, на которой найдено совпадение,
4. `input` -- исходная строка,
5. `groups` -- объект с содержимым именованных скобок (см. главу [](info:regexp-groups)).

Например, переведём выбранные совпадения в верхний регистр:

```js run
let str = "html and css";

let result = str.replace(/html|css/gi, str => str.toUpperCase());

alert(result); // HTML and CSS
```

Заменим каждое совпадение на его позицию в строке:

```js run
alert("Хо-Хо-хо".replace(/хо/gi, (match, offset) => offset)); // 0-3-6
```

Используем содержимое скобок:

```js run
let str = "John Smith";

let result = str.replace(/(\w+) (\w+)/, (match, name, surname) => `${surname}, ${name}`);

alert(result); // Smith, John
```

Если в регулярном выражении много скобочных групп, то бывает удобно использовать остаточные аргументы для обращения к ним:

```js run
let str = "John Smith";

let result = str.replace(/(\w+) (\w+)/, (...match) => `${match[2]}, ${match[1]}`);

alert(result); // Smith, John
```

Или, если мы используем именованные группы, то объект `groups` с ними всегда идёт последним, так что можно получить его так:

```js run
let str = "John Smith";

let result = str.replace(/(?<name>\w+) (?<surname>\w+)/, (...match) => {
  let groups = match.pop();

  return `${groups.surname}, ${groups.name}`;
});

alert(result); // Smith, John
```

Использование функции даёт нам  максимальные возможности по замене, потому что функция получает всю информацию о совпадении, имеет доступ к внешним переменным и может делать всё, что угодно.

## regexp.exec(str)

Метод `regexp.exec(str)` ищет совпадения с `regexp` в строке `str`. В отличие от предыдущих методов, вызывается на регулярном выражении, а не на строке.

Он ведёт себя по-разному в зависимости от того, имеет ли регулярное выражение флаг `pattern:g`.

Если нет `pattern:g`, то `regexp.exec(str)` возвращает первое совпадение в точности как `str.match(reg)`. Такое поведение не даёт нам ничего нового.

Но если есть `pattern:g`, то:
- Вызов `regexp.exec(str)` возвращает первое совпадение и *запоминает* позицию после него в свойстве `regexp.lastIndex`.
- Следующий вызов начинает поиск от `regexp.lastIndex`, возвращает следующее совпадение и запоминает позицию после него в `regexp.lastIndex`.
- ...и так далее.
- Если совпадений больше нет, то `regexp.exec` возвращает `null`, а для `regexp.lastIndex` устанавливается значение `0`.

Таким образом, повторные вызовы возвращают одно за другим все совпадения, используя свойство `regexp.lastIndex` для отслеживания текущей позиции поиска.

Ранее, до появления метода `str.matchAll`, вызов `regexp.exec` в цикле использовали для получить всех совпадений с их позициями и группами скобок в цикле:

```js run
let str = 'Больше о JavaScript на https://javascript.info';

let reg = /javascript/ig;

let result;

while (result = regexp.exec(str)) {
  alert( `Найдено ${result[0]} на позиции ${result.index}` );
  // показывает: Найдено JavaScript на позиции 9, затем
  // показывает: Найдено javascript на позиции 31
}
```

Конечно, `matchAll` делает то же самое, по крайней мере, для современных браузеров, но если браузер старый, и не хочется делать полифил, то `regexp.exec` - надёжный выбор.

Также мы можем вызывать `regexp.exec` для поиска совпадения, начиная с нужной позиции, если вручную поставить `lastIndex`:

```js run
let str = 'Hello, world!';

let reg = /\w+/g; // без флага g свойство lastIndex игнорируется
regexp.lastIndex = 5; // ищем с 5-й позиции (т.е с запятой и далее)

alert( regexp.exec(str)); // world
```

Если у регулярного выражения стоит флаг `pattern:y`, то поиск будет вестись не начиная с позиции `regexp.lastIndex`, а только на этой позиции (не далее в тексте).

Если в примере выше заменить флаг `pattern:g` на `pattern:y`, то ничего найдено не будет, поскольку именно на позиции 5 слова нет:

```js run
let str = 'Hello, world!';

let reg = /\w+/y;
regexp.lastIndex = 5; // ищем ровно на 5-й позиции

alert( regexp.exec(str)); // null
```

## regexp.test(str)

Метод `regexp.test(str)` ищет совпадение и возвращает `true/false`, в зависимости от того, находит ли он его.

Например:

```js run
let str = "Я люблю JavaScript";

// эти два теста делают одно и же
alert( *!*/люблю/i*/!*.test(str) ); // true
alert( str.search(*!*/люблю/i*/!*) != -1 ); // true
```

Пример с отрицательным ответом:

```js run
let str = "Ля-ля-ля";

alert( *!*/люблю/i*/!*.test(str) ); // false
alert( str.search(*!*/люблю/i*/!*) != -1 ); // false
```

Если регулярное выражение имеет флаг `pattern:g`, то `regexp.test` ищет, начиная с `regexp.lastIndex`, аналогично `regexp.exec`.

Таким образом, мы можем использовать его для поиска с заданной позиции:

```js run
let reg = /люблю/gi;

let str = "Я люблю JavaScript";

// начать поиск с 10 позиции:
regexp.lastIndex = 10
alert( regexp.test(str) ); // false (совпадений нет)
```

````warn header="Одно и то же регулярное выражение, использованное повторно, может дать другой результат"

Если мы применяем одно и то же регулярное выражение последовательно к разным строкам, это может привести к неверному результату, поскольку вызов `regexp.test` обновляет свойство `regexp.lastIndex`, поэтому поиск в новой строке может начаться с ненулевой позиции.

Например, здесь мы дважды вызываем `regexp.test` для одного и того же текста, и второй раз поиск завершается уже неудачно:

```js run
let regexp = /javascript/g;  // (regexp только что создан: regexp.lastIndex=0)

alert( regexp.test("javascript") ); // true (теперь regexp.lastIndex=10)
alert( regexp.test("javascript") ); // false
```

Это именно потому, что во втором тесте `regexp.lastIndex` не равен нулю.

Чтобы обойти это, можно присвоить `regexp.lastIndex = 0` перед новым поиском. Или вместо методов на регулярном выражении вызывать методы строк `str.match/search/...`, они не используют `lastIndex`.
````