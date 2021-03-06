# Новые #приватные поля классов в JavaScript

### Что это такое, как оно работает, и почему оно выглядит

*Перевод статьи [James Kyle](https://github.com/thejameskyle/): [JavaScript's new #private class fields](http://thejameskyle.com/javascripts-new-private-class-fields.html). Распространяется по лицензии [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*

*Эта статья лучше всего идёт под музыку: ["Noise Pollution" — Portugal. The Man](https://www.youtube.com/watch?v=kOts-yZsu8A)*

[Приватные поля классов](https://github.com/tc39/proposal-class-fields#private-fields) в процессе стандартизации JavaScript добрались до [Stage 2](https://tc39.github.io/process-document/). Это ещё не финальная стадия, но комитет по стандартам JavaScript ожидает, что эта возможность будет разработана и в конечном итоге включена в стандарт (хотя она все ещё может измениться).

Синтаксис (в настоящее время) выглядит так:

```javascript
class Point {
  #x;
  #y;

  constructor(x, y) {
    this.#x = x;
    this.#y = y;
  }

  equals(point) {
    return this.#x === point.#x && this.#y === point.#y;
  }
}
```

В этом синтаксисе есть две важные части:

1. Объявление приватных полей;
2. Обращение к приватным полям.

## Объявление приватных полей

Объявление приватных полей в основном совпадает с объявлением публичных полей:

```javascript
class Foo {
  publicFieldName = 1;
  #privateFieldName = 2;
}
```

Чтобы получить доступ к приватному полю, вам необходимо объявить его. При желании можно объявить поле и не записывая в него никакое значение:

```javascript
class Foo {
  #privateFieldName;
}
```

## Обращение к приватным полям

Ссылка на приватные поля работает аналогично доступу к любому другому свойству, только она имеет специальный синтаксис.

```javascript
class Foo {
  publicFieldName = 1;
  #privateFieldName = 2;
  add() {
    return this.publicFieldName + this.#privateFieldName;
  }
}
```

Также есть краткая форма записи для `this.#`:

```javascript
method() {
  #privateFieldName;
}
```

Это то же самое, что:

```javascript
method() {
  this.#privateFieldName;
}
```

## Обращение к приватным полям экземпляров класса

Обращение к приватным полям не ограничивается `this`. Вы также можете получить доступ к приватным полям экземпляров вашего класса:

```javascript
class Foo {
  #privateValue = 42;
  static getPrivateValue(foo) {
    return foo.#privateValue;
  }
}

Foo.getPrivateValue(new Foo()); // >> 42
```

Здесь `foo` является экземпляром `Foo`, поэтому нам разрешено обращаться к `#privateValue` из описания класса.

## Приватные методы (скоро будут?)

Приватные поля являются частью [предложения](https://github.com/tc39/proposal-class-fields), посвященного добавлению полей классов. Предложение не вносит никаких изменений в методы класса, поэтому приватные методы класса ожидаются в [следующем предложении](https://github.com/tc39/proposal-private-fields/blob/master/METHODS.md) и, скорее всего, будут выглядеть следующим образом:

```javascript
class Foo {
  constructor() {
    this.#method();
  }
  #method() {
    // ...
  }
}
```

Тем временем вы все равно можете присваивать приватным полям функции:

```javascript
class Foo {
  constructor() {
    this.#method();
  }

  #method = () => {
    // ...
  };
}
```

## Инкапсуляция

Если вы используете экземпляр класса, вы не можете ссылаться на приватные поля этого класса. Вы можете ссылаться на приватные поля только внутри класса, который их определяет.

```javascript
class Foo {
  #bar;
  method() {
    this.#bar; // Работает
  }
}
let foo = new Foo();
foo.#bar; // Неверно!
```

Кроме того, чтобы поля оставались действительно приватными, вы не сможете даже обнаружить их существование.

Чтобы быть уверенными, что вы не можете обнаружить частное поле, нам нужно разрешить объявление публичного поля с тем же именем.

```javascript
class Foo {
  bar = 1; // публичный bar
  #bar = 2; // приватный bar
}
```

Если частные поля не разрешают публичные поля с тем же именем, вы можете обнаружить существование приватных полей, попытавшись объявить свойство с тем же именем:

```javascript
foo.bar = 1; // Ошибка: `bar` это приватное свойство! (ой... обнаружили)
```

Или тихая версия (без вывода ошибки):

```javascript
foo.bar = 1;
foo.bar; // `undefined` (ой... снова обнаружили)
```

Эта инкапсуляция также должна быть верна для подклассов. Подкласс должен иметь возможность иметь одноименное поле, не беспокоясь о родительском классе.

```javascript
class Foo {
  #fieldName = 1;
}

class Bar extends Foo {
  fieldName = 2; // Работает!
}
```

*Примечание:* Для получения дополнительной информации о важности инкапсуляции или «жесткой приватности», прочитайте [этот раздел в FAQ](https://github.com/tc39/proposal-private-fields/blob/master/FAQ.md#why-is-encapsulation-a-goal-of-this-proposal).

## Итак, почему хештег?

Многие люди задаются вопросом: «Почему бы не следовать соглашениям из других языков и не использовать ключевое слово `private`»?

Вот пример такого синтаксиса:

```javascript
class Foo {
  private value;

  equals(foo) {
    return this.value === foo.value;
  }
}
```

Давайте рассмотрим две части этого синтаксиса отдельно.

## Почему объявления не используют ключевое слово `private`?

Ключевое слово `private` используется в множестве разных языков для объявления приватных полей.

Давайте посмотрим на синтаксис такого языка:

```javascript
class EnterpriseFoo {
  public bar;
  private baz;
  method() {
    this.bar;
    this.baz;
  }
}
```

В таких языках доступ к публичным и приватным полям осуществляется одинаково. Поэтому такое определение имеет смысл.

Тем не менее, в JavaScript, поскольку мы не можем использовать `this.field` для приватных свойств (почему — объясню ниже), нам нужен иной способ синтаксической связи. Использование `#` в обоих случаях делает отсылку наиболее понятной (мы сразу видим, что ссылаемся на приватное свойство).

## Зачем нужен #хештег в обращении?

Нам нужно использовать `this.#field` вместо `this.field` по нескольким причинам:

1. Из-за #**инкапсуляции** (см. раздел «Инкапсуляция» выше), мы должны разрешить создавать публичные и приватные поля с одним и тем же именем. Поэтому доступ к приватному полю не может быть обычным поиском в объекте.
2. К публичным полям в JavaScript можно обратиться через `this.field` или `this['field']`. Приватные поля не смогут поддерживать второй синтаксис (потому что они должны быть статичным), что может привести к путанице.
3. Вам не потребуются дорогие проверки.

Давайте рассмотрим пример кода.

```javascript
class Point {
  #x;
  #y;

  constructor(x, y) {
    this.#x = x;
    this.#y = y;
  }
  equals(other) {
    return this.#x === other.#x && this.#y === other.#y;
  }
}
```

Обратите внимание, как мы ссылаемся на `other.#x` и `other.#y`. Получая доступ к приватным полям, мы предполагаем, что `other` является экземпляром (`instanceof`) нашего класса `Point`.

Поскольку мы использовали синтаксис `#хештег`, мы сказали компилятору JavaScript, что мы просматриваем приватные свойства текущего класса.

Но что произойдет, если мы не будем использовать `#хештег`?

```javascript
equals(otherPoint) {
  return this.x === otherPoint.x && this.y === otherPoint.y;
}
```

Теперь у нас есть проблема: как мы узнаем, что такое `otherPoint`?

JavaScript не имеет системы статической типизации, поэтому `otherPoint` может быть чем угодно.

Это проблема по двум причинам:

1. Наша функция ведёт себя по-разному в зависимости от того, значения какого типа вы передаёте ей: иногда доступ к приватному свойству, в другое время - поиск публичного свойства.
2. Нам нужно будет проверять тип `otherPoint` каждый раз:

```javascript
if (
  otherPoint instanceof Point &&
  isNotSubClass(otherPoint, Point)
) {
  return getPrivate(otherPoint, 'foo');
} else {
  return otherPoint.foo;
}
```

Хуже того, нам нужно было бы сделать это для каждого доступа к свойствам в классе, чтобы проверить, ссылаемся ли мы на приватное свойство или нет.

Доступ к свойствам и так очень медленный, поэтому мы определенно не хотим добавлять к нему ещё больше веса.

**TL;DR:** Нам нужно использовать `#хештег` для приватных свойств, потому что использование стандартного доступа к свойствам повлечёт неожиданное поведение и приведёт к огромным проблемам с производительностью.

---

Приватные поля - отличное дополнение к языку. Спасибо всем замечательным трудолюбивым людям из TC39, которые сделали/делают их возможными!

---

*Слушайте наш подкаст в [iTunes](https://itunes.apple.com/ru/podcast/девшахта/id1226773343) и [SoundCloud](https://soundcloud.com/devschacht), читайте нас на [Medium](https://medium.com/devschacht), контрибьютьте на [GitHub](https://github.com/devSchacht), общайтесь в [группе Telegram](https://t.me/devSchacht), следите в [Twitter](https://twitter.com/DevSchacht) и [канале Telegram](https://t.me/devSchachtChannel), рекомендуйте в [VK](https://vk.com/devschacht) и [Facebook](https://www.facebook.com/devSchacht).*

[Статья на Medium](https://medium.com/devschacht/javascripts-new-private-class-fields-c60daffe361b)
