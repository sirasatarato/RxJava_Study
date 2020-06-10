# Observable
> Observed라는 단어가 관찰을 통해서 얻은 결과를 의미한다면 Observable은 현재는 관찰되지 않았지만 이론을 통해서 앞으로 관찰할 가능성을 의미한다.

## 알림
> Observable은 세 가지의 알림을 구독자에게 전달한다.
* onNext: Observable이 데이터의 발행을 알림
* onComplete: 모든 데이터의 발행이 완료했음을 알림
* onError: Observable에서 어떠한 이유로 에러가 발생하였음을 알림

## 생성
> Observable을 생성할 때는 인스턴스를 만들지 않고 정적 팩토리 함수를 호출한다.

#### just(): 인자로 넣은 데이터를 차례대로 발행
- 인자수: 1 ~ 최대 10개 까지
- 인자 타입: 모두 동일

#### create(): Observable 알림을 직접 개발자가 호출해주도록 하는 함수
- 인자: 람다, 함수
- 람다 인자: ObservableEmitter\<T> or ObservableOnSubscribe\<T>

#### fromArray(): 배열을 인자로 발행
- 인자 타입: 배열

#### fromIterable(): 데이터 타입에 의존하지 않고 데이터만 발행
- 인자 타입: 컬렉션, 또는 관심 없음

## 구독
> 데이터를 발행하기 위해 구독을 한다. 구독하기 전까지는 데이터를 발행하지 않는다.

#### subscribe(): Observable을 구독하는 함수
- 반환 타입: Disposable
- subscribe(): onError만 감지, 보통 디버깅, 테스트용으로 사용
- subscribe(Consumer<? super T> onNext): onNext만 처리
- subscribe(Consumer<? super T> onNext, Consumer<? super java.lang.Throwable> onError): onNext와 onError 처리
- subscribe(Consumer<? super T> onNext, Consumer<? super java.lang.Throwable> onError, Action onComplete): onNext, onError, onComplete 처리

## 해제
> Observable이 더 이상 데이터를 발행하지 않도록 구독을 해지한다.

#### Disposable: 
- 형식: 인터페이스
- dispose(): Observable이 더 이상 데이터를 발행하지 않도록 구독을 해지하는 함수
- isDisposed(): Observable이 데이터를 발행하지 않는지 확인하는 함수


# Single 클래스
> 데이터 하나만 발행과 동시에 종료하는 Observable

## 생성
- fromObservable(): Observable를 받아서 Single로 변환
- single(): Observable를 받아서 Single로 변환
- first(): 데이터 하나만 받아서 Single로 생성

## 주의
> 데이터를 2개 이상 받을시 에러가 발생한다.

# Maybe 클래스
> 데이터를 하나만 발행할 수도 있고 아닐 수도 있다.  
Maybe = Single + onComplete()

# 뜨거운 Observable
- 차가운 Observable: 구독하지 않을시 데이터를 발행하지 않는 Observable
- 뜨거운 Observable: 구독 여부와 상관없이 데이터를 발행하는 Observable
  - 데이터를 받자마자 바로 발행한다.

## Subject 클래스
> Observable을 뜨거운 Observable로 변환하는 Observable

#### AsyncSubject 클래스
1. 마지막 데이터만 발행한다.
2. 구독자로도 사용이 가능하다.

#### BehaviorSubject 클래스
1. 구독을 할시 가장 최근 값을 혹은 기본값을 발행한다.

#### PublishSubject 클래스
1. 가장 평범한 Observable로 딱히 특별한 기능은 없고 해당 시간에 발생한 데이터를 그대로 구독자에게 전달한다.

#### ReplaySubject 클래스
1. 차가운 Observable처럼 동작한다.
2. 새로운 구독자가 생길시 항상 데이터의 처음부터 끝까지 발행한다.
3. 이 때문에 메머리 누수 문제가 발생할 수도 있다.

## ConnectableObservable 클래스
> subscribe()에 반응하지 않고 Connect()를 호출해야 그때까지 구독했던 구독자들에게 발행한 데이터를 받는다.
- 생성: publish()