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

- Существуют 2 режима работы garbage Collector: малый Scavange и большой. Малый запускается каждые несколько милисекунд. Большой запускается раз в несколько минут/часов
- weakRef - специальный объект со скрытой слабой ссылкой. Слабая ссылка - НЕ сам объект weakRef, а указатель, который не препятствует удалению объекта сборщиком мусора
- weakRef живет по расписанию большого режима работы garbage Collector
- FinalizationRegistry - api, которое позволяет привязать привязать к удалению целевого объекта колбэк очистки (финализатор)
- FinalizationRegistry.register принимает 3 параметра: target (отлеживаемый объект), heldValue (любое значение, пробрасываемое в колбек очистки), unregisterToken (токен для идентификации в случае отмены подписки, используется для unregister)
- FinalizationRegistry.unregister принимает unregisterToken
- Пример FinalizationRegistry + weakRef. [Ссылка на главу](https://learn.javascript.ru/weakref-finalizationregistry)
```js
function fetchImg() {
  // абстрактная функция для загрузки изображений...
}

function weakRefCache(fetchImg) {
  const imgCache = new Map();

  const registry = new FinalizationRegistry((imgName) => { // (1)
    const cachedImg = imgCache.get(imgName);
    if (cachedImg && !cachedImg.deref()) imgCache.delete(imgName);
  });

  return (imgName) => {
    const cachedImg = imgCache.get(imgName);

    if (cachedImg?.deref()) {
      return cachedImg?.deref();
    }

    const newImg = fetchImg(imgName);
    imgCache.set(imgName, new WeakRef(newImg));
    registry.register(newImg, imgName); // (2)

    return newImg;
  };
}

const getCachedImg = weakRefCache(fetchImg);
```

- отсутствие возвращаемого значения в функции лучше всего кодировать уникальным Symbol, чтобы его не спутать с user undefined, null, false, ...
- идея weakMapMemoize - где массив аргументов мемоизированной функции мы переводим в связный список. Связный список имеет особую структуру (каждый элемент списка представляет собой кешированную ноду/cacheNode, где корень - кешированная нода самой мемоинзированной функции). В самой ноде мы храним аргумент и ссылку на следующую кешированную ноду следующего аргумента
- результат (cacheNode.v/value) есть только у cacheNode последнего аргумента.
- если argument примитив, то внутри cacheNode сохраним его в Map, если объект, то в WeakMap. Из-за того, что ссылки на другие cacheNode хранятся в структуре Map/WeakMap, у нас получается не односвязный список, а разветвленное дерево cacheNodes
- так как новый аргумент сравнивается с ключом в Map (через get). Механизм сравнения управляется js через strictEquality, поэтому в отличие от LRUMemoize нет возможности настроить сравнение входящих аргументов


- Как понимать фразу "функция от чего-то зависит"? Результат вычисления этой функции определяется на основе каких-то данных, используемой этой функцией: аргументы, данные в теле функции, данные из замыкания, встроенный this, ...
- НЕ путать аргументы с зависимостями. Зависимости - ВСЕ данные от которых зависит резулььтат вычисления функции. Аргументы - частный случай зависимостей, который тоже влияет на результат вычисления функции


- Реактивность - идея того, что при изменении нижележащих данных поменяются и вышележащие данные. Highorder Data depends on lowOrder Data??? (наше определение, такой терминологии нет).
- push стратегия - изменение нижележащих данных обновит вышележащие данных (rxjs)
- pull стратегия - изменение нижележащих данных НЕ будет обновлять вышележащие данны, пока кто-то не воспользуется этими вышележащим данными. Но сам факт изменения нижележащих данных будет обязателььно зафиксирован (autotracking)
- Асинхронность - НЕ путать с реактивностью, НЕ про данные, а ПРО действия
- autrotracking = reactivity + memoization. МЫ привыкли, что мемоизация - функция, аргументы, результат. Но если аргументы расширить до зависимостей, то получим autrotracking. Мемоизация, которая отслеживает изменение зависимостей и в результате этого пересчитывает cache, дает autrotracking
- Зависимости в виде простых данных (чисел, объектов), их изменений невозможно отследить. Для этого эти простые данные оборачиваются в Cell/Storage, которые имеют перехватчики get/set. Технически перехватчики могут быть выражены разными средствами, например, Proxy, Class getter/setter, дескрипторы
- get - позволяет создать связь между выполняемой функцией и зависимостями встречанными внутри этой функции
- set - либо констатирует факт изменения данных (pull: autrotracking), либо сразу активирует измененные данные (push: rxjs)

- patches - расширение структуры проекта изнутри
- plugin - расширение структуры системы согласно подготовленному интерфейсу системы. НЕ просто пропатчить исходники, а именно в разрешенных точках
- hook - расширение поведения системы в определенных точках
- Objectish - Map/Set/Array/Object
- forEach для map/set
- Symbol.for - глобальные символы
- Сервис (service) - набор взаимосвязанного кода (чаще всего выраженного через модули), решающий общую задачу (единый уровень абстракции), предоставляющий интерфейс для этого взаимодействия

```
action                   actionCreator                    createAction
{                        (payload) => {                   (type, prepareAction) => 
  type: string             type: string //hardcoded         actionCreator плюс 
  payload: any             payload: any                     actionCreator.type,
}                          meta: any                        actionCreator.toString(),
                           error: any                       actionCreator.match()
                         }
```
- actionCreator - это идея, этой функции нет в виде API rtk/redux. Это удобная фабрика по созданию actions. Поля type, payload - признаны всеми, поля meta, error - для служебных целей rtk, но никто не запрещает использовать их программисту. Программист сам определеяет способ их добавления (дополнительными параметрами в actionCreator, после создания action записыывать отдельными свойствами)
- createAction - декоратор actionCreator с binded type. Польза видна при большом количестве actions, позволяя избегать большого количества boilerplate (образуется при объявлении action constants, функций actionCreator). Суть декорирования помогает в дальнейшей работе внутренней логики rtk 
- prepareAction - callBack, обрабатывающий payload и возврашающий объект {payload, meta, error}. Этот объект потом сольётся с объектом type. Если prepareAction не передан, то переданный user payload запишется сразу в action.payload 
- builder - паттерн, который позволяет собрать из маленьких частей большую сущность. Отдельные маленькие части собираются путем вызова отдельных методов builder. Собранные маленькие части в builder надо где-то сохранять!!! пока не вызвали метод build. Маленькие методы лучше чейнить. После вызова финализирующего метода build чейна не будет
________________________________________________________________________________________________________________________
- сreateReducer - функция, которая создаёт reducer, принимает initialState и callback, первым параметром которого будет builder. В отличие от redux в rtk initialState обязателен (избавляет от багов reducer)
- initialState/fallbackState/default - обязательное первоначальное состояние необходимое для работы reducer, preloadedState/not default - заранее заданное пользовательское состояние, с которым создаётся store, опциональное первоначальное состояние
- если обычный reducer представляет большой switchCase, то createReducer этот switchCase рзабивает на набор методов (addCase, addMatcher, addDefaultCase). Эти методы доступны в builder
  - addCase один action -> один caseReduсer
  - addMatcher несколько action -> один caseReduсer
  - addDefaultCase неизвестный action -> один caseReduсer
- в обычном redux отрабатывала ВСЕГДА одна ветка switchCase, а в createRedcuer благодаря matchers может отработать несколько веток switchCase
- если возникает ситуация отработки нескольких веток switchCase, то они будут выполнены через композицию в порядке их добавления программистом
- сreateReducer под капотом использует Immer, поэтому caseReduсer можно писать в мутационном стиле
________________________________________________________________________________________________________________________
- createAsyncThunk - идея автоматизированной обработки жизненного цикла асинхронной операции
- createAsyncThunk принимает 3 параметра:
  - string prefix as base for actionType (prefix/pending, prefix/reject, prefix/fulfilled)
  - user callback (обычно асинхронный)
  - options - доп. настройка asyncThunk
- createAsyncThunk возвращает actionCreator, который при вызове возвращает функциональный action. Функциональный action - action, который является функцией. При dispatch перехыватывается redux thunk middleware, выполняется, после возвращает всегда resolved promise. Пример: `const result = await thunk()(dispatch, getState, null)`

- Ключевые составляющие:
  - dispatch action согласно жизненному циклу асинхронной операции pending/fulfilled/reject
  - промисификация callback
  - контроль асинхронной операции (прервать, не запустить)
  - набор инструментов, расширяющих информацию о протекающей асинхронной операции (thunkApi, анализ возвращенной ошибки через action.meta, unwrap - распаковка данных, ...)

- у любой асинхронной операции есть начало, время выполнения и завершение успешное/нет. И мы прикрепляем к этим моментам времени dispatches. dispatch pending/fulfilled/reject
- промисификация - перевод callback на рельсы promise. Значит, что мы как программисты к результату callback (асинхронный/синхронный) операции привязываемся через promise. Мы модифицируем этот callback, чтобы он возвращал результат в promise
- user callback принимает 2 параметра:
  - payload для функционального action (см. выше). Пример: `fetchUserById = createAsyncThunk(...), fetchUserById(3), 3 - payload нашего функционального action`
  - thunkApi - помогает осуществлять контроль асинхронной операции. Например, origDispatch, signal, rejectWithValue, .... Полный перечень см. доку
- набор инструментов - часть берется из options, часть из thunkApi, часть из внутренней логики asyncThunk. Полный перечень см. доку. Например:
  - idGenerator позволяет генерить уникальный requestId (часть options)
  - extra from reduxThunk middleware (часть thunkApi)
  - unwrap, unwrapResult позволяют удобного извлекать payload из возвращенного promise от нашей thunk (часть внутренней логики asyncThunk)
 
___________________________________________________________________________________________________________________________
- createSlice - идея мини редакса.
- createSlice принимает 3 параметра:
   - name                (для actions type)
   - initialState        (определяет форму slice)
   - reducers            (в своей работе полагается на автоматически сгенерированные slice actions, куча способов задать reducers, см. доку)
   - extraReducers       (полагается на внешние НЕ slice actions)
   - reducerPath         (путь хранения мини redux в глобальном redux)
   - selectors           (пользовательская логика deriving данных)
- createSlice возвращает кучу барахла:
   - name                (префикс actions)
   - reducer             (очевидно)
   - actions             (набор actionCreators авматически сформированных по name)
   - caseReducers        (все caseReducers, включая взятые из reducers и extraReducers)
   - getInitialState     (очевидно)
   - reducerPath         (ключ state, т.е. это по сути statePath, это кусок state в rootState, который изменяет данный reducer)
   - selectSlice         (логика селектирования root state, по умолчанию селектируется по state[reducerPath])
   - selectors           (обёртка над композицией selectSlice(который идет по умолчанию в slice, по state[reducerPath]), userSelectors(options.selectors))
   - getSelectors        (обёртка над композицией selectSlice(мы его указываем лично сами), userSelectors(options.selectors))
   - injectInto          (под капотом эта команда переадресует нас на combineSlice.inject, см. туда)
- Философия:
- ![wrapped Selectors](/img/wrapped_selector.png)
- state redux - единый источник информации в приложении. По итогу он содержит много данных, которыми трудно управлять. Возникает идея этот набор данных поделить. slice - reducer, который определяет кусок state. Акцент не на state, а на reducer. Основное, что возвращает slice - reducer, а остальное - обвесы. Вместе с state наворчивают обвесы в виде reducer, actionCreators, selectors. createSlice позволяет автоматизировать создание slice с обвесами. reducer, actionCreators идут из redux и не требуют дополнительных объяснений
- selectors - на входе и на выходе createSlice - разные вещи!!! selectors на входе - пользовательская логика нарезки state самого slice. selectors на выходе - wrapped обертка (принимает rootState, не sliceState), которая применяет пользовательские входные selectors после selectSlice

___________________________________________________________________________________________________________________________
- combineSlice - то же, что и combineReducer, т.е. собрать кучу reducers в один корневой
- combineSlice принимает:
  - куча reducers может быть представлены в 2 форматах (sliceObjects/reducersMap). sliceObjects - slice1, slice2, .../ reducersMap - отдельный объект, где ключ - reducersPath, value - reducer
- combineSlice возвращает:
  - большой reducer с обвесами. Обвесы: inject (вставить slice потом), selector (wrapper, чтобы вставленный slice селектировал значение по умолчанию, полагается на работу Proxy state), withLazyLoadedSlices (добавление типизации ленивого slice)
- Философия:
  - автоматизация по отношению к slice
  - возможность донастройки с помощью обвесов (см. возвращаемые значения)
______________________________________________________________________________________________________________________
- configureStore - обёртка над createStore. Суть: одним конфигурационным объектом можно передать все настройки store (самое главное: reducers, middleware), раньше отделььными командами. Раньше можно было пробросить только root reducer. Если хотел много, то надо было руками через combineReducer создать rootReducer. А теперь можно пробросить объект, а configureStore под капотом опираясь на combineReducer сам за тебя это сделает. Существуют доп. опции вроде devTools, enhancers, см. доку
______________________________________________________________________________________________________________________
- createEntityAdapter - переходник между нормализованными данными и partreducers
- combineSlice принимает:
  - объект настроек. Callback выборки id, callback сортировки entities
- combineSlice возвращает:
  - объект настроек
  - кучу caseReducers
  - selectors
  - метод получения state нормализованной формы ({ids: [], entities: {}}) getInitialState. Его можно донастроить:
    - первоначальными данными
    - дополнительными свойствами кроме ids и entities
- Философия:
  -  это partreducers, созданные через createSlice или createReducer
  -  нормализацию данных заранее должен сделать сам программист. createEntityAdapter нормализацию не проводит
  -  данные ДОЛЖНЫ иметь следующий интерфейс {ids: [], entities: {}}
  -  есть add/insert, set, update, upsert:
    -  add - добавить новую сущность чаще всего в конец коллекции
    -  insert - добавить новую сущность в коллекцию с акцентом на позицию
    -  set - добавить новую сущность, если не существует в коллекции. Заменить ПОЛНОСТЬЮ, если существует
    -  update - merge новых свойств в сущность, если та существует
    -  upsert - update + insert/add
  -  global vs globalized:
    - global - статичное определение чего-то всеобъемлющего
    - globalized - то, что применено/создано на основе global
