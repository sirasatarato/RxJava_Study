# Operator1
> RX자바 연산자는 여러 각 역할을 담당하고 연산자에서 파생된 파생 연사자도 존재한다.  
map(), flatMap(), filter(), reduce()는 연산자 중에서 모든 함수형 연산의 기본이 되는 연산자이다.

## map()
> 입력값을 어떤 함수에 넣어서 원하는 값으로 변환하는 함수
```
val balls = arrayListOf("1", "2", "3", "5")
val source: Observable<String> = Observable.fromIterable(balls).map{ball -> ball + "O"}
source.subscribe {println(it)}
```

## flatMap()
> 입력값을 어떤 함수에 넣어서 원하는 값으로 변환한 데이터를 가진 Observable을 반환하는 함수
```
val balls = arrayListOf("1", "2", "3", "5")
val source: Observable<String> = Observable.fromIterable(balls).flatMap{ball -> Observable.just(ball + "O", ball + "X")}
source.subscribe {println(it)}
```

## filter()
> Observable에서 원하는 데이터만 걸러내는 함수
```
val data = 0..10
val source: Observable<Int> = Observable.fromIterable(data).filter{num -> num % 2 == 0}
source.subscribe {println(it)}
```

## reduce()
> Observable에서 발행한 데이터를 모두 사용하는 함수
```
val balls = arrayListOf("1", "3", "5")
val source: Maybe<String> = Observable.fromIterable(balls).reduce{ball1, ball2 -> "$ball2($ball1)"}
source.subscribe {println(it)}
```