# Operator2
> RX자바 연산자는 여러 각 역할을 담당하고 연산자에서 파생된 파생 연사자도 존재한다.  
모든 연산자를 설명하는 것은 어렵지만 알면 좋은 것은 아래에 서술하겠다.

## 생성 연산자
> 데이터 흐름, Observable을 생성하는 연산자

#### interval(): 일정 시간 간격으로 데이터 흐름을 생성
- interval(long period, TimeUnit unit): 일정시간을 지연하고 unit에 처리를 정의
- interval(long initialDelay, long period, TimeUnit unit): 최초 지연시간을 지정할 수 있고 나머지는 위와 같다.
- interval(): 영원히 지속 실행, 폴링 용도로 쓰임
- 계산 스케쥴러에서 실행
```
val source: Observable<Long> = Observable.interval(100, TimeUnit.MICROSECONDS)
            .map { data -> (data + 1) * 100 }
            .take(5)
source.subscribe{ println(it) }
Thread.sleep(1000)
```

#### timer(): 일정 시간이 지난 후 한 개의 데이터를 발행
- 발행과 동시에 onComplete() 발생

```
val startTime = System.currentTimeMillis()

val source = Observable.timer(500L, TimeUnit.MILLISECONDS)
            .map { SimpleDateFormat("yyyy/MM/dd HH:mm:ss").format(Date()) }

source.subscribe {
    val time = System.currentTimeMillis() - startTime
    println("${Thread.currentThread().name} | $time | value = $it")
}

Thread.sleep(1000)
```

#### range(): 주어진 값부터 원하는 개수의 Int 객체를 발행
- range(final int start, final int count): start는 시작 값, count는 개수
- 반복문 대체 가능
```
val source: Observable<Int> = Observable
            .range(1, 10)
            .filter{ num -> num % 2 == 0 }
source.subscribe{ println(it) }
```

#### defer(): 데이터 흐름 생성을 구독자가 subscribe()를 호출하기 전까지 미룸
- 인자로 Callabel\<Observable\<T>>
- call(): 비로소 데이터를 발행하는 함수
```
private val colors = listOf("1", "3", "5", "6").iterator()

fun main() {
    val supplier = { getObservable() }
    val source = Observable.defer(supplier) 
    source.subscribe { val1 -> println("subscriber #1 : $val1") }
    source.subscribe { val2 -> println("subscriber #2 : $val2") }
}

private fun getObservable(): Observable<String> {
    return if (colors.hasNext()) {
        val color = colors.next()
        Observable.just("$color-B", "$color-R", "$color-P")
    } else {
        Observable.empty()
    }
}
```

#### repeat(): 단순 반복하는 함수
- 서버 통신 연결 확인용으로 잘 사용
- repeate(): 무한 반복
- repeate(N): N번 반복
```
val balls = arrayOf("1", "3", "5")
val source: Observable<String> = Observable.fromArray(*balls).repeat(3)
source.doOnComplete{ println("onComplete") }.subscribe{ println("Repeat value: $it") }
```