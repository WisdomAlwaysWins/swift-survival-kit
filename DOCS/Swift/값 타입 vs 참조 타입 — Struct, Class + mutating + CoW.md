#swift #struct #class #memory 

>[!info] 값 타입과 참조 타입, 왜 중요한가?
>Swift에서 struct와 class를 고르는 것은 "데이터가 복사될 때 어떤 일이 일어나는가"를 결정하는 것이다. 변수를 넘길 때 값이 통째로 복사되는지, 같은 객체를 여러 변수가 공유하는지 — 이 차이가 버그, 성능, 스레드 안전성에 직접 영향을 준다.

## 1. 개념 정리
#### 1.1 값 타입 vs 참조 타입 — 핵심 차이

##### 1.1.1 값 타입 (Value Type)

- 변수에 값 자체가 들어간다. 
- 다른 변수에 넘기면 복사본이 만들어진다. 
- 서로 독립적이다
- Struct, Enum, Tuple, 기본 타입

##### 1.1.2 참조 타입 (Reference Type)

- **주소(포인터)** 가 들어간다.
- 다른 변수에 넘기면 같은 객체를 가리키는 포인터가 복사된다.
- 하나를 바꾸면 다른 쪽도 바뀐다.
- Class, Closure

```swift
struct Wallet { // 값 타입
	var balance: Int
}

var myWallet = Wallet(balance: 10000)
var yourWallet = myWallet // 값 복사
yourWallet.balance = 0

print(myWallet.balance) // 10000 (영향 없음) 
rint(yourWallet.balance) // 0

// 참조 타입
class BankAccount { 
	var balance: Int 
	
	init(balance: Int) { self.balance = balance } 
}

var myAccount = BankAccount(balance: 10000) 
var yourAccount = myAccount // 주소 복사
yourAccount.balance = 0

print(myAccount.balance) // 0 (같이 바뀜) 
print(yourAccount.balance) // 0
```

#### 1.2 메모리에서의 차이

##### 1.2.1 값 타입 

값 자체가 Stack에 저장되고 독립적인 복사본이 생성된다.

![[값타입.png]]

##### 1.2.2 참조 타입

둘 다 같은 주소로, 하나의 객체를 공유한다.

![[참조타입.png]]

#### 1.3 비교표

| 특성               | 값 타입 (Struct) | 참조 타입 (Class)                |
| ---------------- | ------------- | ---------------------------- |
| 메모리 위치           | 일반적으로 Stack   | Heap                         |
| 할당 속도            | 빠름            | 느림                           |
| 복사 동작            | 깊은 복사 (독립)    | 얕은 복사 (공유)                   |
| 변경 영향            | 자신만           | 같은 객체를 가리키는 모든 참조            |
| mutating 필요      | 필요            | 불필요                          |
| 상속               | 불가능           | 가능                           |
| deinit           | 없음            | 있음                           |
| ARC 관리           | X             | O                            |
| Identity (`===`) | 없음            | 있음                           |
| 스레드 안전성          | 높음 (복사니까)     | 낮음 (공유 상태)                   |
| 멀티스레드 시          | 각자 복사본 → 안전   | 같은 객체 접근 → Race Condition 위험 |
>[!abstract] `===` 
>동일성 연산자로 class인 두 참조 변수가 같은 객체를 가리키는지 확인한다. 
>```swift
>let a = BankAccount(balance: 100)
>let b = a
>let c = BankAccount(balance: 100)
>
>a === b // true (같은 객체)
>a === c // false (다른 객체, 값만 같음)
>```

---
## 2. mutating 키워드

값 타입 (struct, enum)의 메서드에서 자신의 프로퍼티를 변경하겠다는 선언이다. 값 타입은 기본적으로 메서드 안에서 `self` 를 변경할 수 없는데, `mutating` 를 붙이면 가능해진다. 내부적으로 self 전체를 새로운 값으로 교체하는 것이다.

```swift
struct Score {
	var point = 0
	
	// mutating 없어서 컴파일 에러
	//func addPoint() {
	//	point += 1
	//}
	
	// mutating 있으면 self 변경 가능 👍
	mutating func addPoint() {
		point += 1
	}
	
	// mutating은 self 전체를 교체할 수도 있다
	mutating func reset() {
		self = Score(point: 0) // 새 인스턴스로
	}
}

var score = Score(point: 5)
score.addPoint() // score.point = 6
score.reset() // score.point = 0
```

>[!question] 왜 class는 mutating이 필요 없는가?
>class는 참조 타입이라 메서드 안에서 프로퍼티를 바꿔도 포인터(참조)는 바뀌지 않는다. 
>프로퍼티 변경 != 참조 변경
>반면 struct는 프로퍼티를 바꾸면 값 자체가 바뀌는 것이므로, "바꾸겠다"고 명시해야한다.

```swift
let fixScore = Score(point: 10)
// fixedScore.addPoint() // 🚩 let이므로 mutating 호출 불가
```

---
## 3. Copy-on-Write (CoW)

값 타입을 복사할 때 실제로는 바로 복사하지 않고, 수정이 일어나는 시점에 비로소 복사하는 최적화다. Swift의 Array, Dictionary, Set, String 등 표준 컬렉션에 적용되어 있다. 

**"복사는 했지만 아직 안 바꿨으니 바꾸는 순간에만 진짜 복사하자"**는 전략이다.

```swift
var original = [1, 2, 3, 4, 5]
var copy = original // 이 시점 : 실제 복사 안함

// 내부 상태:
// original, copy → [1, 2, 3, 4, 5] RC = 2

copy.append(6) // 이 시점 : 수정 발생 → 진짜 복사 실행

// 내부 상태:
// original → [1, 2, 3, 4, 5] RC = 1
// copy → [1, 2, 3, 4, 5, 6] RC = 1 (새 버퍼)
```

>[!question] CoW가 왜 필요한데 .. ?
>값 타입은 넘길 때마다 복사되는데, 배열 같은 큰 데이터를 매번 통째로 복사한다고 생각해보자. 엄청 느리겠지?
>CoW 덕분에 "읽기만 하는" 경우에는 복사 비용이 0이고, 수정할 때만 실제 복사가 일어나서 성능과 안전성을 동시에 잡는다.

---
## 4. Struct가 Heap에 저장되는 경우

>[!warning] 구조체가 항상 스택에 저장되는 것은 아니다!
>아래 3가지 경우에는 Heap으로 간다는 것을 명심하도록.

#### 4.1 클로저에 캡처될 때

```swift
func makeIncrementer() -> () -> Int {
	var count = 0
	
	return {
		count += 1 // 클로저가 count를 캡처
		return count // 클로저가 함수 스코프 밖에서 살아남으므로 count가 Heap으로 이동
	}
}

let inc = makeIncrementer()
inc() // 1
inc() // 2 → count가 Heap에서 유지되고 있음
```

#### 4.2 프로토콜 타입에 담길 때

```swift
protocol Describable {
	func describe() -> String
}

struct LargeStruct: Describable {
	var a: Int, b: Int, c: Int, d: Int
	
	func describe() -> String { "large" }
}

let item: Describable = LargeStruct(a: 1, b: 2, c: 3, d: 4)
```

#### 4.3 class의 프로퍼티일 때

```swift
struct Coordinate {
    var lat: Double, lng: Double
}

class MapPin {
    var position: Coordinate  // Struct이지만
    init(position: Coordinate) { self.position = position }
}

let pin = MapPin(position: Coordinate(lat: 37.5, lng: 127.0))
// MapPin 인스턴스 전체가 Heap → 그 안의 position도 Heap에 존재
```

---
## 5. Apple이 Struct를 권장하는 이유

Apple 공식 문서 "Choosing Between Structures and Classes"에서:

- **기본으로 Struct를 사용하라**
- 다음 경우에만 Class를 써라:
    - Objective-C와 상호운용이 필요할 때
    - Identity(`===`)가 필요할 때
    - 상속이 필요할 때 (하지만 프로토콜로 대체 가능한 경우가 많음)

---
## ❓ 스스로에게 물어봐

Q1. 값 타입과 참조 타입의 차이를 설명해주세요.

>값 타입은 변수에 값 자체가 저장되어 복사 시 독립적으로 복사본이 만들어집니다. 
>참조 타입은 변수에 포인터가 저장되어 복사 시 같은 객체를 공유합니다.
>
>값 타입은 일반적으로 스택에 할당되어 빠르고, 복사가 독립적이라 thread-safe 합니다. 
>참조 타입은 힙에 할당되어 느리지만, 하나의 객체를 여러 곳에서 공유할 수 있습니다.
>
>스위프트에서 struct, enum, tuple은 값 타입이고, class, closure는 참조 타입입니다.

Q2. Copy-On-Write를 설명해주세요.

>값 타입을 복사할 때 실제로는 바로 복사하지 않고 내부 버퍼를 공유합니다. 수정이 일어나는 시점에만 실제 복사가 발생합니다. 스위프트의 배열, 딕셔너리, 세트, 문자열에 적용되어있습니다.

Q3. mutating 키워드는 왜 필요한가요?

> struct는 값 타입이라 메서드 안에서 기본적으로 `self` 가 불변입니다. 프로퍼티를 변경하려면 self 전체를 새 값으로 교체해야 하는데, 이것을 허용하겠다고 명시하는 것이 mutating 입니다.
> class는 참조 타입이라 프로퍼티를 바꿔도 참조(포인터) 자체는 변하지 않으므로 mutating이 필요 없습니다.

Q4. struct 인스턴스가 heap에 할당되는 경우를 설명해주세요.

>1. 클로저에 캡처될 때
>2. 프로토콜 타입에 담길 때
>3. class의 프로퍼티일 때