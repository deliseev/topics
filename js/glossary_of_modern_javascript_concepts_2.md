# Глоссарий современных концепций JavaScript: Часть 2

### Узнайте об области видимости, замыканиях, потоке данных и концепциях, широко используемых в современных JS-фреймворках и приложениях.

Автор [оригинала](https://auth0.com/blog/glossary-of-modern-javascript-concepts-part-2/): Kim Maida

2 мая 2017

**TL;DR:** В [первой части серии "Глоссарий современных концепций JS"](./glossary_of_modern_javascript_concepts_1) мы узнали о функциональном, реактивном и функционально-реактивном программировании. В **части 2** мы получим представление о таких понятиях, как _scope_, _closures_, _tree shaking_, _components_ и т.д., а также о таких темах приложений JavaScript, как _data flow_ и _change detection_.

* * *

Введение
--------

Современный JavaScript за последние годы пережил массовое распространение и не подает признаков замедления. Многие концепции, появляющиеся в блогах и документации по JS, все еще незнакомы многим разработчикам front-end. В этой серии статей мы познакомимся с промежуточными и продвинутыми концепциями в современном ландшафте front-end программирования и рассмотрим, как они применяются к современному JavaScript, JS-фреймворкам и JS-приложениям.

* * *

Концепции
---------

В этой статье мы рассмотрим концепции, которые имеют решающее значение для понимания современного JavaScript и JS-приложений, включая **объем и закрытие, поток данных, обнаружение изменений, компоненты, компиляцию и многое другое**.

Вы можете сразу перейти к каждому понятию здесь или продолжить чтение, чтобы узнать о них по порядку.

* [Область видимости (глобальная, локальная, лексическая) и замыкания](#scope-closures)
* [Односторонний поток данных и двустороннее связывание данных](#data-flow-binding)
* [Обнаружение изменений в JS-фреймворках: грязная проверка, аксессоры, виртуальный DOM](#change-detection)
* [Веб-компоненты](#web-components)
* [Умные и тупые компоненты](#smart-dumb-components)
* [JIT (Just-In-Time) Compilation](#jit)
* [AOT (Ahead-of-Time) Compilation](#aot)
* [Дрожание деревьев](#tree-shaking)

* * *

Область видимости (глобальная, локальная, лексическая) и замыкания
------------------------------------------------------------------

_"Объясните замыкания"_ - это печально известный вопрос технического собеседования по JavaScript. Правда в том, что многие опытные разработчики JS испытывают трудности с _объяснением_ замыканий, даже если они концептуально _понимают_ (и даже используют) их. Давайте вернемся назад и поговорим о концепциях, необходимых для объяснения замыкания.

### Область видимости

Для того чтобы понять _замыкания_, нам нужно сначала понять **область видимости**. Область - это просто контекст нашего кода: где переменные и функции _доступны_.

Следующий пример демонстрирует две области видимости, **глобальную область видимости** и **локальную область видимости**:

```js
    // Глобальная область видимости
    var globalVar = 'Hello, ';
    console.log(localVar); // Uncaught ReferenceError: localVar не определена
    
    someFunction() {
      // Локальная область видимости
      var localVar = 'World!';
      console.log(globalVar + localVar); // 'Hello, World!'
    }
```

Все имеет доступ к **глобальной области видимости**. Если мы откроем пустой файл `.js` и введем `var globalVar`, то эта переменная будет доступна всему, что мы создадим. Если бы мы выполнили файл в браузере, областью видимости функции `globalVar` было бы `window`.

> **Примечание:** Если мы объявим новую переменную без ключевого слова `var`, она будет помещена в область видимости _global_, независимо от того, где она находится в коде. Возможно, вы уже сталкивались с этим (возможно, случайно).

Функция `someFunction` создает свою собственную **локальную область видимости**. Она также наследует доступ к глобальной области видимости. Мы можем свободно использовать `globalVar` _внутри_ `someFunction`. Однако глобальная область видимости _не_ имеет доступа к вложенным контекстам, таким как локальная область видимости `someFunction`. Если мы попытаемся записать `localVar` из глобальной области видимости, то получим ошибку, поскольку `localVar` не определен в глобальной области видимости.

В двух словах, вложенные функции имеют свою собственную область видимости. Функции, объявленные внутри другой функции, также имеют доступ к областям видимости своих родительских функций. Это называется _цепочка областей видимости_.

**Лексическая область видимости** (или _статическая область видимости_) означает, что каждая вложенная функция может обращаться к функциям, которые ее содержат.

Рассмотрим этот пример:

```js
    // Лексическая область видимости и цепочка областей видимости
    var a = 1;
    
    function outerFunc() {
      var b = 2;
      console.log(a + b);
    
      function middleFunc() {
        var c = 3;
        console.log(a + b + c);
    
        function innerFunc() {
          var d = 4;
          console.log(a + b + c + d);
        }
    
        innerFunc(); // log 10 (1 + 2 + 3 + 4)
      }
    
      middleFunc(); // логи 6 (1 + 2 + 3)
    }
    
    outerFunc(); // логи 3 (1 + 2)
```

Этот код можно запустить на этом [JSFiddle: JS Scope](https://jsfiddle.net/kmaida/7frrzym4/) (откройте консоль браузера, чтобы увидеть результаты). `innerFunc` - это самая внутренняя функция. Она объявляется внутри `middleFunc`, которая в свою очередь объявляется в `outerFunc`.

Функция `innerFunc` может обращаться к переменным, объявленным во всех ее родительских диапазонах. Ее цепочка областей видимости позволяет получить доступ к:

* `a` из _глобальной_ области видимости,
* `b` из `outerFunc`,
* `c` из `middleFunc`, и
* `d` из локальной области видимости `innerFunc`.

Это работает только _вниз по вложенным функциям, но не вверх_. Например, локально скопированная переменная `d` объявлена в `innerFunc` и _не_ доступна для `middleFunc`, `outerFunc` или глобальной области видимости.

### Замыкания

В [первой части Глоссария современных концепций JavaScript](./glossary_of_modern_javascript_concepts_1) мы узнали о _функциях высшего порядка_ и функциях как объектах первого класса. Если это не кажется вам знакомым, уделите время разделу [Функции высшего порядка](./glossary_of_modern_javascript_concepts_1.md/#higher-order-functions).


Теперь давайте вернемся к следующему примеру функции высшего порядка, который мы рассматривали в части 1:

```js
    // Функция высшего порядка
    function whenMeetingJohn() {
      return function() {
        alert('Hi!');
      }
    }
    var atLunchToday = whenMeetingJohn();
    
    atLunchToday(); // оповещает "Привет!".
```

Это функция, которая возвращает другую функцию. Давайте обновим этот пример, чтобы добавить аргумент (`salutation`) и переменную `greeting` в локальную область видимости `whenMeetingJohn`. Мы также назовем ранее анонимную возвращаемую функцию `alertGreeting`, чтобы нам было легче на нее ссылаться:

```js
    // Замыкания
    function whenMeetingJohn(salutation) {
      var greeting = salutation + ', John!';
    
      function alertGreeting() {
        alert(greeting);
      }
      return alertGreeting;
    }
    var atLunchToday = whenMeetingJohn('Hi');
    
    atLunchToday(); // оповещает "Привет, Джон!".
    whenMeetingJohn('Whassup')(); // оповещения "Whassup, John!".
```

Этот код доступен для выполнения по этой ссылке [JSFiddle: JS Closures](https://jsfiddle.net/kmaida/c9wuupz8/).

Замыкание формируется, когда функция (`alertGreeting`) _объявляется_ внутри внешней функции (`whenMeetingJohn`) и ссылается на переменные из локальной области видимости внешней функции (например, переменную `greeting`).

Термин "замыкание" относится к функции _и_ лексическому окружению (любые локальные переменные, которые были в области видимости, когда было создано замыкание), в котором эта функция была объявлена.

Когда мы выполняем `atLunchToday()`, мы получаем оповещение с аргументом, который мы передали при присваивании (`'Hi'` в данном случае) и переменной `greeting`, которая была доступна в лексическом окружении `alertGreeting`.

> **Примечание:** Мы также можем вызвать _возвращенную_ функцию (`alertGreeting`) _без_ присвоения. Это выглядит следующим образом: `whenMeetingJohn('Whassup')()`.

Надеюсь, вы сможете увидеть ценность замыканий при рассмотрении этого простого примера. Мы можем поприветствовать Джона несколькими разными приветствиями. Каждый раз мы создаем закрытие с доступом к конкретным данным приветствия, находящимся в области видимости на момент создания.

Другой распространенный пример, демонстрирующий закрытия, использует простое выражение сложения:

```js
    // Замыкания
    function addCreator(x) {
      return function(y) {
        alert(x + y);
      }
    }
    var add1 = addCreator(1);
    var add5 = addCreator(5);
    
    add1(2); // оповещения 3
    add5(2); // предупреждения 7
```

Этот код можно выполнить по этой ссылке [JSFiddle: JS Closures - Adder](https://jsfiddle.net/kmaida/qfvyofcs/).

И `add1` и `add5` являются закрытиями с разными лексическими средами, хранящими разные значения для аргумента `x`. Эти значения защищены тем, что они "заключены" в лексическое окружение каждого закрытия. Мы можем использовать фабричную функцию `addCreator(x)`, чтобы создать столько функций `add_`, сколько нам нужно.

### Область видимости и замыкания. Основные выводы

Область видимости - это то, чему многие разработчики JS учатся с самого начала, но, возможно, им не приходилось _объяснять_ словами или на конкретных примерах. Понимание границ жизненно важно для написания хорошего JavaScript.

> **Примечание:** Существует нечто большее, чем мы рассмотрели здесь. Есть несколько замечательных ресурсов, позволяющих достичь большего понимания, особенно в отношении ключевого слова [`this`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this). Дополнительные ссылки смотрите ниже.

Закрытия связывают данные с функцией, используя лексическое окружение, в котором была объявлена функция.

Чтобы узнать больше о **области видимости и замыканиях** (и `this`), ознакомьтесь со следующими ресурсами:

* [Everything you wanted to know about JavaScript scope](https://toddmotto.com/everything-you-wanted-to-know-about-javascript-scope/)
* [Explaining JavaScript Scope and Closures](https://robertnyman.com/2008/10/09/explaining-javascript-scope-and-closures/)
* [Область видимости в JavaScript](http://www.digital-web.com/articles/scope_in_javascript/)
* [Что такое лексическая область видимости?](http://stackoverflow.com/a/1047491)
* [MDN: Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
* [Что такое "this"?](https://howtonode.org/what-is-this)
* [MDN: this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
* [Понять "this" в JavaScript с ясностью и освоить его](http://javascriptissexy.com/understand-javascripts-this-with-clarity-and-master-it/)
* [JavaScript: Как работает ключевое слово "this"?](http://stackoverflow.com/a/3127440)

* * *

Односторонний поток данных и двустороннее связывание данных
-----------------------------------------------------------

С распространением фреймворков JavaScript и одностраничных приложений (SPA) для разработчиков JS важно понимать такие понятия, как поток данных / привязка данных, и как инструменты, которые мы используем, управляют этим.

### Односторонний поток данных

Приложение или фреймворк с **односторонним потоком данных** использует модель как единственный источник истины. [React](https://facebook.github.io/react/) - широко известный пример одностороннего потока данных (или _одностороннего связывания данных_). Сообщения посылаются из пользовательского интерфейса в виде событий, чтобы сигнализировать модели об обновлении.

Взгляните на следующий пример React:

```js
    // Односторонний поток данных в React
    class OneWay extends React.Component {
      constructor() {
        super();
        this.handleChange = this.handleChange.bind(this);
        // устанавливаем начальный текст this.state.text в пустую строку
        this.state = {
          text: ''
        };
      }
      handleChange(e) {
        // получаем новое входное значение из события и обновляем состояние
        this.setState({
          text: e.target.value
        });
      }
      render() {
        return (
          <div>
            <input type="text" onChange={this.handleChange} />
            <p>Text: {this.state.text}</p>
          </div>
        );
      }
    }
```

Этот код доступен для выполнения на сайте [JSFiddle: React One-Way Data Flow](https://jsfiddle.net/kmaida/045znrsf/).

Мы видим, что объектная модель `state` создана в функции `constructor`. Начальное значение `this.state.text` - пустая строка. В нашей функции `render()` мы добавляем обработчик `onChange` для нашего элемента `<input>`. Мы используем этот обработчик для `setState()`, сигнализируя объектной модели `state` обновить свойство `text` новым значением поля ввода.

Данные текут только в одном направлении: от модели вниз. Ввод пользовательского интерфейса _не_ имеет прямого доступа к модели. Если мы хотим обновить состояние в ответ на изменения из пользовательского интерфейса, вход должен отправить сообщение, несущее полезную нагрузку. Единственный способ, которым пользовательский интерфейс может влиять на модель, - через это событие и метод [`setState()`](https://facebook.github.io/react/docs/react-component.html#setstate). Пользовательский интерфейс никогда не будет [_автомагически_](https://en.wiktionary.org/wiki/automagical) обновлять модель.

> **Примечание:** Чтобы отразить изменения от модели _до_ пользовательского интерфейса, React создает новый виртуальный DOM и сравнивает старый виртуальный DOM с обновленным виртуальным DOM. Затем только _изменения_ отображаются в реальном DOM. Подробнее об этом мы поговорим в разделе [обнаружение изменений] (#change-detection).

### Двустороннее связывание данных

При **двусторонней привязке данных** данные передаются в обоих направлениях. Это означает, что JS может обновлять модель _и_ пользовательский интерфейс также может это делать. Обычным примером двустороннего связывания данных является [AngularJS](https://angularjs.org).

> **Примечание:** В этой статье _AngularJS_ относится именно к версии 1.x фреймворка, а _Angular_ относится к версиям 2.x и выше, согласно [Branding Guidelines for Angular](http://angularjs.blogspot.com/2017/01/branding-guidelines-for-angular-and.html).

Давайте реализуем тот же пример, что и выше, но с двусторонней привязкой данных AngularJS:

```js
    // AngularJS two-way data binding
    // script.js
    (function() {
      angular
        .module('myApp', [])
        .controller('MyCtrl', function($scope) {
          // устанавливаем начальный $scope.text в пустую строку
          $scope.text = '';
          // следим за изменениями $scope.text
          $scope.$watch('text', function(newVal, oldVal) {
            console.log(`Старое значение: ${oldVal}. Новое значение: ${newVal}`);
          });
        });
    }());
```

```html
    <!-- AngularJS двустороннее связывание данных -->
    <!-- index.html -->
    <body ng-app="myApp">
      <div ng-controller="MyCtrl">
        <input type="text" ng-model="text" />
        <p>Text: {{text}}</p>
      </div>
    </body>
```

Этот код доступен для выполнения на [Plunker: AngularJS Two-Way Binding](http://plnkr.co/edit/guuX5XYIYwI7OcoflTur?p=preview).

В нашем контроллере мы устанавливаем модель `$scope.text`. В нашем шаблоне мы связываем эту модель с `<input>`, используя `ng-model="text"`. Когда мы изменяем значение ввода в пользовательском интерфейсе, модель также будет обновлена в контроллере. Мы можем видеть это в `$watch()`.

> **Примечание:** Использование `$watch()` в контроллере является спорной практикой. Мы сделали это здесь для _примера_. В ваших собственных приложениях AngularJS учитывайте, что существуют альтернативы использованию `$watch()` в контроллерах (например, события), и если вы используете `$watch()`, всегда [отменяйте регистрацию часов `$onDestroy`](https://www.bennadel.com/blog/2480-unbinding-watch-listeners-in-angularjs.htm).

Это и есть двустороннее связывание в AngularJS. Как вы можете видеть, мы не установили никаких событий или обработчиков, чтобы явно сигнализировать контроллеру о том, что модель была обновлена в пользовательском интерфейсе. Привязка данных `text` в шаблоне автоматически использует [watcher](https://medium.com/@kentcdodds/counting-angularjs-watchers-11c5134dc2ef) для отображения изменений в модели. Мы также можем `$watch()` модель. Наблюдение обычно следует выполнять в сервисах или функциях директив `link`, а не в контроллерах.

> **Примечание:** AngularJS использует так называемый цикл [digest cycle](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$digest) (грязная проверка) для сравнения значения с предыдущим значением. Подробнее о грязной проверке в AngularJS вы можете прочитать в разделе [обнаружение изменений](#change-detection).

### Отступление: Двустороннее связывание данных в Angular

Но подождите! В [Angular](https://angular.io) (v2+) есть "банан в коробке" `[(ngModel)]`, верно? На первый взгляд, это может выглядеть как сохранение автоматической двусторонней привязки данных. Однако это не так. [Синтаксис двусторонней привязки Angular `[()]`](https://angular.io/docs/ts/latest/guide/template-syntax.html#!#two-way) просто сокращает привязку свойств и событий в шаблоне, а директива [`ngModel`](https://angular.io/docs/ts/latest/api/forms/index/NgModel-directive.html) предоставляет вам событие `ngModelChange`. Чтобы узнать больше об этом, ознакомьтесь с [этой статьей о двусторонней привязке в Angular](https://blog.thoughtram.io/angular/2016/10/13/two-way-data-binding-in-angular-2.html).

Следующие примеры функционально эквивалентны и демонстрируют директиву `ngModel`:

```html
    // директива ngModel: синтаксис двустороннего связывания
    <input [(ngModel)]="text" />
    <p>{{text}}</p>
    // привязка свойств и событий ngModel
    <input [ngModel]="text" (ngModelChange)="text=$event" />
    <p>{{text}}</p>
```

В [Angular docs on two-way binding](https://angular.io/docs/ts/latest/guide/template-syntax.html#!#two-way) подробно рассматривается этот синтаксис.

### Поток данных и связывание

Многие современные JavaScript-фреймворки и библиотеки используют однонаправленный поток данных ([React](https://facebook.github.io/react/), [Angular](https://angular.io), [Inferno](http://infernojs.org/), [Redux](http://redux.js.org/) и т.д.). Почему? Односторонний поток данных поощряет чистую архитектуру в отношении того, как данные перемещаются через приложение. Состоянием приложения также легче управлять, обновления более предсказуемы, а производительность также может быть выше.

Хотя автоматическая двусторонняя привязка данных была одной из самых популярных демонстраций [AngularJS](https://angularjs.org/) в 2009 году, [Angular](https://angular.io) оставил ее позади. Некоторые разработчики Angular поначалу сетовали на это, но в конечном итоге многие пришли к выводу, что прирост производительности и больший контроль перевешивают автоматизацию.

Как мы видели на примере React выше, важно помнить, что односторонний поток данных _не_ означает, что трудно обновлять хранилище из пользовательского интерфейса. Это означает лишь то, что такие обновления выполняются намеренно, с помощью специальных инструкций. Это менее волшебно, но гораздо более _управляемо_.

> **Примечание:** Обычно, когда разработчики упоминают "реализацию двустороннего связывания данных" во фреймворках с односторонним потоком данных (таких как React), они имеют в виду шаги, необходимые для того, чтобы изменения пользовательского интерфейса уведомляли состояние о том, что оно должно быть обновлено. Они не ищут способ реализовать _автомагическое_ двустороннее связывание.

Чтобы узнать больше о **одностороннем потоке данных и двусторонней привязке данных**, ознакомьтесь со следующими ресурсами:

* [Видео: Введение в односторонний поток данных](https://www.sitepoint.com/video-introducing-one-way-data-flow/)
* [Диаграммы, сравнивающие одностороннее и двустороннее связывание данных](http://stackoverflow.com/a/37566693)
* [Почему React делает упор на однонаправленный поток данных и архитектуру Flux?](https://hashnode.com/post/why-does-react-emphasize-on-unidirectional-data-flow-and-flux-architecture-ciibz8ej600n2j3xtxgc0n1f0)
* [Код связывания данных в 9 JavaScript-фреймворках](http://engineering.paiza.io/entry/2015/03/12/145216) (от 2015 года, но все еще стоит посмотреть)
* [Двустороннее связывание данных в Angular (v2+)](https://blog.thoughtram.io/angular/2016/10/13/two-way-data-binding-in-angular-2.html)
* [AngularJS Docs - Data Binding](https://docs.angularjs.org/guide/databinding)
* [Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html)

* * *

Обнаружение изменений в JS-фреймворках: Грязная проверка, аксессоры, виртуальный DOM
-------------------------------------------------------------------------

Обнаружение изменений важно для любого динамического одностраничного приложения JavaScript (SPA). Когда пользователь что-то обновляет, у приложения должен быть способ обнаружить это изменение и отреагировать на него соответствующим образом. Поэтому некоторые виды обнаружения изменений жизненно важны для SPA-фреймворков.

На достаточно высоком уровне рассмотрим несколько методов обнаружения изменений, используемых в популярных на сегодняшний день JavaScript-фреймворках.

### Грязная проверка

Хотя [Angular](https://angular.io) был выпущен, [AngularJS](https://angularjs.org) все еще является основой многих приложений, находящихся в производстве или разработке прямо сейчас. AngularJS использует так называемый цикл [digest](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$digest) для обнаружения изменений в приложении. Под капотом цикл digest - это _грязная проверка_. Что это значит?

**Грязная проверка** относится к глубокому сравнению, которое выполняется на всех моделях в представлении для проверки измененного значения. Цикл дайджеста AngularJS добавляет _наблюдателя_ для каждого свойства, которое мы добавляем в `$scope` и связываем в пользовательском интерфейсе. Еще один наблюдатель добавляется, когда мы хотим следить за изменениями значений с помощью `$scope.$watch()`.

> _"AngularJS запоминает значение и сравнивает его с предыдущим значением. Это базовая грязная проверка. Если значение изменилось, то срабатывает событие change."_
> 
> -[Мишко Хевери](https://twitter.com/mhevery), создатель AngularJS и Angular.

Цикл дайджеста - это _петля_. AngularJS проходит по своему списку наблюдателей и проверяет, не изменилась ли какая-либо из наблюдаемых переменных `$scope` (она же "грязная"). Если переменная не изменилась, он переходит к следующей наблюдаемой переменной. Если найдена "грязная" переменная, она запоминает ее новое значение и _вновь входит в цикл_. Когда во всем списке не обнаружено новых изменений, DOM обновляется.

Основными преимуществами проверки на загрязнения являются простота и предсказуемость: нет расширения объектов и нет API. Однако это также неэффективно. Каждый раз, когда _любое_ изменение, запускается цикл дайджеста. Поэтому при создании наблюдателей в AngularJS важно соблюдать осторожность. Каждый раз, когда свойство `$scope` привязывается к пользовательскому интерфейсу, добавляется наблюдатель. Каждый раз, когда реализуется `$watch()`, добавляется еще один наблюдатель. Многие директивы также добавляют наблюдателей, как и переменные scope, фильтры и повторители.

Хотя в _простых_ приложениях проверка на загрязнения выполняется достаточно быстро, мы можем легко увидеть, как это может выйти из-под контроля в сложной реализации. Это привело к появлению таких статей, как [11 советов по улучшению производительности AngularJS: 1. Минимизация/отказ от наблюдателей] (https://www.alexkras.com/11-tips-to-improve-angularjs-performance/#watchers) и [Ускорение цикла $digest в AngularJS] (https://coderwall.com/p/d_aisq/speeding-up-angularjs-s-digest-loop).

> **Примечание:** Angular (v2+) [больше не](https://gofore.com/en/angular-2-change-detection-part-2/) [использует](https://blog.thoughtram.io/angular/2016/02/22/angular-2-change-detection-explained.html) [грязную проверку](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c).

### Аксессоры

[Ember](https://emberjs.com/) и [Backbone](http://backbonejs.org/) используют **аксессоры данных** (_getters_ и _setters_) для обнаружения изменений. Объекты [Ember](https://emberjs.com/api/classes/Ember.Object.html) наследуют API Ember и имеют методы [`get()`](https://emberjs.com/api/classes/Ember.Object.html#method_get) и [`set()`](https://emberjs.com/api/classes/Ember.Object.html#method_set), которые должны использоваться для обновления моделей с привязкой к данным. Это позволяет установить связь между пользовательским интерфейсом и моделью данных, после чего Ember точно знает, что изменилось. В свою очередь, только измененные данные вызывают события изменения для обновления приложения.

> **Примечание:** В Backbone это делается с помощью [Backbone models](http://backbonejs.org/#Model) с методами [`get()`](http://backbonejs.org/#Model-get) и [`set()`](http://backbonejs.org/#Model-set).

Этот метод прост и заставляет автора приложения быть очень внимательным к привязке данных. Однако, с другой стороны той же медали, он может иногда приводить к путанице, поскольку `Ember.Object` используются только при привязке данных к шаблонам. Если нет привязки данных к UI, обновления не используют `Ember.Object`. Такой смешанный подход может привести к тому, что разработчик будет ломать голову, когда что-то не обновляется из-за забытого сеттера или геттера.

### Виртуальный DOM

**Виртуальный DOM** используется в [React](https://facebook.github.io/react/) (и [Inferno.js](https://infernojs.org)) для реализации обнаружения изменений. React не определяет каждое изменение отдельно. Вместо этого [_виртуальный_ DOM](https://medium.com/@rajikaimal/react-js-internals-virtual-dom-d054347b7f00) используется для _различения_ предыдущего состояния пользовательского интерфейса и нового состояния, когда происходит изменение. React уведомляется о таких изменениях с помощью метода [`setState()`](https://facebook.github.io/react/docs/react-component.html#setstate), который запускает метод [`render()`](https://facebook.github.io/react/docs/react-component.html#render) для выполнения диффа.

Виртуальный DOM (иногда известный как V-DOM) - это модель данных JavaScript, которая _представляет_ реальное дерево DOM. Когда создается виртуальный DOM, в браузер ничего не выводится. Старая модель сравнивается с новой моделью](https://facebook.github.io/react/docs/reconciliation.html), и как только React определяет, какие части виртуального DOM изменились, только эти части исправляются в _реальном_ DOM.

### Выводы по обнаружению изменений

Существует множество способов, которыми JavaScript-фреймворки управляют обнаружением изменений, в том числе [больше, которые](https://vuejs.org/v2/guide/reactivity.html#How-Changes-Are-Tracked) [не были](https://www.polymer-project.org/2.0/docs/devguide/observers) [рассмотрены здесь](https://auth0.com/blog/understanding-angular-2-change-detection/). У каждого из них есть сильные и слабые стороны, но современная тенденция направлена на более продуманные и менее автоматизированные методы, многие из которых используют [_шаблон наблюдателя_](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#observerpatternjavascript) под капотом.

Чтобы узнать больше о **обнаружении изменений** в JS-фреймворках, ознакомьтесь со следующими ресурсами:

* [Изменения и их обнаружение в JavaScript-фреймворках](https://teropa.info/blog/2015/03/02/change-and-its-detection-in-javascript-frameworks.html)
* [Обзор обнаружения изменений](https://gofore.com/en/change-detection-overview-part-1/)
* [Dirty checking in AngularJS](http://stackoverflow.com/questions/9682092/how-does-data-binding-work-in-angularjs/9693933#9693933)
* [Дайджест Loop и Apply](https://www.ng-book.com/p/The-Digest-Loop-and-apply/)
* [Класс Ember.Object](https://emberjs.com/api/classes/Ember.Object.html)
* [Accessors vs. Dirty Checking in JavaScript Frameworks](http://blog.bguiz.com/2013/08/05/accessors-vs-dirty-checking-in-javascript-frameworks/)
* [React.JS internals: Virtual DOM](http://reactkungfu.com/2015/10/the-difference-between-virtual-dom-and-dom/)
* [Разница между виртуальным DOM и DOM](http://reactkungfu.com/2015/10/the-difference-between-virtual-dom-and-dom/)
* [React: Согласование](https://facebook.github.io/react/docs/reconciliation.html)
* [virtual-dom](https://github.com/Matt-Esch/virtual-dom)
* [VueJS: как отслеживаются изменения](https://vuejs.org/v2/guide/reactivity.html#How-Changes-Are-Tracked)
* [Polymer 2.0: Observers and Computed Properties](https://www.polymer-project.org/2.0/docs/devguide/observers)
* [Понимание обнаружения изменений в Angular 2](https://auth0.com/blog/understanding-angular-2-change-detection/)
* [Angular Change Detection Explained](https://blog.thoughtram.io/angular/2016/02/22/angular-2-change-detection-explained.html)

* * *

Веб-компоненты
--------------

** Веб-компоненты ** представляют собой инкапсулированные, повторно используемые виджеты, основанные на API веб-платформы. Они состоят из четырех стандартов:

* [Пользовательские элементы](https://w3c.github.io/webcomponents/spec/custom/)
* [HTML-шаблоны](https://html.spec.whatwg.org/multipage/scripting.html#the-template-element)
* [Теневой ДОМ](https://w3c.github.io/webcomponents/spec/shadow/)
* [Импорт HTML](https://w3c.github.io/webcomponents/spec/imports/)

Веб-компоненты позволяют нам создавать и импортировать пользовательские элементы, которые автоматически связывают поведение JS с шаблонами и могут использовать shadow DOM для обеспечения области видимости CSS и инкапсуляции DOM.

Веб-компоненты состоят из набора [API веб-платформы](https://www.w3.org/standards/techs/components). Существуют библиотеки (такие как [Polymer](https://www.polymer-project.org/)) и полифиллы (такие как [webcomponents.js ](https://github.com/webcomponents/webcomponentsjs)) чтобы преодолеть разрыв между [текущей поддержкой браузера](http://jonrimmer.github.io/are-we-componentized-yet/) и будущая поддержка веб-API.

Допустим, мы хотим создать простой веб-компонент (`my-component`), который отображает некоторый статический текст. Мы хотели бы использовать атрибуты HTML, чтобы компонент менял цвет текста и регистрировал что-либо в консоли. Чтобы отобразить пользовательский элемент "<my-component>" на нашем веб-сайте или в приложении, мы могли бы импортировать и использовать его следующим образом:

```html
<!-- index.html -->
<html>
  <head>
    ...
    <script src="https://cdnjs.cloudflare.com/ajax/libs/webcomponentsjs/0.7.24/webcomponents.min.js"></script>
    <link rel="import" href="my-web-cmpnt.html">
  </head>
  <body>
    <my-web-cmpnt color="red" log="Hello"></my-web-cmpnt>
    ...
```

Чтобы создать веб-компонент `my-component`, использующий shadow DOM, наш `my-component.html` может выглядеть примерно так:

```html
<!-- my-component.html -->
<template>
  <style>
    .my-component {
      display: block;
      padding: 20px;
    }
  </style>
  <div class="my-component">
    <p>This is a custom element!</p>
  </div>
</template>
<script>
  (function(window, document, undefined) {
    var doc = document;
    // my-component document
    var self = (doc._currentScript || doc.currentScript).ownerDocument;
    var template = self.querySelector('template').content;
    // ShadowCSS shim, if needed
    if (window.ShadowDOMPolyfill) {
      WebComponents.ShadowCSS.shimStyling(template, 'my-component');
    }
    class MyComponent extends HTMLElement {
      constructor() {
        super();
      }
      connectedCallback() {
        // get attributes
        var color = this.getAttribute('color');
        var log = this.getAttribute('log');
        // utilize shadow DOM
        var shadowRoot = this.attachShadow({mode:'open'});  
        var clone = doc.importNode(template, true);
        var myComponent;
        shadowRoot.appendChild(clone);
        myComponent = shadowRoot.querySelector('.my-component');
        // style with color and output log
        myComponent.style.color = color;
        console.log(log);
      }
    }
    window.customElements.define('my-component', MyComponent);
  }(window, document));
</script>
```
Этот код доступен для запуска по адресу [Plunker: Веб-компоненты](http://embed.plnkr.co/6uAQPXbFbjQIIYRqX1cS/).

`<template>` определяет стиль CSS элемента и HTML-разметку. Затем, чтобы воспользоваться преимуществами функциональности shadow DOM и JS, мы добавляем тег `<script>` в наш `my-component.html `файл после закрывающего тега `</template>` и реализуем желаемую функциональность пользовательского элемента JS, чтобы получить атрибуты элемента и использовать их для изменения цвета текста и зарегистрируйте наше сообщение.

При проверке с помощью Chrome devtools наш компонент выглядит следующим образом:

![современный глоссарий js: пользовательский веб-компонент в Chrome inspector](https://images.ctfassets.net/23aumh6u8s0i/3OdEVD74LeQ43aE2SRaT3L/5887ee2d614d7bfeac09687dc8623361/screenshot_web-component)

Для получения гораздо более подробного руководства ознакомьтесь с [Веб-компонентами: Как создавать свои собственные пользовательские компоненты](https://auth0.com/blog/web-components-how-to-craft-your-own-custom-components/).

Веб-компоненты предоставляют браузерам мощные возможности, подобные фреймворку, и, хотя спецификация и поддержка все еще дорабатываются, концепции вдохновили такие фреймворки, как [Angular](https://angular.io/docs/ts/latest/api/core/index/Component-decorator.html ) (который первоначально пытался использовать веб-компоненты Web API, но в итоге получил свою собственную реализацию). Многие JS-фреймворки ([Angular](https://angular.io/docs/ts/latest/api/core/index/Component-decorator.html), [React](https://facebook.github.io/react/docs/react-component.html ), [Ember](https://www.emberjs.com/api/classes/Ember.Component.html), [Vue](https://vuejs.org/v2/guide/components.html)) используйте концепцию компонентизации с различной степенью сходства с API веб-компонентов.

### Основные выводы по веб-компонентам

Веб-компоненты позволяют нам создавать и использовать пользовательские HTML-элементы с помощью JS. Они также могут использовать shadow DOM для обеспечения определения области видимости CSS и инкапсуляции DOM. Сами по себе веб-компоненты не являются заменой всех функций SPA-фреймворка. Однако их основные концепции в значительной степени используются многими фреймворками, используемыми сегодня, и их будущее в интерфейсном ландшафте открывает множество возможностей для растущего [сообщества ресурсов](https://www.webcomponents.org/).

Чтобы узнать больше о **веб-компонентах**, ознакомьтесь со следующими ресурсами:

* [Текущее состояние веб-компонентов W3C](https://www.w3.org/standards/techs/components)
* [webcomponents.org ](https://www.webcomponents.org/introduction)
* [Веб-компоненты: Как создавать свои собственные пользовательские компоненты](https://auth0.com/blog/web-components-how-to-craft-your-own-custom-components/)
* [MDN: Веб-компоненты](https://developer.mozilla.org/en-US/docs/Web/Web_Components)
* [Использование веб-компонентов в React](https://facebook.github.io/react/docs/web-components.html)
* [Создайте свое первое приложение с использованием полимерных и веб-компонентов](https://auth0.com/blog/build-your-first-app-with-polymer-and-web-components/)
* [Мы уже скомпонованы?](http://jonrimmer.github.io/are-we-componentized-yet/)
* [webcomponentjs на GitHub](https://github.com/webcomponents/webcomponentsjs)
* [Google Polymer](https://www.polymer-project.org/)

* * *

Умные и тупые компоненты
-------------------------

Как мы говорили выше, некоторые современные JS-фреймворки в значительной степени основаны на компонентах. Это приводит к понятиям "компонентной архитектуры" и "компонентной коммуникации". В некоторых случаях (например, в [React](https://facebook.github.io/react/) и [Angular](https://angular.io)), компонентная архитектура использует **умные и немые компоненты**. Их также называют "контейнерными" (интеллектуальными) и "презентационными" (немыми) компонентами.

### Интеллектуальные компоненты

Также известные как компоненты _container_, **интеллектуальные компоненты** могут управлять взаимодействиями с [состоянием](https://auth0.com/blog/glossary-of-modern-javascript-concepts/#state) приложения и данными. Они обрабатывают бизнес-логику и реагируют на события, исходящие от дочерних элементов (которые часто являются немыми компонентами).

### Тупые компоненты

Также известные как компоненты _представления_ , **тупые компоненты** полагаются на _ввод_, предоставляемые их родителями, и не знают о состоянии приложения. Иногда их можно считать [чистыми](https://auth0.com/blog/glossary-of-modern-javascript-concepts/#purity) и являются модульными и многоразовыми. Они могут общаться со своими родителями, реагируя на какое-либо событие, но сами не управляют им.

### Презентационные и контейнерные компоненты в React

[Дэн Абрамов](https://github.com/gaearon), соавтор [Redux](http://redux.js.org/), [Создать приложение React](https://github.com/facebookincubator/create-react-app), [React Hot Loader](https://github.com/gaearon/react-hot-loader) и многое другое, первоначально написанное о [презентационных и контейнерных компонентах](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) и их значение в [React](https://facebook.github.io/react/), особенно при использовании с управлением состоянием, таким как Flux или Redux. В двух словах, эти концепции можно резюмировать следующим образом:

#### Презентационные (они же тупые) компоненты:

* Сосредоточьтесь на том, "как все выглядит".
* Разрешить сдерживание с помощью `this.props.children`
* Отсутствие зависимостей от остальной части приложения (т.е. никаких действий Flux или Redux или хранилищ)
* Только принимайте данные; не загружайте и не изменяйте их
* Обычно являются [функциональными](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components) (с исключениями)

#### Компоненты контейнера (он же Smart):

* Сосредоточьтесь на том, "как все работает".
* Предоставлять данные и поведение другим компонентам
* Обычно не имеют разметки DOM или очень мало ее, и никогда не имеют стилей
* Часто являются источниками данных с отслеживанием состояния
* Обычно генерируются из [более высокого порядка components](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)

Ознакомьтесь с [его статьей](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) для получения более подробной информации и объяснений.

### Пример: Умные и тупые компоненты в Angular

Для краткого примера, который не связан с контейнером состояний, давайте рассмотрим несколько простых интеллектуальных и немых компонентов, использующих [Angular](https://angular.io).

Допустим, нам нужна функция в стиле madlib, где мы [приносим извинения за резкость](https://www.threadless.com/product/7719/im_sorry_for_what_i_said_when_i_was_hungry) во время голода, усталости или отладки. Когда мы закончим, в браузере это должно выглядеть примерно так:

![современный глоссарий js: Angular умный и тупой components](https://images.ctfassets.net/23aumh6u8s0i/2H8G3F7yQOgdAuvoyzYp1p/96f62dec9ec70e23f909fe59acd77f01/angular-smart-dumb)

Интеллектуальный компонент (контейнер) выглядит следующим образом:

```ts
// app/smart.component.ts
import { Component } from '@angular/core';
import { DumbComponent } from './dumb.component';

@Component({
  selector: 'my-smart-cmpnt',
  template: `
    <h1>I'm sorry for what I said when I was {{selectedOption}}.</h1>

    <my-dumb-cmpnt
      [options]="optionsArr"
      (changedOption)="onOptionChange($event)"></my-dumb-cmpnt>
  `
})
export class SmartComponent {
  optionsArr = ['hungry', 'tired', 'debugging'];
  selectedOption = '______';

  onOptionChange(e: string) {
    this.selectedOption = e;
  }
}
```
Этот компонент управляет всеми данными, необходимыми для этой функции. Он предоставляет параметры для madlib (`optionsArr`), а затем обрабатывает, когда пользователь выбирает опцию (`onOptionChange()`). Он сохраняет "selectedOption", передает возможные параметры в немой компонент и устанавливает выбранный параметр, когда немой компонент генерирует событие "changedOption".

```ts
// app/dumb.component.ts
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'my-dumb-cmpnt',
  template: `
    <div *ngFor="let option of options">
      <button (click)="select(option)">{{option}}</button>
    </div>
  `
})
export class DumbComponent {
  @Input() options: Array;
  @Output() changedOption = new EventEmitter();

  select(option) {
    this.changedOption.emit(option);
  }
}
```

В свою очередь, компонент dumb принимает массив options в качестве входных данных и выполняет итерацию по каждому элементу, чтобы создать кнопки для выбора опции. При нажатии на опцию генерируется событие `changedOption` с выбранной опцией в качестве полезной нагрузки. Родительский интеллектуальный компонент затем обрабатывает это событие и устанавливает его "selectedOption` для отображения в пользовательском интерфейсе.

Этот код доступен для запуска по адресу [Plunker: Интеллектуальные и немые компоненты Angular](https://embed.plnkr.co/OeUYze7djU90AWmc2Sno/).

### Умные и тупые компоненты на вынос

Интеллектуальные (контейнерные) компоненты управляют данными, реализуют бизнес-логику и обрабатывают события. Немые (презентационные) компоненты принимают входные данные и могут генерировать события, которые обрабатываются родительским интеллектуальным компонентом. Немые компоненты являются модульными и могут повторно использоваться во всем приложении из-за их большей независимости от состояния. При использовании контейнера состояний, такого как [Redux](https://egghead.io/courses/getting-started-with-redux) или [ngrx/store](https://auth0.com/blog/managing-state-in-angular-with-ngrx-store/), только интеллектуальные компоненты будут отправлять действия или взаимодействовать с хранилищем.

Чтобы узнать больше о **интеллектуальных и немых компонентах**, ознакомьтесь со следующими ресурсами:

* [Компоненты контейнера](https://medium.com/@learnreact/container-components-c0e67432e005)
* [Презентационные и контейнерные компоненты](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
* [Презентационные и контейнерные компоненты](https://twitter.com/dan_abramov/status/711614407537774594?lang=en)
* [Реагируйте на Preact: интеллектуальные компоненты](https://preact.gitbooks.io/react-book/content/jsx/smart.html)
* [Реагировать на предварительное действие: тупые компоненты](https://preact.gitbooks.io/react-book/content/jsx/dumb.html)
* [Интеллектуальные компоненты Angular против компонентов презентации: в чем разница, когда использовать каждый из них и почему?](http://blog.angular-university.io/angular-2-smart-components-vs-presentation-components-whats-the-difference-when-to-use-each-and-why/)
* [Управление состоянием в Angular с помощью ngrx/store](https://auth0.com/blog/managing-state-in-angular-with-ngrx-store/)

* * *


JIT (Just-In-Time) компиляция
------------------------------

**Just-In-time (JIT) компиляция** - это процесс перевода кода, написанного на языке программирования, в машинный код во время выполнения (во время выполнения программы или приложения). Во время выполнения программы доступна определенная динамическая информация, например, идентификация типа. JIT-компилятор _мониторит_, чтобы обнаружить функции или циклы кода, которые выполняются несколько раз - такой код считается "теплым". Затем эти фрагменты кода компилируются. Если они выполняются довольно часто ("горячие"), JIT оптимизирует их и сохраняет оптимизированный, скомпилированный код для выполнения.

Когда компилятор оптимизирует "горячий" код, он делает предположения о его типах и форме, основываясь на последовательности предыдущих выполнений. На любой итерации, если эти предположения оказываются неточными, оптимизированный код отбрасывается.

[Браузеры используют](https://softwareengineering.stackexchange.com/a/291343) [JIT-компиляцию для выполнения JavaScript](https://hacks.mozilla.org/2017/02/a-crash-course-in-just-in-time-jit-compilers/). В современном ландшафте фреймворков JavaScript компиляторы в таких фреймворках, как Angular, могут использовать JIT для компиляции TypeScript и кода Angular в JS в машинный код во время выполнения локальной разработки, компилируя каждый файл отдельно. Это дает определенные преимущества, такие как отсутствие необходимости пересобирать проект при отслеживании изменений кода и более быстрое время первоначальной сборки.

### Выводы по JIT-компиляции

JIT-компиляция используется браузерами для компиляции JavaScript во время выполнения и фреймворками, такими как Angular, для обеспечения быстрой локальной разработки.

Чтобы узнать больше о **JIT-компиляции**, ознакомьтесь со следующими ресурсами:

* [Что делает компилятор just-in-time (JIT)?](http://stackoverflow.com/a/95679)
* [Обзор JIT-компилятора](https://www.ibm.com/support/knowledgecenter/SSYKE2_7.0.0/com.ibm.java.win.70.doc/diag/understanding/jit_overview.html)
* [1/3 Мультипликационное введение в WebAssembly](https://hacks.mozilla.org/2017/02/a-cartoon-intro-to-webassembly/)
* [2/3 Краткий курс по компиляторам just-in-time (JIT)](https://hacks.mozilla.org/2017/02/a-crash-course-in-just-in-time-jit-compilers/)
* [3/3 Краткий курс по ассемблеру](https://hacks.mozilla.org/2017/02/a-crash-course-in-assembly/)
* [Гонка за скоростью часть 2: Как работают компиляторы JavaScript](http://creativejs.com/2013/06/the-race-for-speed-part-2-how-javascript-compilers-work/)
* [Mozilla TraceMonkey](https://wiki.mozilla.org/JavaScript:TraceMonkey)
* [Как работает движок V8?](http://thibaultlaurens.github.io/javascript/2013/04/29/how-the-v8-engine-works/)
* [JIT vs AOT в Angular2 и как это использовать](https://pub.scotch.io/@kashyapmukkamala/jit-vs-aot-in-angular2-and-how-to-use-it)

* * *

AOT (Ahead-Of-Time) Compilation
-------------------------------

**Ahead-Of-Time (AOT) компиляция** - это процесс перевода кода, написанного на языке программирования, в машинный код перед выполнением (в отличие от [at runtime](#jit)). Это позволяет снизить накладные расходы во время выполнения и компилировать все файлы вместе, а не по отдельности.

Применительно к приложению JavaScript [AOT, например, используемому в Angular](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html), это означает инлайнинг HTML и CSS и возможность развертывания без компилятора, что значительно экономит размер. Браузер также может сразу отрисовать приложение, поскольку оно предварительно скомпилировано.

Существует несколько преимуществ AOT для производственных сборок:

* Меньше асинхронных запросов: шаблоны и стили встроены в JS.
* Меньший размер загрузки: компилятор не нужно загружать, если приложение уже скомпилировано.
* Обнаружение ошибок шаблонов раньше: компилятор обнаруживает ошибки привязки во время сборки, а не во время выполнения.
* Лучшая безопасность: оценка уже сделана, что снижает вероятность инъекций.

AOT также позволяет использовать [tree shaking](#tree-shaking). В браузере приложение, скомпилированное с помощью AOT, имеет более короткое общее время загрузки и загрузки благодаря тому, что оно предварительно скомпилировано. Время начального рендеринга также сокращается, поскольку требуется разобрать меньше кода.

Однако время начальной сборки AOT больше, чем у JIT, и в случае внесения каких-либо изменений потребуется полная перекомпиляция всего приложения.

### Основные моменты компиляции AOT

AOT-компиляция происходит до времени выполнения и объединяет все файлы вместе. По сравнению с JIT, она обеспечивает "древовидное дрожание", меньший размер загружаемых файлов и повышенную безопасность.

Чтобы узнать больше о **AOT-компиляции**, ознакомьтесь со следующими ресурсами:

* [Angular: Is AOT Worth It?](https://blog.nrwl.io/angular-is-aot-worth-it-8fa02eaf64d4)
* [Ahead-of-Time Compilation in Angular](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)
* [Ahead-of-Time Comilation](https://en.wikipedia.org/wiki/Ahead-of-time_compilation)
* [IBM: компилятор опережающего времени](https://www.ibm.com/support/knowledgecenter/en/SSSTCZ_2.0.0/com.ibm.rt.doc.20/realtime/aot.html)

* * *

Дрожание деревьев (Tree Shaking)
------------

**Тряска деревьев** - это термин, обозначающий статический анализ всего импортируемого кода и исключение из него того, что на самом деле не используется.

В качестве более буквальной аналогии рассмотрим живое дерево. Дерево трясут, и это заставляет мертвые листья опадать, оставляя после себя листья, которые дерево активно использует для фотосинтеза. Концепция встряхивания дерева - это _включение живого кода_: мы включаем те части, которые нужны изначально, в отличие от удаления ненужных частей в конце (_устранение мертвого кода_).

Встряхивание деревьев опирается на модуль ES2015 [`импорт`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) и [`экспорт`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export). Операторы `import` и `export` составляют [статическую структуру модулей](http://exploringjs.com/es6/ch_modules.html#static-module-structure) приложения. Когда модули собираются в пакет для развертывания, древовидный шейкер анализирует статическую структуру модулей, чтобы исключить неиспользуемые экспорты, уменьшая размер конечного пакета.

ES2015 позволяет нам указывать явный импорт. Например, вместо того чтобы импортировать всю библиотеку RxJS, мы можем импортировать только то, что нам нужно:

```js
    import { BehaviorSubject } from 'rxjs/BehaviorSubject';
```

Это отличается от динамического оператора `require`, используемого CommonJS или AMD. Принцип tree shaking использует этот принцип, чтобы пройтись по графу зависимостей и исключить то, что не нужно, чтобы уменьшить размер пакетов развертывания.

Принцип древовидной встряски не нов, но недавно он был _популяризирован_ модулем [rollup.js](https://rollupjs.org/) bundler. В [Webpack 2](https://webpack.js.org/guides/tree-shaking/) также используется принцип древовидной встряски. Концепция встряхивания деревьев и написания кода, способствующего этому, также распространена в [Angular with](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html#!#tree-shaking)[AOT compilation](#aot).

### Основные выводы о Tree Shaking

Тряска деревьев" - это термин, обозначающий включение живого кода JavaScript в бандлеры модулей, которые используют [ES2015](https://auth0.com/blog/a-rundown-of-es6-features/) статические `импорт` и `экспорт` для "вытряхивания" ненужных зависимостей на более детальном уровне.

Чтобы узнать больше о **тряске деревьев**, ознакомьтесь со следующими ресурсами:

* [Tree-shaking versus dead code elimination](https://medium.com/@Rich_Harris/tree-shaking-versus-dead-code-elimination-d3765df85c80)
* [rollup.js](https://rollupjs.org/)
* [Tree Shaking with Webpack](https://webpack.js.org/guides/tree-shaking/)
* [Tree-shaking with webpack 2 and Babel 6](http://2ality.com/2015/12/webpack-tree-shaking.html)
* [Angular Docs: Tree shaking](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html#!#tree-shaking)
* [Kendo UI for Angular: Tree Shaking](http://www.telerik.com/kendo-angular-ui/components/framework/tree-shaking/)
* [How To Clean Up Your JavaScript Build With Tree Shaking](https://blog.engineyard.com/2016/tree-shaking)

* * *

Заключение
----------

С быстрым развитием фреймворков одностраничных приложений JavaScript и парадигм, основанных на компонентах, важно понимать темы JS, связанные с определением объема, потоком данных, компонентами, компиляцией и комплектацией.

Используя этот глоссарий в качестве отправной точки, вы сможете начать использовать преимущества этих концепций и парадигм программирования, чтобы повысить свою компетентность в JavaScript. Если что-то осталось неясным в отношении этих тем, пожалуйста, обратитесь к ссылкам в каждом разделе для получения дополнительных ресурсов. Вы также можете ознакомиться с [первой частью Глоссария современных концепций JS](https://auth0.com/blog/glossary-of-modern-javascript-concepts/), чтобы узнать о концепциях, необходимых для понимания функционального программирования, реактивного программирования и функционально-реактивного программирования.
