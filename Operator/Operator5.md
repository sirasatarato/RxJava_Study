# Operator5
> RX자바 연산자는 여러 각 역할을 담당하고 연산자에서 파생된 파생 연사자도 존재한다.  
모든 연산자를 설명하는 것은 어렵지만 알면 좋은 것은 아래에 서술하겠다.

## 수학 및 기타 연산자
### 수학 함수
> RxJava의 확장 모튤(RxAndorid도 확장 모듈이다.)로 수학과 관련된 함수들을 가지고 있다.

#### Gradle
> https://github.com/akarnokd/RxJavaExtensions

#### list
1. count
2. max
3. min
4. sumInt
5. acerageDouble
6. etc...

### 기타 함수
#### delay: 지정된 시간만큼 데이터 발행을 미루는 함수
```
val data1 = arrayOf("1", "2", "3", "4")
val source: Observable<String> = Observable.fromArray(*data1).delay(100L, TimeUnit.MICROSECONDS)
source.subscribe{ println(it) }
Thread.sleep(1000)
```
#### timeInterval: 어떤 값을 발행했을 때 이전 값을 발행한 이후 얼마나 시간이 흘렀는지 알려주는 함수
```
val startTime = System.currentTimeMillis()
val data = arrayOf("1", "3", "5")
val source: Observable<Timed<String>> = Observable
        .fromArray(*data)
        .delay { item ->
            Thread.sleep(Random.nextInt(100).toLong())
            return@delay Observable.just(item)
        }.timeInterval()
source.subscribe {
    val time = System.currentTimeMillis() - startTime
    println("${Thread.currentThread().name} | $time | value = $it")
}
Thread.sleep(1000)
```