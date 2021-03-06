# Anxiete

Список экзистенциальных проблем процесса разработки ПО и попытка найти их решение

## Противоречие "динамика" vs "статика", "надежность" vs "удобство"

### Проблема
Изначально никакой статики и динамики конечно нету, а есть только "спецификации" или "контракты" или "типы" или "инварианты" - ограничения на поведение работы программы которые справедливы для того или иного участка программы(функции, переменной, модуля итд).
Пример ограничения - функция возвращает список уникальных элементов, каждый элемент имеет тип `Number`
Но появляется два способа проверять подобные ограничения

#### Статически - по исходному коду без его выполнения
Cейчас это называют статическими типами. 
Плюсы:
 - надежно как швейцарские часы - если тайпчекер сказал что все в ок то программа гарантированно не содержит нежелательного поведения - например функция `add(a: number, b: number): number` никогда не вернет строку вместо числа
 - быстро - можно проверить даже очень большой проект за приемлимое время(меньше получаса)

Минусы:
 - по умолчанию считают, что ваш код не соотвествует спецификации и заставляют писать много дополнительного кода для того, чтобы доказать, что код все-таки ей соответствует
 - сложные инварианты(вроде того что функция `sort` идемподентна) не выражаются вне мозговзывательных систем типов вроде refinement и dependent типов

#### Динамически - во время исполнения
Сейчас это называют контрактами или спеками.
Плюсы:
 - по умолчанию считается, что наш код работает согласно спецификации - нет необходимости в написании какого-либо специального кода для доказательства корректности нашего кода
 - можно легко выразить любые динамические инварианты вроде идемподентности итд.

Минусы:
 - дают только вероятностные гарантии
 - чтобы хоть как то их проверить нужно либо иметь много тестов либо генерировать эти тесты из самих контрактов(при помощи инструментов property-based testing) - в любом случае прогон такого количества тестов для большого проекта это будет очень-очень медленно

### Как решать?

Нужен единый способ описания спецификаций - не привязанный к способу проверки(динамически или статически) - это может быть специальная DSL или просто библиотека для языка. Затем мы просто должны мочь выбрать как проверять те или иные спецификации - статически или динамически.
То что можем легко и безболезнено проверить статически проверяем статически - то что не можем проверяем динамически при помощи property-based testing.
Так же можно будет управлять степенью строгости проверок и их количеством в процессе разработки программы: в зависимости от того, что важнее для нас на данном этапе разработки – корректность или скорость и удобство разработки.

TODO тесты проперти и спецификации все должно быть видно в иде при редактировании кода - лежать может гдето в другом месте но в коде мы должны мочь быстро просмотреть все тесты и постоянные ограничения на поведение(как статические так и динамические) 

TODO итого было бы очень клево иметь такой пайплайн
ручками натыкали -> получили тесты -> проанализировав тесты в рантайме получили спецификации данных(читай типы) -> по типам нагенерили проперти тестов -> круг замкнулся

### Ссылки
 - https://arxiv.org/pdf/1607.05443.pdf - а вот ребята написали пейпер на похожий язык(только забыли про статику похоже)
 - http://blog.csssr.ru/2017/04/25/property-testing/ - введение в property-based testing и постановка вопроса
 - https://github.com/pelotom/runtypes, https://github.com/gcanti/tcomb, https://github.com/andreypopp/validated - либы для JS для динамической валидации но с частичной поддержкой статических типов.
 - https://blog.acolyer.org/2016/02/05/is-sound-gradual-typing-dead/
 - http://danluu.com/empirical-pl/ - многочисленные сравнения статики и динамики
 - http://blog.guillermowinkler.com/blog/2015/04/12/verifying-state-machine-behavior-using-test-dot-check/ - statefull example property
 - https://www.reddit.com/r/haskell/comments/30ixt4/programming_with_refinement_types/
 - https://arxiv.org/abs/1507.00385
 - http://noamz.org/papers/isbell.pdf
 - https://cstheory.stackexchange.com/questions/21836/why-does-coq-have-prop
 - https://news.ycombinator.com/item?id=7875309
 - https://cs.stackexchange.com/questions/54957/difference-between-dependent-type-refinement-type-and-hoare-logic
 - https://www.youtube.com/watch?v=jVyz3lWH2bA

### Что дальше?
 - написать генератор данных для одной из либ из списка выше
 - написать язык с встроенной поддержкой подобной системой спецификаций
 - причем тут градуал типизация и какова ее судьба
 - написать самостоятельный DSL для спецификаций чтобы можно было юзать в штках типо swagger для связи с АПИ


## Отсуствие в мейнстриме first-class control-flow

### Проблема
Такие проблемы написания кода как:
 - обработка ошибок
 - асинхронность и событийность
 - сайд-эффекты
 - проверка на нулл

Сводятся к проблеме отсутсвие first-class control-flow в большинстве языков.

Control-flow - порядок выполнения (порядок исполнения, порядок вычислений) — это способ упорядочения инструкций программы в процессе её выполнения.

Примеры(TODO примеры диаграм контрол флоу):

 - последовательный 
```javascript
var res1 = doSmth1(start);
var res2 = doSmth2(res1);
var res3 = doSmth3(res2);
```

 - при обработке ошибок
```javascript
var res1 = doSmth1(start);
if (res1.isError()) {
    var res2 = doSmth2(res1);
    if (res2.isError()) {
        var res3 = doSmth2(res2);
    }
}
```

 - асинхронный
```javascript
doSmth1(res1 => {
    doSmth2(res1, res2 => {
        doSmth3(res2);
    });
});
```

 - с исполнением сайд-эффектов
```javascript
var ioAction1 = doSmth1(start, IO);
ioAction.after(res1 => {
    var ioAction2 = doSmth2(res1);
    ioAction2.after(res2 => {
        doSmth3(res2);
    });
});
```

Стандартная единица абстракции кода - функция - не имеет доступа к управлению порядком управления программы. Она не может вернуть контроль исполнения, а потом его забрать обратно или произвольно переключать его.

Когда то давно когда компьютеры были большие был такой оператор goto(представим что он есть в JS и покажем как он помогает при обработке ошибок):
```javascript
var exitIfErr = res => res.isError ? goto exit : res;
var res1 = exitIfErr(doSmth1(start));
var res2 = exitIfErr(doSmth2(res1));
var res3 = exitIfErr(doSmth3(res1));
label exit;
```

Таким образом goto дает нам возможность абстрагировать произвольный control-flow. Однако широко известны проблемы [goto](https://ru.wikipedia.org/wiki/Goto#.D0.9A.D1.80.D0.B8.D1.82.D0.B8.D0.BA.D0.B0) - они делают его использование не возможным. Однако нам всетаки нужны способы абстракции control-flow.

### Как решать?
Существуют минимум 3 способа:
 
 - монады
 - обмен сообщениями(Stream)
 - [продолжения](http://beautifulracket.com/explainer/continuations.html) или корутины(разделить?)

А уже при помощи них можно решать более частные задачи. И на текущий момент мы так и делаем:
 - Promise,Future - решение вопроса асинхронности при помощи монад
 - Exception - решение вопроса обработки ошибок при помощи продолжений
 - Cycle.js - решение вопроса сайд-эффектов при помощи обмена сообщениями
 - Redux-saga - решение вопроса сайд-эффектов при помощи обмена корутин
 - [Algebraic Effects](https://esdiscuss.org/topic/one-shot-delimited-continuations-with-effect-handlers) - решение вопроса сайд-эффектов при помощи обмена продолжений

Однако все это очень не генерализированно и вообще нет понимания что это все одна и та же проблема(ОРЛИ?) и вообще все эти способы ее решать так или иначе изоморфны друг другу(ОРЛИ?).
TODO контрол флоу это просто вычисления высшего порядка - то есть вычисления над вычислениями.
ControlFlow(F1, F2) -> F3

### Ссылки
 - http://chris-taylor.github.io/blog/2013/02/09/io-is-not-a-side-effect/
 - https://blog.jle.im/entry/first-class-statements.html
 - https://github.com/typeetfunc/csssr.github.io/blob/feature/effects/_posts/2016-12-16-side-effects-1.md - на примере сайд-эффектов чем плох second-class control-flow
 - http://blog.sigfpe.com/2008/12/mother-of-all-monads.html  - продолжения как прородитель монад
 - https://blog.melding-monads.com/2009/12/20/are-continuations-really-the-mother-of-all-monads/ - или нет
 - https://courses.cs.washington.edu/courses/cse505/01au/functional/functional-io.pdf - описание трех способов на примере абстракции IO
 - https://youtu.be/Tkjg179M-Nc
 - https://github.com/cyclejs/cyclejs/issues/581#issuecomment-301344221
 - https://github.com/noflo/noflo http://www.jpaulmorrison.com/fbp/#More - Flow-based Programming
 - Что такое стрелки?  https://www.haskell.org/arrows/biblio.html
 - http://blog.ielliott.io/continuations-from-the-ground-up/

### Что дальше?
 - доказать что все эти проблемы стекаются к одной или опровергнуть это
 - доказать что есть 3 способа решения проблемы и они изоморфны или опровергнуть это
 - может во всем виновата лень? так как именно ленивые вычисления делают наши языки тьюринг полными. и как она связана с этим
 - шедулинг?
 - может нет 3 способов а есть один общий а все остальное вытекает из него
 - примеры
 - композиция control-flow - проблема монадных трансформеров(как решать и есть ли в остальных)

## Постановка задач не использует VCS

### Проблема

Что такое посути своей задача? Задача это _изменение_.
Как программисты работают с изменениями? Через VCS.
Как работают с изменениями в постановке? Через отдельные тикеты.
К чему это приводит?

Ко всем тем же проблема которые были у нас при разработке в до VCS эпоху:
 - не единого актуального снепшота описания всей системы(посути есть только набор слабо связанных патчей с разной историей). Чтоб составить единое представление системы нам надо осуществить мысленный процесс синтеза всех сделанных тикетов.
 - нет снепшотов на момент времени в прошлом - нельзя сделать дифф бизнесс описания системы на текущий момент с тем что было год назад(или день)
 - не видно конфликтов постановки при мерже веток с разными задачами(например в релиз)

Эти проблемы можно решить если помимо постановки задач вести отдельную версионируемую бизнесс-доку с описанием системы. Однако очевидно что никто никогда этим заниматся не будет - так как это двойной обьем работы для постановшика. Сначала ему надо будет сделать тикет а после релиза задачи перенести смысл тикета в бизнесс-доку.
 
### Как решать?

А почему бы не сделать наоброт - вести только доку а тикеты генерить из диффов этой доки?
Причем желательно эту доку хранить там же где весь код и тесты - в репе.
И тогда в ПР у нас был бы виден дифф:
 - постановки(что хотели сделать)
 - тестов(зафиксированные правки)
 - кода(как мы добились этих правок - то есть что таки сделали)

### Что дальше?
 - BDD достаточное решение(скорее нет чем да - неверю что вообще здесь хватит искуственного языка(геркина точно не хватит), и то что прям на все всегда будут написаны тесты я тоже не верю)
 - как быть с one-time тасками
 - нужна возможность как то аннотировать изменения(доп комменты писать, аттрибуты ставить итд)  - это нужно к примеру для багов(чеклисты?)
 - нужен более умный чем стандартый гитовый дифф - желательно семантический, и при этом для естественного языка. Причем желательно чтоб он понимал семантику доменной области и проекта
 - и еще вагон и маленькая тележка проблем

## Интерактивная разработка

### Проблема

На данный момент разработка крайне медлительна и изза своей не интерактивности. Нужна возможность писать код, тут же его исполнять на примерах, записывать подобные исполнения и иметь возможность проиграть их в будующем. На текущий момент среда исполнения, среда тестирования и среда написания кода это все разные вещи что создает проблему переключения контекста между ними.

### Как решать?

Среда написания кода должна быть встроена в среду исполнения. Взаимодействывать со средой исполнения должно быть можно как программно(REPL) так и физически - кликать мышкой кнопки, отправлять запросы в API итд.
При этом любое взаимодействие должно быть:
 - отслеживаемым - мы должны мочь понять вызов каких частей в нашем коде и с какими аргументами повлек например клик мышкой, или вызов функции в REPL. Отслеживааемость также поможет создать графическое представление программы(например как в [Cycle.js](https://github.com/cyclejs/cyclejs/tree/master/devtool#cyclejs-devtool-for-chrome))
 - сериализуемым и воспроизводимым - мы должным мочь на любом уровне(системы, модуля, функции итд) сериализовать входные взаимодействия и выходные реакции и автоматически получить из них автотест любого уровня
 - безопастно-отменяемым - time-travel учитывающий весь стейт(не только явный но и скрытый вроде сайд-эффектов)

Очевидно что для этого необходимо полностью избавится от side-effect в логике программы - все эффекты должны быть описаны first-class значениями которые можно будет как то безопастно отобразить.
А также научится контроллировать скрытый стейт вроде замыканий или операций вроде reduce в `rxjs`.

Все эти же техники можно будет использовать для remote REPL по типу erlang remsh и логгирования и мониторинга. Любую проблему у юзера с мониторинга можно будет автоматом сконвертировать в автотест.

### Ссылки
 - http://tonsky.me/blog/interactive-development/
 - http://lighttable.com/
 - http://witheve.com/
 - https://wallabyjs.com/
 - http://www.cs.cmu.edu/~NatProg/whyline.html
 - http://nightcoders.net/
 - https://sekao.net/nightlight/
 - http://pharo.org/
 - http://elm-lang.org/blog/the-perfect-bug-report
 - http://reactide.io/
 - http://www.luna-lang.org/
 - https://github.com/forresto/choo-graph-viz/issues/1
 - https://github.com/rempl/rempl

### Что дальше?
 - необходимо решить проблему first-class control-flow и следовательно сайд-эффектов и анализа control-flow в рантайме
 - понять что делать со стейтом
 - делать тулзы для разных языков фреймворков и понять можно ли выделать чтото общее и независимое от языка и платформы - по примеру того что делает http://langserver.org/ для компиляторов языков

## Модель данных для statefull приложений

### Проблема

Большая часть современного фронтенда и довольно приличная часть бекенда хранит стейт приложения в памяти, и может хранить его сутками(вкладка может быть открытой у пользователя почти вечно).

Возникает вопрос: нужна ли этому стейту какая то преодопределенная модель данных?

Минусы модели данных для стейтфул приложения:
 - отсуствие гибкости в хранении данных
 - если надо брать данные из внешнего источника то их надо как то конвертить в модель данных и обратно
 - нельзя просто так обратится к данным как мы привыкли с ними работать - обычно есть какой то специальный язык запросов со своими ограничениями - в то время как с обычными коллекциями можно работать миллионом разных способов
 - необходимо писать тулзы для отображения и работы с такими данными - в том время как JSON можно просто вывести `console.log`

Именно изза этих минусов большинство классических бекендов(стейтлесс) используют СУБД с некотрой моделью данных, но в большинстве случаев они потом изобретают поверх нее ОРМ который пытается представить что ты работаешь просто с коллекциями. Тогда может модель данных не нужна и надо просто все хранить в коллекциях?

Именно с хранения всего в коллекция и начали писать statefull приложения на клиенте. Однако очень быстро все пришли к тому что поверх коллекций начали накручивать модель данных:

 - https://github.com/paularmstrong/normalizr
 - graphql, falcor и другие
 - "redux action это транзакция"(с)
 - https://github.com/tonsky/datascript
 - https://google.github.io/lovefield/ - и даже гугл

Плюсы модели данных для стейтфул приложения:
 - автоматические индексы - не нужно писать специальный код для того чтобы различные манипуляции с данными были быстрыми. Данные хранятся оптимально с точки зрения удобства и корректности, но при этом все манипуляции происходят асимптотически оптимально.
 - предсказуемые ограничения на то что может и что не может быть в данных(однако этого можно добится просто с помощью контрактов или типов)
 - автоматическая синхронизация данных между различными источниками с одинаковой моделью данных - например имея на бекенде и фронтенд одинаковую модель данных можно синхронизировать их состояние при помощи мастер-мастер репликации и предсказуемо решать конфликты
 - работа со связями из коробки
 - обычно язык запросов получается намного более лаконичным и простым чем работа с данными при помощи [линз](https://github.com/calmm-js/partial.lenses), [zipper](https://github.com/polytypic/fastener)

Итого: хочется модель данных но не хочется всех ее ограничений. 

### Как решать?

Тут два направления на подумать:

 - ищем способ быть "здоровым и богатым" одновременно. А именно модель данных которая имея все плюсы не несет минусов - скорее всего невозможно - так как плюсы и минусы взаимосвязаны
 - думаем как сделать так чтоб данные можно было легко конвертить из одной модели данных в другую. Из тупого JSON в реляции, а из реляций к примеру в RDF тройки. То есть как сделать переносимые(и возможно даже сериализуемые) индексы, связи, констрейнты, правила разруливания конфликтов при синхронизации.

### Ссылки
 - https://disqus.com/home/discussion/devzen/0117/#comment-2999112703 - умный человек отвечает на вопрос
 - https://github.com/tonsky/datascript
 - https://www.youtube.com/watch?v=R2Aa4PivG0g
 - https://www2.eecs.berkeley.edu/Pubs/TechRpts/2009/EECS-2009-173.pdf
 - http://docs.datomic.com/tutorial.html

### Что дальше?
 - что вообще есть модель данных? к примеру JSON - это модель данных? ключевые особенности и их формализация. Может ли у данных не быть модели?
 - какие есть модели данных? pros&cons
 - изучить реализации этих моделей

## Domain specific language: free monad, macro systems, embedded language
Примеры - рендеринг
https://www.youtube.com/watch?v=vzLK_xE9Zy8

## Противоречие выразительность vs простота

## Вики-убийца
прочитать про майндмапы http://glebkalinin.ru/the-brain/
пролог и диалектика
