# Operator3
> RX자바 연산자는 여러 각 역할을 담당하고 연산자에서 파생된 파생 연사자도 존재한다.  
모든 연산자를 설명하는 것은 어렵지만 알면 좋은 것은 아래에 서술하겠다.

## 변환 연산자
> 데이터의 흐름을 원하는 대로 변형하는 연산자 

#### concatMap(): flatMap() + 끼어들기 방지하는 함수
- flatMap()은 중간에 새로운 데이터가 들어오면 나중에 들어온 데이터의 처리 결과가 먼저 출력될 수 있다.
- concatMap()은 이를 방지하기 위한 함수이다.
- 중간에 끼어들어와도 순서대로 처리한다.
```
val startTime: Long = System.currentTimeMillis()
val balls = arrayOf("1", "3", "5")
val source = Observable.interval(100L, TimeUnit.MILLISECONDS)
        .map { it.toInt() }
        .map { balls[it] }
        .take(balls.size.toLong())
        .concatMap { ball ->
            Observable.interval(200L, TimeUnit.MILLISECONDS)
                    .map { notUsed -> "$ball <>" }
                    .take(2)
        }

source.subscribe {
    val time = System.currentTimeMillis() - startTime
    println("${Thread.currentThread().name} | $time | value = $it")
}

Thread.sleep(2000)
```

#### switchMap(): 순서를 보장하기 위해 진행중이던 작업을 중단하는 함수
- 새로운 값이 끼어들면 기존 값의 작업을 중단한다.
- 센서 같은 값을 얻어서 동적으로 처리하는 작업에 매우 유용하다. 
```
val startTime: Long = System.currentTimeMillis()
val balls = arrayOf("1", "3", "5")
val source = Observable.interval(100L, TimeUnit.MILLISECONDS)
        .map { it.toInt() }
        .map { balls[it] }
        .take(balls.size.toLong())
        .switchMap { ball ->
            Observable.interval(200L, TimeUnit.MILLISECONDS)
                    .map { "$ball <>" }
                    .take(2)
        }

source.subscribe {
    val time = System.currentTimeMillis() - startTime
    println("${Thread.currentThread().name} | $time | value = $it")
}

Thread.sleep(2000)
```

#### groupBy(): 어떤 기준으로 단일 Observable을 여러 개의 Observable 그룹으로 만드는 함수
```
val objs = arrayOf("6", "4", "2-T", "2", "6-T", "4-T")
    
val source = Observable.fromArray(*objs).groupBy {
    if (it.endsWith("-T")) "TRIANGLE"
    else "NO_SHAPE"
}
    
source.subscribe { obj ->
    obj.subscribe { value ->
        println("GROUP: ${obj.key} value:+$value")
    }
}
```

#### scan(): 실행할 때마다 중간 결과 및 최종 결과를 구독자에게 발행
- reduce()는 최종 결과만 주고 scan은 중간 값도 같이 준다.
```
val balls = arrayOf("1", "3", "5")
val source: Observable<String> = Observable.fromArray(*balls).scan{ ball1, ball2 -> "$ball2($ball1)" }
source.subscribe {println(it)}
```