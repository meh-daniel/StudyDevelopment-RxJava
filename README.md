# StudyDevelopment-RxJava

# Что такое RxJava?

RxJava - это библиотека реактивного программирования для составления асинхронных и событийных программ с использованием наблюдаемых последовательностей.

Реактивное программирование основано на потоках данных и распространении изменений. С помощью реактивного программирования можно легко выражать статические (например, массивы) или динамические (например, излучатели событий) потоки данных.

## Объясните, как «Как будто мне пять лет», что такое RxJava?

Допустим, Донна — кассир McDonald's, и менеджер думает, что она либо крадет наличные, либо выдает неправильную сдачу. Поэтому он попросил Джоша следить за Донной и отчитываться перед ним обо всем, что она делает.

Джош внимательно наблюдал за тем, как Донна дважды выдала неправильную сдачу и по ошибке уронила некоторые изменения, так как у нее состояние подергивания руки.

Джош, будучи хорошим сотрудником, которым он является, немедленно сообщил своему менеджеру о том, как происходили события.

В этой ситуации Джош — Наблюдатель, а Донна — данные. Джошу было сказано наблюдать и сообщать Донне о том, как меняется ее состояние, и он должен сделать обратный звонок тому, кто слушает его (менеджера).

## Наглядный пример на псевдоКотлине
```kotlin
val donna: Observable<Mistakes>
val josh: Subscriber

donna = Observable.just(
  Mistakes("Wrong change"),
  Mistakes("Dropped change"),
  Mistakes("Wrong change")
)

josh = donna.subscribe({ whatHappened -> reportToMangement(whatHappened) })
```

## Теперь, где вы можете использовать RxJava?
  Существует множество мест, где вы можете использовать RxJava, и ниже приведены наиболее распространенные места, где вы можете его реализовать:
  1. Сетевые вызовы (например, вызовы API через HTTP с функцией Модернизации, которая полностью поддерживает RxJava);
  2. События пользовательского интерфейса, которые должны вызывать действия;
  3. Чтение и запись базы данных и/или файлы в системе;
  4. Данные, поступающие с датчиков;
  5. И так далее..

# База 1 - Основы

Наблюдаемый(observable) — это место, откуда исходит поток данных, он выполняет некоторую работу и выдает значения.

Оператор(operator)- имеет возможность изменять данные из одной формы в другую.

Наблюдатель(observer)- получает значения.

<img src="https://github.com/meh-daniel/StudyDevelopment-RxJava/blob/main/photo-for-readme/Observeable-operator-observer.png" width="600" height="600">

Подумайте об этом так: Наблюдаемый — это Говорящий, Оператор — Это Переводчик, а Наблюдатель — Слушатель..

## Давайте создадим Observable
Есть много способов сделать это, и мы перечислим некоторые из них ниже. Примеры могут стать несколько сложными, но не торопитесь, чтобы понять, что происходит в каждой строке.

### Just
Оператор just преобразует Item в Наблюдаемый и излучает его.
```kotlin
Observable.just("Hello Reactive World")
    .subscribe { value -> println(value) }
```
Результат:
```kotlin
Hello Reactive World
```
Давайте добавим к этому некоторую сложность. Мы хотим знать, когда элемент получен, если есть ошибка и когда она завершается.
```kotlin
Observable.just("Apple", "Orange", "Banana")
    .subscribe(
        { value -> println("Received: $value") }, // onNext
        { error -> println("Error: $error") },    // onError
        { println("Completed!") }                 // onComplete
    )
```
Результат:
```kotlin
Received: Apple
Received: Orange
Received: Banana
Completed!
```
В этой ситуации у нас есть onNext , onError , и onComplete в лямбда-выражении. Их названия в значительной степени говорят сами за себя, но я хотел бы сгенерировать ошибку, чтобы проверить ее. Не беспокойтесь о методе map на данный момент, так как мы поговорим об этом позже.
```kotlin
Observable.just("Apple", "Orange", "Banana")
    .map({ input -> throw RuntimeException() } )
    .subscribe(
        { value -> println("Received: $value") },
        { error -> println("Error: $error") },
        { println("Completed!") }
    )
```
Результат:
```kotlin
I/System.out: Error: java.lang.RuntimeException
```

### From*

Есть несколько способов, которые вы можете использовать, и некоторые из них перечислены ниже:

```kotlin
Observable.fromArray("Apple", "Orange", "Banana")
    .subscribe { println(it) }
```
Результат:
```kotlin
Apple
Orange
Banana
```
Другой пример:
```kotlin
Observable.fromIterable(listOf("Apple", "Orange", "Banana"))
    .subscribe(
        { value -> println("Received: $value") },      // onNext
        { error -> println("Error: $error") },         // onError
        { println("Completed") }                       // onComplete
    )
```
Результат:
```kotlin
Received: Apple
Received: Orange
Received: Banana
Completed
```

### Create
C помощью create вы можете создать Observable с нуля. Рассмотрим пример.
Сначала создадим функцию, которая преобразует список в Observable:
```kotlin
fun getObservableFromList(myList: List<String>) =
    Observable.create<String> { emitter ->
        myList.forEach { kind ->
            if (kind == "") {
                emitter.onError(Exception("There's no value to show"))
            }
            emitter.onNext(kind)
        }
        emitter.onComplete()
```
Приведенная выше функция создаст string типа Observable, а затем прочитает каждый элемент списка и выполнит проверку, если он пуст, а если это так, то должна отображаться ошибка, в противном случае переходите к следующему до завершения.

Во-вторых, давайте вызовем эту функцию из onCreate, чтобы протестировать ее. Запишите исключение onError, так как мы проверим его позже.
```kotlin
getObservableFromList(listOf("Apple", "Orange", "Banana"))
    .subscribe { println("Received: $it") }
```
И вышесказанное приведет к:
```kotlin
Received: Apple
Received: Orange
Received: Banana
```

Теперь давайте протестируем onError, просто удалив строку "Orange", заменив ее пустой строкой и добавив onError в метод subscribe.
```kotlin
getObservableFromList(listOf("Apple", "", "Banana"))
    .subscribe(
        { v -> println("Received: $v") },
        { e -> println("Error: $e") }
    )
```
Результат:
```kotlin
Received: Apple
Error: java.lang.Exception: There's no value to show
```
Обратите внимание, что на этот раз мы получили только первый элемент, и так как была ошибка со вторым, он прервал поток данных, и появилось сообщение об ошибке.

### Interval
Эта функция создаст бесконечную последовательность тиков, разделенных заданной длительностью.
```kotlin
Observable.intervalRange(
    10L,     // Start
    5L,      // Count
    0L,      // Initial Delay
    1L,      // Period
    TimeUnit.SECONDS
).subscribe { println("Result we just received: $it") }
```
Результат:
```kotlin
Result we just received: 10
Result we just received: 11
Result we just received: 12
Result we just received: 13
Result we just received: 14
```


В приведенном выше примере Observable излучает каждую секунду. Давайте проверим простой, бесконечный интервал.
```kotlin
Observable.interval(1000, TimeUnit.MILLISECONDS)
    .subscribe { println("Result we just received: $it") }
```
Результат:
```kotlin
Result we just received: 0
Result we just received: 1
Result we just received: 2
...
```



# База 2 - Emmiter

## Backpressure
Backpressure(противодавление) - это процесс обработки излучателя, который будет производить много предметов очень быстро. Предположим, что Observable производит 100000 элементов в секунду, как подписчик, который может обрабатывать только 100 элементов в секунду, будет обрабатывать эти элементы?

Класс Observable имеет неограниченный размер буфера, он буферизует все и передает подписчику, и если он излучает больше, чем может обработать, вы обязательно получите OutOfMemoryException .

Мы можем обрабатывать такой поток данных, если мы применяем Backpressure к элементам по мере необходимости, таким образом, ненужные элементы будут выброшены или даже дадут производителю знать, когда создавать или когда отправлять вновь выпущенный элемент.


### Как можно решить эту проблему?

Решение простое. Вместо Observable вы можете использовать Flowable, который будет обрабатывать обратное давление для вас, поскольку он принимает его во внимание, в то время как Observable нет.

Думайте об этом как о воронке, когда в нее течет слишком много жидкости. Это представление Observable :

<img src="https://github.com/meh-daniel/StudyDevelopment-RxJava/blob/main/photo-for-readme/backpressure1.png" >

И с Flowable, принимая во внимание обратное давление, вы получите:

<img src="https://github.com/meh-daniel/StudyDevelopment-RxJava/blob/main/photo-for-readme/backpressure2.png">

Давайте запрограммируем пример обратного давления и решение.

```kotlin
val observable = PublishSubject.create<Int>()
observable.observeOn(Schedulers.computation())
.subscribe (
{
    println("The Number Is: $it")
},
{t->
    print(t.message)
}
)
for (i in 0..1000000){
    observable.onNext(i)
}
```
Приведенный выше код может привести к OutOfMemoryException если устройство не является первоклассным.

Чтобы справиться с обратным давлением в этой ситуации, мы преобразуем его в Flowable.

```kotlin
val observable = PublishSubject.create<Int>()
observable
.toFlowable(BackpressureStrategy.DROP)
.observeOn(Schedulers.computation())
.subscribe (
{
    println("The Number Is: $it")
},
{t->
    print(t.message)
}
)
for (i in 0..1000000){
    observable.onNext(i)
}
```
Приведенный выше код использует стратегию обратного давления DROP, которая удалит некоторые элементы, чтобы сохранить возможности памяти.
Теперь, когда мы поговорили о Backpressure, который является важной частью RxJava, давайте перейдем к другим типам Emitte(излучателей).

## Типы emmiter(излучатели)

Мы много говорили о Observable, однако есть и другие типы излучателей, которые можно использовать вместо Observable. Давайте поговорим о некоторых из них.

### Flowable

Он работает точно так же, как Observable но поддерживает противодавление.

```kotlin
Flowable.just("This is a Flowable")
.subscribe(
     { value -> println("Received: $value") },
     { error -> println("Error: $error") },
     { println("Completed") }
)
```

### Maybe

Этот класс используется, когда требуется вернуть одно необязательное значение. Методы являются взаимоисключающими, другими словами, вызывается только один из них. Если есть выданное значение, он вызывает onSuccess , если значения нет, он вызывает onComplete или, если есть ошибка, он вызывает onError .

```kotlin
Maybe.just("This is a Maybe")
    .subscribe(
        { value -> println("Received: $value") },
        { error -> println("Error: $error") },
        { println("Completed") }
    )
```

### Single

Он используется, когда есть одно возвращаемое значение. Если мы используем этот класс и есть выдаваемое значение, будет onSuccess. Если значения нет, будет вызван onError.

```kotlin
Single.just("This is a Single")
    .subscribe(
        { v -> println("Value is: $v") },
        { e -> println("Error: $e")}
    )
```

### Completable

Завершенный файл не будет выдавать никаких данных, он позволяет узнать, была ли операция успешно завершена. Если это так, он вызывает onComplete, а если нет, то вызывает onError . Типичным вариантом использования completable является REST API, где успешный доступ вернет HTTP 204 , а ошибки могут отличаться от HTTP 301, HTTP 404, HTTP 500 и т. Д. Мы могли бы что-то сделать с информацией.

```kotlin
Completable.create { emitter ->
    emitter.onComplete()
    emitter.onError(Exception())
}
```


# База 3 - Планировщики

отя RxJava активно продается как асинхронный способ выполнения реактивного программирования, важно уточнить, что RxJava по умолчанию является однопоточным, и вам нужно указать иное, и именно здесь появляются планировщики.

Краткое напоминание о разнице между синхронным и асинхронным.

