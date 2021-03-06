# Частина 2: Функції першого класу

## Короткий огляд
Коли ми кажемо функції "першого класу", ми кажемо, що вони такі як і всі...іншими словами - звичайний клас. Ми можемо розцінювати функції, як будь-який інший тип даних і в них немає нічого такого особливого: вони можуть зберігатися в масивах, передаватись в якості аргументів до інших функцій, їх можна призначити змінним і взагалі, зробити усе, що заманеться.

Усе наведене нижче - це основи JavaScript, але пройшовшись по коду на github можна зрозуміти, що багатьма такий підхід просто ігнорується. Чи розглянемо ми один вигаданий приклад? Чом би й ні:

```js
var hi = function(name) {
  return 'Hi ' + name;
};

var greeting = function(name) {
  return hi(name);
};
```

Як ви вже могли зрозуміти, обгортка навколо `hi` у функції `greeting` просто непотрібна. Чому? Та тому, що функції у JavaScript є викликаємими. Коли після методу `hi` вказані дужки `()` - це значить, що вкінці вона запуститься і поверне значення. А якщо дужок немає - метод просто поверне функцію, записану у змінну. Щоб переконатися, просто погляньте самі:


```js
hi;
// function(name) {
//  return 'Hi ' + name
// }

hi('jonas');
// "Hi jonas"
```

Оскільки `greeting` лише викликає метод `hi` з тим самим аргументом, ми можемо спростити наш код:

```js
var greeting = hi;


greeting('times');
// "Hi times"
```

Інакше кажучи, `hi` вже являється функцією, яка очікує єдиний аргумент, тож навіщо її обгортати іншою функцією, яка б просто викликала `hi` з тим єдиним нещасним аргументом? Це просто безглуздо. Це всеодно, що посеред липня вдягнути найтеплішу парку, для того, щоб запікатися і потребувати морозивка для охолодження.

Не вистачить жодних слів та емоцій, щоб передати наскільки ж це погано обгортати одну функцію іншою, лише тільки задля відтермінування її виконання.

Впевненне розуміння цього є вкрай важливим для продовження нашої мандрівки, тож давайте поглянемо на ще кілька прикольних прикладів з бібліотек, що були викопані з недр npm-пакетів.

```js
// неосвічений підхід
var getServerStuff = function(callback) {
  return ajaxCall(function(json) {
    return callback(json);
  });
};

// просвітлений
var getServerStuff = ajaxCall;
```

Всесвіт просто переповнений подібними цьому реалізаціями ajax-запитів. Ось чому обидва приклади - це одне й те саме:

```js
// ця лінія
return ajaxCall(function(json) {
  return callback(json);
});

// рівнозначна цій лінії
return ajaxCall(callback);

// тож getServerStuff можна переписати
var getServerStuff = function(callback) {
  return ajaxCall(callback);
};

// ...і це буде рівносильно ось цьому
var getServerStuff = ajaxCall; // <-- глянь мам, жодних дужок ()
```

І це, друзі, те, як воно має робитися. І давайте ще один разочок, для кращого розуміння, чому я такий наполегливий.

```js
var BlogController = (function() {
  var index = function(posts) {
    return Views.index(posts);
  };

  var show = function(post) {
    return Views.show(post);
  };

  var create = function(attrs) {
    return Db.create(attrs);
  };

  var update = function(post, attrs) {
    return Db.update(post, attrs);
  };

  var destroy = function(post) {
    return Db.destroy(post);
  };

  return {
    index: index,
    show: show,
    create: create,
    update: update,
    destroy: destroy,
  };
})();
```

Цей абсурдний контролер на 99% - пшик. Ми могли б його краще переписати отак:

```js
var BlogController = {
  index: Views.index,
  show: Views.show,
  create: Db.create,
  update: Db.update,
  destroy: Db.destroy,
};
```

... Ой-ой, здається воно все в купі тому, що воно нічогісінько не робить окрім як з'єднує разом наші в'юхи (view (eng) - показувати, те що відповідає за відображення, _прим. перекл._) з нашою базою даних.

## Навіщо віддавати перевагу першому класу?

Ну що ж, давайте нарешті перейдемо до причин, чому ж варто цінувати функції першого класу.
Як ми побачили на прикладах з `getServerStuff` та `BlogController`, дуже легко додавати прошарки, які не додають жодної цінності, а лише збільшують кількість непотрібного коду, який потім доведеться підтримувати і в якому копирсатися.

В додачу до всього, якщо нам потрібно щось змінити у функції, яку ми обгорнули непотрібним контейнером - нам доведеться змінювати і сам цей контейнер.

```js
httpGet('/post/2', function(json) {
  return renderPost(json);
});
```

Якщо нам захочеться змінити `httpGet`, для відправки можливого `err`, то нам доведеться змінювати і "клей".

```js
// повертається до кожного виклику httpGet у додатку/програмі та явно передає `err`.
httpGet('/post/2', function(json, err) {
  return renderPost(json, err);
});
```

Якби ми написали цей код з використанням функції першого класу - нам би довелось вносити менше змін:

```js
// `renderPost` виклиаканий у рамках `httpGet` з усіма потрібними йому аргументами
httpGet('/post/2', renderPost);
```

Окрім того, щоб видаляти непотрібні функції, ми маємо іменувати і посилатись на аргументи, які передаємо. Але з іменами теж можуть виникати проблеми, особливо за умови розростання та старішання кодової бази.

Мати кілька назв для однієї і тієї ж самої концепції - досить розповсюджена причина непорозуміннь у проектах. Ось, наприклад, дві функції, які роблять абсолютно одне й те саме, проте одна з них виглядає більш загальною і багаторазовою:

```js
// конкретно для нашого блогу
var validArticles = function(articles) {
  return articles.filter(function(article) {
    return article !== null && article !== undefined;
  });
};

// більш актуальна для майбутніх проектів
var compact = function(xs) {
  return xs.filter(function(x) {
    return x !== null && x !== undefined;
  });
};
```

Використовуючи більш конкретизовані назви для функцій, ми прив'язуємо самі себе до якихось конкретних даних (в нашому випадку `articles`). Таке трапляється і над цим треба працювати та вдосконалюватись.

Я маю відзначити, що як і з об'єктно-орієнтованим кодом, ви маєте пам'ятати, що `this` може болюче вжалити. Якщо основні функції вживають `this` і ми називаємо їх функціями першого класу, ми маємо бути дуже обережними.

```js
var fs = require('fs');

// страшнувато
fs.readFile('freaky_friday.txt', Db.save);

// трохи менше
fs.readFile('freaky_friday.txt', Db.save.bind(Db));

```

Прив'язавши `Db` до самого себе, ми надаємо йому вільний доступ до його сміттєвого коду з його ж прототипу. Я намагаюсь уникати використання `this` як брудного памперса. В цьому немає жодної потреби при написанні функціонально коду. Однак, при взаємодії з іншими бібліотеками, вам таки доведеться прийняти божевільність оточуючого нас Світу.

Хтось може посперечатися, мовляв `this` важливе для оптимізації швидкості. Будь ласка, якщо ви один з тих фанатів мікро-оптимізацій - просто закрийте цю книгу. Навіть якщо ви не можете отримати свої гроші назад - спробуйте обміряйте її на щось більш потрібне для вас.

І тепер, ми готові перейти до наступного кроку.

[Частина 3: Справжнє щастя з чистими функціями.](ch3-uk.md)
