>[!info] 제네릭, 왜 알아야하는가?
>제네릭은 "타입을 나중에 결정한다"는 개념이다. 같은 로직인데 Int용, String용, Double용 함수를 따로 만드는 건 낭비다. 제네릭으로 한 번만 쓰면 어떤 타입이든 동작한다. Swift 표준 라이브러리의 Array, Dictionary, Optional이 전부 제네릭으로 만들어져 있다.

## 1. 개념 정리
#### 1.1 제네릭 함수

타입을 파라미터로 받는 것이다. `<T>` 에서 `T`는 아직 정해지지 않은 타입을 뜻하는 자리 표시자다. 함수를 호출할 때 실제 타입이 결정된다.

>[!question] `<T>` 의 T는 무슨 뜻인가?
>*아무 뜻도 없다..* 
>`<Element>`, `<Value>`, `<Mewo>` 등 다 가능. 관례상 단일 문자를 쓰거나, 의미가 있으면 `Element`, `Key`, `Value` 같은 이름을 쓴다. 스위프트 표준 라이브러리에서 `Array<Element>`, `Dictionary<Key, Value>`, `Optional<Wrapped>` 같은 것들이 보이는 이유다.

```swift
// 제네릭 없이: 타입마다 함수를 따로 만들어야 함
func swapInts(_ a: inout Int, _ b: inout Int) { 
	let temp = a; a = b; b = temp 
} 
func swapStrings(_ a: inout String, _ b: inout String) { 
	let temp = a; a = b; b = temp 
}
```

```swift
// 제네릭: 한번에
func swapAnything<T>(_ a: inout T, _b: inout T) {
	let temp = a; a = b; b = temp
}
var x = 10, y = 20 
swapAnything(&x, &y) // T = Int로 결정 
print(x, y) // 20, 10 

var s1 = "hello", s2 = "world" 
swapAnything(&s1, &s2) // T = String으로 결정 
print(s1, s2) // "world", "hello"
```

#### 1.2 제네릭 타입

```swift
// 어떤 타입이든 담을 수 있는 스택
struct SimpleStack<Element> {
	private var items: [Element] = []
	
	mutating func push(_ item: Element) {
		items.append(item)
	}
	
	mutating func pop() -> Element? {
		return items.isEmpty ? nil : items.removeLast()
	}
	
	var top: Element? {
		return items.last
	}
}

var intStack = SimpleStack<Int>() 
intStack.push(1) 
intStack.push(2) 
print(intStack.pop()) // Optional(2) 

var stringStack = SimpleStack<String>() 
stringStack.push("Swift") 
stringStack.push("Python") 
print(stringStack.top) // Optional("Python")
```

#### 1.3 타입 제약 (Type Constraints)

`<T>`는 아무 타입이나 받을 수 있는데, 때로는 "이 타입은 최소한 이런 능력은 가지고 있어야 함"이라고 제한하고 싶을 때가 있다. 그때 `<T: 프로토콜>` 형태로 제약을 건다.

```swift
// T가 아무거나 되면 == 비교할 수가 없음
//func findIndex<T>(of target: T, in items: [T]) -> Int? {
//	for (i, item) in items.enumerated() {
//		if item == target { return i } // 🚩 T가 == 지원하는지 모름
//	}
//	return nil
//}

// T를 Equatable로 제약하면 == 사용 가능
func findIndex<T: Equatble>(of target: T, in items: [T]) -> Int? {
	for (i, item) in items.enumerated() { 
		if item == target { return i } // 👍 Equatable이니까 == 가능 
	} return nil
}

findIndex(of: 3, in: [1, 2, 3, 4]) // Optional(2) 
findIndex(of: "b", in: ["a", "b", "c"]) // Optional(1)
```

#### 1.4 where 절

```swift
// 더 복잡한 제약을 걸여야한다면?
func hasSameElements<T: Equatable>(_ a: [T], _ b: [T]) -> Bool {
	guard a.count == b.count else { return false }
	for (x, y) in zip(a, b) {
		if x != y { return false }
	}
	return true
}

// where로 여러 제약을 동시에
func procuessCollection<C: Collection>(_ items: C) where C.Element: Comparable {
	let sorted = items.sorted()
	print(sorted)
}

procuessCollection([3, 1, 4, 1, 5])
procuessCollection(["c", "a", "b"])
```

#### 1.5 제네릭 vs any 의 성능 차이

컴파일러가 제네릭 함수를 호출하는 각 구체 타입에 대해 **별도의 코드를 생성**한다. 
`process<T>(T)` → `process(Int)`, `process(String)` 등 타입별 전용 함수가 만들어지는데, 이러면 타입이 컴파일 타임에 확정된다.

```swift
func printAll<T: CustomStringConvertible>(_ items: [T]) {
	for item in items {
		print(item) // 컴파일 타임에 타입 확정
	}
}

func printAll(_ items: [any CustomStringConvertible]) {
	for item in items { 
		print(item) // 런타임에 타입 결정
	}
}
```

|          | 제네릭 `<T>`             | `any`                  |
| -------- | --------------------- | ---------------------- |
| 타입 결정    | 컴파일 타임                | 런타임                    |
| 여러 타입 섞기 | 불가 (`[T]`는 한 타입)      | 가능 `[any P]`는 다른 타입 혼합 |
| 용도       | 같은 타입의 컬렉션, 성능이 중요할 때 | 여러 타입 저장, 런타임 다형성      |

---
## ❓ 스스로에게 물어봐

Q1. 제네릭이 왜 필요하고 any와의 차이는?

>제네릭은 타입 안전성을 유지하면서 코드를 재사용하기 위해 필요합니다. 같은 로직을 Int, String용으로 따로 쓰지 않고 `<T>` 로 한 번에 쓰면 됩니다.
>`any`와의 차이는 성능과 타입 결정 시점입니다. 제네릭은 컴파일 타임에 타입이 확정되어 빠르고, `any`는 런타임에 타입이 결정되어 느립니다.
>여러 다른 타입을 하나의 배열에 담으려면 `any`를 써야 하고, 같은 타입의 컬렉션이면 제네릭이 성능상 유리합니다.