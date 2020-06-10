# Operator4
> RX자바 연산자는 여러 각 역할을 담당하고 연산자에서 파생된 파생 연사자도 존재한다.  
모든 연산자를 설명하는 것은 어렵지만 알면 좋은 것은 아래에 서술하겠다.

## 결합 연산자
> 다수의 Observable을 하나의 Observable로 만드는 연산자

#### zip(): 입력 Observable에서 데이터를 모두 새로 발행했을 때 그것을 합하는 함수
```
val num = arrayOf("1", "2", "3")
val alpha = arrayOf("a", "b", "c")
val source: Observable<String> = Observable.zip(
        Observable.fromArray(*num),
        Observable.fromArray(*alpha),
        BiFunction { s, n -> "$n$s" }
)

source.subscribe{ println(it) }
```

#### conbineLatest(): 2개 이상의 Observable을 기반으로 Observable 각각의 값이 변경도었을 때 갱신해주는 함수
```
val data1 = arrayOf("6", "7", "4", "2")
val data2 = arrayOf("DIAMOND", "STAR", "PENTAGON")
val source: Observable<String> = Observable.combineLatest(
        Observable.fromArray(*data1)
                .zipWith(Observable.interval(100L, TimeUnit.MILLISECONDS),
                BiFunction { t1, _ -> t1}),
        Observable.fromArray(*data2)
                .zipWith(
                Observable.interval(150L,200L,TimeUnit.MILLISECONDS),
                BiFunction { t1, _ -> t1}),
        BiFunction { v1, v2 -> "$v1 $v2" }
)

source.subscribe{ println(it) }

Thread.sleep(2000)
```

#### merge(): 먼저 입력되는 데이터를 그대로 발행
```
val data1 = arrayOf("1", "3")
val data2 = arrayOf("2", "4", "6")
val source1: Observable<String> = Observable
        .interval(0L, 100L, TimeUnit.MILLISECONDS)
        .map { it.toInt() }
        .map { data1[it] }
        .take(data1.size.toLong())
val source2: Observable<String> = Observable
        .interval(50L, TimeUnit.MILLISECONDS)
        .map { it.toInt() }
        .map { data2[it] }
        .take(data2.size.toLong())

val source: Observable<String> = Observable.merge(source1, source2)
source.subscribe{ println(it) }

Thread.sleep(1000)
```

#### concat(): Observable을 이어 붙이는 함수
```
val data1 = arrayOf("1", "3", "5")
val data2 = arrayOf("2", "4", "6")
val source1: Observable<String> = Observable.fromArray(*data1)
val source2: Observable<String> = Observable.fromArray(*data2)

val source: Observable<String> = Observable.concat(source1, source2)
source.subscribe{ println(it) }
```

## 조건 연산자
> Observable의 흐름을 제어하는 연산자

#### amb(): 여러 개의 Observable 중 가장 먼저 데이터를 발행하는 Observable을 가지는 함수
```
val data1 = arrayOf("1", "3", "5")
val data2 = arrayOf("2", "4", "6")
val source: List<Observable<String>> = listOf(
        Observable.fromArray(*data1),
        Observable.fromArray(*data2).delay(100L, TimeUnit.MICROSECONDS)
    )
Observable.amb(source).subscribe{ println(it) }
```

#### takeUntil(): 다른 Observable에서 값을 발행하기 전까지의 데이터를 가진 Observable을 가지는 함수
```
val data1 = arrayOf("1", "2", "3", "4", "5")
val source: Observable<String> = Observable.fromArray(*data1)
        .zipWith<Long, String>(Observable.interval(100L, TimeUnit.MILLISECONDS), BiFunction { t1, _ -> t1 })
        .takeUntil(Observable.timer(500L, TimeUnit.MILLISECONDS))
source.subscribe{ println(it)}

Thread.sleep(1000)
```

#### skipUntil(): 다른 Observable에서 값을 발행한 후의 데이터를 가진 Observable을 가지는 함수
```
val data1 = arrayOf("1", "2", "3", "4", "5")
val source: Observable<String> = Observable.fromArray(*data1)
        .zipWith<Long, String>(Observable.interval(100L, TimeUnit.MILLISECONDS), BiFunction { t1, _ -> t1 })
        .skipUntil(Observable.timer(500L, TimeUnit.MILLISECONDS))
source.subscribe{ println(it)}
Thread.sleep(1000)
```

#### all(): 주어진 조건을 만족하는지 확인 후 Single<Boolean>을 반환
```
val data1 = arrayOf("1", "2", "3", "4")
val source: Single<Boolean> = Observable.fromArray(*data1).map{ "BALL" }.all("BALL"::equals)
source.subscribe{it -> println(it) }
Thread.sleep(1000)
```