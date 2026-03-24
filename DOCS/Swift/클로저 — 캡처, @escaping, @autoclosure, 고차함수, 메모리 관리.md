#swift #closure #high-order-function

>[!info] 클로저, 왜 중요한가?
>클로저는 "이름 없는 함수"이면서, 주변 변수를 "캡처"할 수 있는 코드 블록이다. iOS에서 네트워크 콜백, 애니메이션, 정렬, 이벤트 핸들링 — 거의 모든 비동기 패턴이 클로저로 동작한다. 캡처 메커니즘을 모르면 메모리 누수를 만들고, `@escaping`을 모르면 컴파일 에러를 해결할 수 없다.

## 1. 개념 정리
#### 1.1 클로저의 3가지 형태

Apple 공식 문서에서 정의하는 클로저의 3가지 형태:

1. **Global Function**: 이름이 있고, 캡처 안 함
2. **Nested Function**: 이름이 있고, 바깥 함수의 값을 캡처 가능
3. **Closure Expression**: 이름 없고, 주변 컨텍스트의 값을 캡처 가능

>[!abstract] 모든 [[함수 — 1급 객체와 스택 프레임]]는 클로저의 한 형태다.
>우리가 보통 "클로저"라고 부르는 건 세 번째 형태(Closure Expression)이다.

#### 1.2 클로저 축약 문법 (5단계)

```swift
let prices = [ 3000, 1500, 4200, 800 ]

// 0단계: 일반 함수
func ascending(_ a: Int, _ b: Int) -> Bool { return a < b }

// 1단계: 클로저 표현식 (전체 작성)
prices.sorted(by: { (a: Int, b: Int) -> Bool in return a < b) }

// 2단계: 타입 추론 (파라미터/리턴 타입 생략)
prices.sorted(by: { a, b in return a < b })

// 3단계: 암시적 리턴
prices.sorted(by: { a, b in a < b })

// 4단계: 축약 인수 ($0, $1)
prices.sorted(by: { $0 < $1 })

// 5단계: 연산자 메서드 (연산자 자체가 함수니까)
prices.sorted(by: <)
```

#### 1.3 Trailing Closure

```swift
// 마지막 파라미터가 클로저면 괄호 밖으로 뺄 수 있다
prices.sorted { $0 < $1 }

// 파라미터가 클로저뿐이면 () 생략 가능
let doubled = prices.map { $0 * 2 }

// 다중 Trailing Closure (Swift 5.3+)
UIView.animate(withDuration: 0.3) {
    view.alpha = 0
} completion: { finished in
    view.removeFromSuperview()
}
```

#### 1.4 캡처 매커니즘

클로저가 자기 바깥에 있는 변수를 "붙잡아서" 클로저 안에서 사용하는 것이다. 기본적으로 **참조를 캡처**한다 — 값을 복사하는 게 아니라, 변수 자체에 대한 참조를 잡는다.

캡처된 변수는 Heap으로 이동한다. 클로저가 함수 스코프를 벗아나도 캡처된 변수가 살아있어야 하기 때문이다.

```swift
var score = 0
let addScore = {
	score += 10 // score 변수의 참조를 캡처
}

score = 50
addScore()
print(score) // 60 (50 + 10)

// 값 캡처 (캡처 리스트)
var level = 1
let showLevel = { [level] in // 캡처 시점의 값을 복사
	print(level)
}
level = 99
showLevel() // 1 (캡처 시점의 값)
```

#### 1.5 @escaping vs non-escaping

클로저가 함수가 반환된 후에도 살아있는 것을 허용하는 키워드다. 프로퍼티에 저장되거나, 비동기 콜백으로 나중에 실행되는 경우에 필요하다. 기본은 non-escaping (함수 안에서만 사용하고 끝)

```swift
// non-escaping
func process(action: () -> Void) {
	action() // 바로 실행하고 끝
}

// @escaping
class Downloader {
	var completion: (() -> Void)?
	
	func download(onDone: @escaping () -> Void) {
		completion = onDone // 프로퍼티에 저장 → 함수 반환 후에도 completion에 존재
	}
}
```

|               | non-escaping      | @escaping     |
| ------------- | ----------------- | ------------- |
| 수명            | 함수 실행 중에만         | 함수 반환 후에도 생존  |
| 저장            | 불가                | 프로퍼티에 저장      |
| `[weak self]` | 불필요               | 필요 (순환 참조 방지) |
| 최적화           | 컴파일러가 더 공격적으로 최적화 | 제한적           |

#### 1.6 @autoclosure

표현식을 **자동으로 클로저로 감싸는** 키워드다. 호출하는 쪽에서 `{ }` 없이 그냥 값을 쓰면, 컴파일러가 알아서 `{ 그 값 }` 클로저로 만들어준다. **지연 평가(lazy evaluation)** 에 사용한다.

```swift
// @autoclosure 없이
func check(_ condition: () -> Bool) {
	if condition() { print("통과") }
}

check({ 2 > 1 }) // { } 필요

// @autoclosure 있으면
func check(_ condition: @autoclosure () -> Bool) { 
	if condition() { print("통과") } 
}

check(2 > 1) // { } 없이 그냥 표현식
```

## 2. 고차 함수
#### 2.1 map — 각 요소를 변환

```swift
let temps = [36.5, 37.2, 38.1, 36.8]
let fahrenheit = temps.map { $0 * 9/5 + 32 }
// [97.7, 98.96, 100.58, 98.24]
```

#### 2.2 filter — 조건에 맞는 요소만

```swift
let ages = [15, 22, 17, 30, 12, 25]
let adults = ages.filter { $0 >= 18 }
// [22, 30, 25]
```

#### 2.3 reduce — 하나의 값으로 합침

```swift
let bills = [12000, 8000, 15000, 3000]
let total = bills.reduce(0) { $0 + $1 }  // 38000
// 또는
let total2 = bills.reduce(0, +)  // 38000
```

#### 2.4 compactMap — 변환 + nil 제거

```swift
let inputs = ["10", "abc", "30", "xyz", "50"]
let numbers = inputs.compactMap { Int($0) }
// [10, 30, 50] — 변환 실패(nil)는 제거됨
```

#### 2.5 flatMap — 중첩 배열 평탄화

```swift
let nested = [[1, 2], [3, 4], [5]]
let flat = nested.flatMap { $0 }
// [1, 2, 3, 4, 5]
```

#### 2.6 forEach — 각 요소에 작업 수행

```swift
["Swift", "Kotlin", "Dart"].forEach { print($0) }
```

---
## ❓ 스스로에게 물어봐

Q1. @escaping과 non-escaping의 차이는?

>none-escaping은 기본값으로, 클로저가 함수 실행중에만 존재한다. @escaping은 클로저가 함수 반환 후에도 살아있어서 프로퍼티에 저장되거나 비동기 콜백으로 사용될 때 필요하다. self를 캡처할 수 있으므로 `[weak self]` 로 순환 참조를 방지해야한다.

Q2. 캡처에서 참조 캡처, 값 캡처는?

>기본 캡처는 변수의 참조를 잡아서, 클로저 안팎으로 같은 변수를 공유한다. 캡처 리스트 `[x]`를 쓰면 캡처 시점의 값을 복사해 클로저가 독립적인 복사본을 갖는다.

Q3. 다음 코드의 출력은?

```swift
var count = 0
let increment = { count += 1 }
count = 100
increment()
print(count)
```

>101 → 참조 캡처이므로 count = 100 + 1

Q3. 다음 코드의 출력은?

```swift
var x = 10
let capture = { [x] in print(x) }
x = 999
capture()
```

>10 → 캡처 리스트 `[x]`로 캡처 시점의 값(10)을 복사.

Q3. compactMap과 map의 차이는?

>map은 nil을 포함한 옵셔널 배열, compactMap은 nil이 제거된 일반 배열