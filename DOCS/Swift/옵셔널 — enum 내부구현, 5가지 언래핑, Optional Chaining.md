#swift #optional

>[!info] 옵셔널, 왜 중요할까?
>스위프트에서 "값이 없을 수 있다"를 표현하는 방법이 Optional이다. 스위프트 코드에서 `?`,`!`,`if let`,`guard let`,`??`를 보게 된다면? Optional을 알아야 무슨 의미인지 알 수 있을 것!

## 1. 개념 정리
#### 1.1 Optional의 내부 구현

```swift
// Optional의 진짜 모습 (Swift 표준 라이브러리)
enum Optional<Wrapped> {
	case non // 값 없음 = nil
	case some(Wrapped) // 값 있음
}

// 이것들은 전부 같은 의미
let a: Int? = 42
let b: Optional<Int> = .some(42)

let c: Int? = nil
let d: Optional<Int> = .none
```

>[!question] 왜 enum으로 만들었을까?
>"값이 있다 / 없다" 두 가지 상태를 표현하는 데 enum이 가장 적합하기 때문. `.some`에서 Associated Value는 실제 값이 붙어있고, `.none`은 값이 없는 상태다. [[열거형 — Associated Value, Raw Value, 패턴 매칭]]에서 정리한 Associated Value가 여기서 쓰인다.

#### 1.2 5가지 언래핑 방법

Optional과 일반 타입은 다른 타입!  `Int`와 `Int?`는 다른 타입이라 직접 연산할 수 없다.

##### 1.2.1 `if let` — 가장 기본

```swift
let optionalName: String = "Swift"

if let name = optionalName {
	print("이름: \(name)")
} else {
	print("이름 없음")
}

// Swift 5.7+ 이후로는 같은 이름으로 바인딩 가능
if let optionalName {
	print(optionalName) // 언래핑된 String
}
```

##### 1.2.2 `guard let` — 조기 종료

```swift
func greet(_ name: String?) {
	guard let name = name else {
		print("이름 몰라서 인사 못함")
		return // 스코프 탈출
	}
	
	print("안녕, \(name)")
}
```

>[!question] `if let`과 `guard let`의 차이가 뭘까?
>- `if let` : 값이 있을 때 실행할 코드를 `{   }` 안에 쓴다.
>- `guard let` :  값이 없을 때 바로 빠져나가고 이후 코드에서 언래핑된 변수를 계속 쓸 수 있다.

##### 1.2.3 ?? (nil coalescing) — 기본값 제공

```swift
let input: String? = nil
let displayName = input ?? "Guest" // nil이면 Guest 사용
print(displayName)

// 체이닝도 가능
let first: String? = nil
let second: String? = nil
let third: String? = "fallback"
let result = first ?? second ?? third ?? "default"
// result = fallback 
```

##### 1.2.4 ! (force unwrap) — 강제 언래핑

```swift
let definitelyExists: String? = "확실히 있음"
let value = definitelyExists!
print(value)

// nil이면 런타임 크래시
```

>[!danger] `!`는 최후의 수단으로 100% 값이 있다고 확신할 때만 사용
>그냥 거의 안 쓴다고 생각하자

##### 1.2.5 ?. (Optional Chaining) — 체인 접근

```swift
class Address {
	var city = "서울"
}

class Person {
	var address: Address?
}

let person: Person? = Person()
let city = person?.address?.city

// 중간에 nil이 아니라도 있으면 nil
// city: String?
```

>[!question] 왜 Optional Chaining의 결과는 항상 Optional이야?
>체인의 어느 한 지점이라도 `nil`일 수 있으므로, 최종 결과도 "nil일 수 있다" = Optional이어야 한다. `person`이 nil일 수도, `address`가 nil일 수도 있으니까.

#### 1.3 Optional Map과 flatMap

```swift
let optionalAge: Int? = 25
let nextYear = optionalAge.map { $0 + 1 } // Optional(26)

let nothing: Int? = nil
let nope = nothing.map { $0 + 1 } // nil

let optionalString: String? = "42"
let bad = optionalString.map { Int($0) } // Int?? → Optional(Optional(42))
let good = optionalString.flatMap { Int($0) } // Int? → Optional(42)
```

>[!abstract] map vs flatMap 차이가 뭔데?
>- map: 변환 함수의 결과를 그대로 Optional로 감싼다. 변환 함수가 Optional을 반환하면 이중 Optional(`??`)이 된다.
>- flatMap: 변환 함수가 Optional을 반환해도 한 겹만 남긴다.(평탄화) 변환 자체가 실패할 수 있을 때 사용

---
## ❓ 스스로에게 물어봐

Q1. Optional의 내부 구현

>Optional은 `enum Optional<Wrapped> { case none; case some(Wrapped) }`로 구현된 제네릭 열거형이다.  `nil`은 `.none`이고, 값이 있으면 `.some(값)`이다. Associated Value에 실제 값이 담겨 있어서, 언래핑은 사실 enum의 패턴 매칭으로 `.some` 케이스에서 값을 꺼내는 것.

Q2: if let과 guard let의 차이는?

>`if let`은 값이 있을 떄 실행할 코드를 중괄호 안에 쓰고, 언래핑된 변수를 그 스코프 안에서만 사용 가능하다.
>`guard let`은 값이 없을 때 즉시 스코프를 빠져나가고, 언래핑된 변수는 이후 코드 전체에서 사용 가능하다.
