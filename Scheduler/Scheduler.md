# Scheduler
> 스케줄러는 스레드를 지정할 수 있다.
* subscribeOn(): Observable에서 구독자가 subscribe()를 호출했을 때 데이터 흐름을 발행하는 스레드를 지정하는 함수
  - 처음 호출한 그대로 스레드를 사용한다.
  - 또 다시 subscribeOn()를 호출해도 무시하게 된다.
  - 지정하지 않을 시 main 스레드를 기본적으로 지정하게 된다.

* observeOn(): 처리된 결과를 구독자에게 전달하는 스레드를 지정하는 함수
  - 지정하지 않을 시 subscribeOn()에서 지정한 스레드를 자동적으로 지정하게 된다.
  - 또 다시 observeOn()를 호출하면 호출한 함수에서 지정한 스레드로 변경된다.

## 스케줄러 종류
> 각 스케줄러마다 특정한 역할을 가지고 있고 사용자는 상황에 맞게 스케줄러를 지정하자

#### 뉴 스레드 스케줄러: 새로운 스레드를 생성
- 요청할 때마다 새로운 스레드를 생성한다.
```
val data = arrayOf("1", "3", "5")
val func = { it: String -> println("${Thread.currentThread().name}  $it") }

Observable
        .fromArray(*data)
        .doOnNext { func(it) }
        .map { "<<$it>>" }
        .subscribeOn(Schedulers.newThread())
        .subscribe { func(it) }

Thread.sleep(1000)

Observable
        .fromArray(*data)
        .doOnNext { func(it) }
        .map { "<<$it>>" }
        .subscribeOn(Schedulers.newThread())
        .subscribe { func(it) }

Thread.sleep(1000)
```

#### 계산 스케줄러: CPU에 대응하는 계산용 스케줄러
- 대기 시간 없이 빠르게 결과를 도출하는데 초점 맞춰 있다.
- 계산 작업: 입출력이 없는 작업
- 내부적으로 스레드 풀을 생성한다.
- 스레드 개수는 기본적으로 프로세스 개수와 같다.
```
val data = arrayOf("1", "3", "5")
val func = { it: String -> println("${Thread.currentThread().name}  $it") }

val source: Observable<String> = Observable
        .fromArray(*data)
        .zipWith(Observable.interval(100L, TimeUnit.MICROSECONDS),
                BiFunction { a, b -> a }
        )

source.map { "<<$it>>" }
        .subscribeOn(Schedulers.computation())
        .subscribe { func(it) }

source.map { "##$it##" }
        .subscribeOn(Schedulers.computation())
        .subscribe { func(it) }

Thread.sleep(1000)
```

#### IO 스케줄러: 네트워크 상의 요청을 처리하거나 각종 입출력 작업을 실행하기 위한 스케줄러
- 필요할 때마다 스레드를 계속 생성한다.
- 네트워크 특성상 대기 시간이 길다.
```
val root = "C:\\"
val func = { it: String -> println("${Thread.currentThread().name}  $it") }
val files: MutableList<File> = File(root).listFiles().toMutableList()

val source: Observable<String> = Observable
        .fromIterable(files)
        .filter { it.isDirectory }
        .map { it.absolutePath }
        .subscribeOn(Schedulers.io())

source.subscribe { func(it) }
Thread.sleep(500)
```

#### 트램펄린 스케줄러: 스레드를 생성하지 않고 현재 스레드를 무한한 크기의 대기 큐를 생성하는 스케줄러
- 새로운 스레드를 생성하지 않는다.
- 대기 큐를 자동으로 만들어 준다.
```
val orgs = arrayOf("1", "2", "3")
val func = { it: String -> println("${Thread.currentThread().name}  $it") }
val source: Observable<String> = Observable.fromArray(*orgs)

source.subscribeOn(Schedulers.trampoline())
        .map { "<<$it>>" }
        .subscribe { func(it) }

source.subscribeOn(Schedulers.trampoline())
        .map { "##$it##" }
        .subscribe { func(it) }

Thread.sleep(500)
```

#### 싱글 스레드 스케줄러: 단일 스레드
- 말 그대로 스레드가 한 개이다.
- 새로운 스레드를 생성하지는 않고 RxJava 내부에서 한 번만 생성한다.
- 잘 사용하지는 않는다.
```
val numbers: Observable<Int> = Observable.range(100, 5)
val chars: Observable<String> = Observable
        .range(0, 5)
        .map { it.toString() + "S" }
val func = { it: String -> println("${Thread.currentThread().name}  $it") }


numbers.subscribeOn(Schedulers.single())
        .subscribe { func(it.toString()) }
chars.subscribeOn(Schedulers.single())
        .subscribe { func(it) }

Thread.sleep(500)
```