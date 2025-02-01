# Computer Science

## СОДЕРЖАНИЕ
- Redux (#redux)
- RTK (#rtk)
___________________________________________________________________________________________________
Redux

- midleware (философское) - промежуточное ПО/посредник
- декоратор storeEnhancer - функция, которая принимает оригинальный createStore и возвращает отдекорированный createStore
- createStore - функция, которая принимает первоначальный state (preLoaded) и логику его изменения (reducer)
- compose - функция, которая принимает в качестве параметров соединяемые функции. Возвращает функцию, в которой результат вызова одной из функций становится параметром для другой, и так до бесконечности. Финальный результат вызова должен быть возвращен

- applyMiddleWare структура (3 уровня вложенности): 
  - applyMiddleWare принимает конструкторы midleware, а возвращает decoratedcreateStore (enhancer)
  - decoratedcreateStore принимается reducer, preloadedState, а возвращается отдекарированный store (содержит отдекорированный dispatch)
  - отдекарированный store содержит отдекорированный dispatch
- applyMiddleWare логика:
  Создается store => Вызываются конструкторы midelwares => создаются объекты midlewares => через compose midlewares связываются между собой в chain => на выход созданной chain подается dispatch => на выходе получится отдекорированный dispatch => в созданный store вставляем отдекорированный dispatch => возвращаем отдекорированный store
- bindActionCreator - нужен для связывания dispatch и actionCreator. Возвращаем функцию, внтури которой происходит связывание. updateCount = bindActionCreator(actionCreator, dispatch) => later updateCount() => это вызовет связанный dispatch => dispatch(actionCreator.apply(this, args))
- bindActionCreators - для связывания dispatch и множества actionCreator. Множества actionCreator представлено не в виде [], Set, Map, а виде объекта, поэтому наружу тоже возвращается объект со связанными actionCreator
  combineReducers -
___________________________________________________________________________________________________

RTK (Redux tool kit)
- deriving - выведение данных на основе других данных. Не надо заводить отдельный state для deriving data
- cache/memoization - cache (получение данных не из источника, например, copy); memoization (частный случай cache для ускорения получения результата, для предотвращения повторного запуска функции)
- selector - функция, которая дерайвит данные
- selector лучше иметь ближе к источнику данных (source of truth), иначе во всех местах, где этот селектор используется, его придется исправлять
- selectors можно располагать там, где есть source of truth (middleware, sage, reduce, component, ...)

Философия RTK, HOS (selectors высшего порядка)
- redux в своей логике обновления state использует structural sharing (НЕ deepClone)
- high-order functions пришло из ункционального декларативного программирования. Там функции представляют собой не процесс вычисления, сопоставление входынх и выходных данных (как на бумаге). Таких понятий, как замыкание, создание вложенных функций там нет. Поэтому единственным 2 способами использовать функцию в функции являются либо передача в параметрах, либо возвращение в качестве результата. Однако философия императивного js иная, с этой точки зрения можно взять функцию из замыкания, что также будет high-order function
-  Бывают selectors первого порядка, работают с источником данных. Selectors высшего порядка, которые работают с результатами selectors низшего порядка. Можно ли назвать selectors высшего порядка high-order selectors, такого определения нет нигде, но это удобно???
- createSelector принимает параметры: последний является output selector, все до него input selectors.Selectors низшего порядка являются input для High-order-selectors/HOS (selectors высшего порядка).  output callback являющийся частью HOS (selectors высшего порядка) рассчитывает результаты на основе результатов input selectors
- input selector не должен всегда возвращать новый результат (в случае redux весь state), иначе теряется смысл оптимизириующей логики HOS (selectors высшего порядка), в частном случае мемоизации
- output selector не имеет смысла, если он не производит трансформирующую логику. Легче сразу забрать данные от низшего selector без создания HOS (selectors высшего порядка), т.е. без использования re-select
- философски redux and СУБД схожи в сути нормализации данных, селектировании, ..., но сильно отличаются технически

- HOS (selectors высшего порядка) м.б. input selector для другого selector
- HOS (selectors высшего порядка) кеширует данные для одного потребителя. Если один HOS (selectors высшего порядка) используется 2 и более потребителями, то они могут перезаписывать cache и смысли мемоизации текущего HOS (selectors высшего порядка) теряется
- Вариант решения проблемы:
  - фабрика selectors (пользовательская функция, задача которой каждый раз возвращать новый selector)
  - вручную создавать HOS (selectors высшего порядка) для каждого потребителя
  - использовать re-reselect (кеширование по дополнительному и редко меняющемуся атрибуту. Среди параметров скидываемых для HOS и этот параметр или их комбинация будет ключом cache)
  - не смотря на все вышеописанные свободы, связи вида один HOS на один потребитель не избежать
- СХЕМА: мемоизация использует кеширование, кеширование использует сравнение (equalityCheck). Данная схема универсальна
- memoization спасает при тяжелых вычислениях
- HOS спасает при ссылочных типах данных (в случае возврата каждый раз новой ссылки селектором нижнего порядка, calculation над этими ссылками м. возвращать постоянное значени, что предотвратит react от перерендера)
- если логика программиста НЕ подразумевает calculation, а только селектирование, то смысла в HOS нет
- не нужно создавать селектора первого уронвя на каждое свойство state, это ухудшает читаемость/поддерживаемость кода

- каскадная мемоизация (cascade memoization) - мемоизация вложенных уровней. Возникает, когда есть вложенная цепочка функций, которые зависят друг от друга. Каждый уровень вложенности мы можем замемоизировать
- локальные и глобальные селекторы - локальные селекторы получают кусок store, глобальные - весь store
- В официальной доке reselect, redux и наших определениях есть расхождения. Синхронизация терминов:
  - HOS / Memoized selector / Output Selector (исп. на сайте reselect)
  - Selector низшего порядка / Dependencies / Input Selector
  - Result Func / Output Selector (исп. на сайте redux)
- argsMemoize (функция мемоизации работы inputs selectors)
- memoize (функция мемоизации  работы resultFunc)
- LRU (least recently used) распределение задач в структуре данных в зависимости от того времени добавления. Есть линиия времени, есть линия списка. Самая свежие данные встают в конец списка, а самые старые данные остаются вначале списка. Если размер списка превышен при добавлении новых данных свежие данные вставляются в конец, а самые старые выкидываются
- LRU м. релизовываться разными техническими прёмами (array, list, map, ...). Самый легкий способ - Map, т.к. он сохарняет порядок вставки сразу же
- важно понимать и уметь написать функцию memoize. Реализация:
```javascript
function checkForEquality(arg1, arg2) {
  return JSON.stringify(arg1) === JSON.stringify(arg2);
}

function memoize(targetFunc) {
  let memoArgs = [];
  let memoResult = null;

  function decoratedFunc(...args) {
    if (checkForEquality(args, memoArgs)) {
      return memoResult;
    } else {
      memoArgs = args;
      memoResult = targetFunc(...memoArgs);
      return memoResult;
    }
  }
  return decoratedFunc;
}
```
- функция memoize - это decorator. targetFunc - исходная функция для мемоизации, значит что если ее аргументы не изменились, то запускать снова targetFunc не надо. decoratedFunc - результат memoize, ее логика состоит в том, чтобы сохранять аргументы вызова и проверять, не изменились ли они. checkForEquality - вспомогательная функция для проверки изменения аргументов в decoratedFunc
- HOS - богатый, в static свойствах и методах лежит много информации (включая информацию связанную с мемоизацией из пердыдущего пункта (наш термин/термин reselect): memoize/memoize, resultFunc/targetFunc, memoResult/lastResult...)

- Кеширование и мемоизация НЕ синонимы. Мемоизирутся функции, кешируются данные. Мемоизация функции означает, что мы пропускаем лишний раз ее вызов за счет того, что ее аргументы не изменились. А аргументы не изменились, если они совпали с ранее закешированными. Количество разов кеширований можно увеличить с помощью LRU
- LRUmemoize theory - обычно HOS при каскадной мемоизации для каждого из двух уровней поддерживает размер cache 1. Можно создать HOS, cache которого для каждого уровня мемоизации обслуживается LRU (увеличить размер cache). Это осуществляется с помощью третьего опционального объекта настроек HOS, где можно настроить как мемоизацию первого уровня (argMemoize), так и мемоизацию второго уровня (memoize). Каждая из этих мемоизаций может настраиваться отдельно с помощью своего объекта настроек:
```javascript
{
  memoizeOptions: {
    equalityCheck: shallowEqual,       // сравнение аргументов
    resultEqualityCheck: shallowEqual, // сравнение результатов вызова функции
    maxSize: 10
  }
}
```
- shallowEqual - сначала проверка объектов по ссылке, затем распаковка и поверхностное сравнение содержимого первого уровня. Рекурсивно вглубдь сравнение НЕ производится
- withtypes - метод-декоратор, который подхватывает один из параметров дженерика. У HOS это state
- WeakMap - структура данных, где ключами являются только объекты. Пока на ключ существует любая другая ссылка кроме данного WeakMap, он существует. Как только остается только ссылка WeakMap, то garbage collector удалляет ключ. У WeakMap меньшее число методов по сравнению с Map,т.к. мы не знаем когда отработает garbage collector