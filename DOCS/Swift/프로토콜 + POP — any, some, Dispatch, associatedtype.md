>[!info] POP, 왜 중요한가?
>스위프트는 스스로를 Protocol-Oriented Programming 언어 라고 정의한다. 클래스 상속 대신 프로토콜로 다형성이 가능하고, 다중 채택이 가능하고, 수평적 확장이 가능하다.

## 1. 개념 정리

#### 1.1 프로토콜 기본

"이런 능력을 가져야 한다"는 청사진이다. 프로퍼티나 메서드의 **이름과 타입만** 정의하고, 실제 구현은 채택하는 타입이 한다. class, struct, enum 모두 채택 가능.

```swift
protocol Playable {
	var title: String { get }
	var duration: Int { get set }
	func play()
}

struct Song: Playable {
	let title: String
	var dduration: Int
	func play() { print("재생: \(title)") }
}

struct Podcast: Playable { 
	let title: String 
	var duration: Int 
	func play() { print("팟캐스트: \(title)") } 
}
```

>[!question] 프로토콜에서 `{ get }`과 `{ get set }`의 차이
>`{ get }`은 **"최소한 읽을 수 있으면 돼"**라는 뜻이다. `let`이든 `var`이든 읽기는 되니까 둘 다 OK.
>`{ get set }`은 **"읽기와 쓰기 둘 다 돼야 해"**라는 뜻이다. `let`은 쓰기가 불가하니까 반드시 `var`로 구현해야 한다.

| 프로토콜 요구       | let 구현 | var 구현 |
| ------------- | ------ | ------ |
| `{ get }`     | O      | O      |
| `{ get set }` | X      | O      |

#### 1.2 프로토콜 Extension — 기본 구현

```swift
extension Playable {
	func play() { print("기본 재생: \(title)") }
	func stop() { print("정지") }
}

struct AudioBook: Playable {
	let title: String
	var duration: Int
	// play()를 구현하지 않으면 기본 구현 사용
}

let book = AudioBook(title: "Swift 입문", duration: 3600)
book.play() // "기본 재생: Swift 입문"
```

#### 1.3 Dispatch 규칙

>[!danger] 함정에 빠지지 마..

> [!abstract] Dispatch
> "어떤 메서드를 실행할지 결정하는 방식"이다. 같은 이름의 메서드가 여러 곳(프로토콜 Extension, 구체 타입)에 있을 때, **어느 쪽 것을 호출할지** 결정하는 규칙이다.
> - **Dynamic Dispatch**: **런타임**에 "실제 타입이 뭔지" 확인해서 결정. 느리지만 다형성 가능. 
> - **Static Dispatch**: **컴파일 타임**에 "선언된 타입이 뭔지"만 보고 결정. 빠르지만 고정.

>[!abstract] Witness Table
>프로토콜을 채택한 각 타입이 **"이 요구사항 메서드의 실제 구현은 여기 있다"** 라고 등록한 테이블이다. 런타임에 이 테이블을 참조해서 실제 구현을 찾는다.

- 프로토콜 요구사항에 있는 메서드 → Dynamic Dispatch (Witness Table 참조)
- Extensio에만 있는 메서드 → Static Dispatch (컴파일 타임에 결정)

```swift
protocol Greetable {
	func hello() // 요구사항에 있음
}

extension Greetable {
	func hello() { print("프로토콜 기본") } // hello의 기본 구현
	func bye() { print("프로토콜 bye") } // Extension에만 있고 요구사항이 아님
}

struct Korean: Greetable {
	func hello() { print("안녕하세요") } // hello() 직접 구현
	func by() { print("안녕히 가세요") } // bye() 직접 구현했지만, 
}

// ═══ 프로토콜 타입으로 호출 ═══
let person: Greetable = Korean()

person.hello() // "안녕하세요"
// Dynamic Dispatch
// → Witness Table 참조 
// → 실제 타입이 Korean이네!

person.bye() // "프로토콜 bye" 
// Static Dispatch 
// → 컴파일 타임에 person은 Greetable이네 
// → Greetable Extension의 bye() 호출!
// → Korean에 bye()가 있어도 무시됨!

// ═══ 구체 타입으로 호출하면 다름 ═══
let korean: Korean = Korean()
korean.hello() // "안녕히 가세요" ← 구체 타입이니 Korean의 구현 호출
```

같은 객체인데 타입 선언에 따라 결과가 다르다!

```swift
let sameObject = Korean()

let asProtocol: Greetable = sameObject
let asConcrete: Korean = sameObject

asProtocol.bye()  // "프로토콜 bye"     ← Static Dispatch (선언 타입 = Greetable)
asConcrete.bye()  // "안녕히 가세요"    ← 구체 타입이니 Korean 것

// 같은 Korean 인스턴스인데 결과가 다름! 이게 함정이다.
```

>[!tip] 함정을 피하는 방법 
>Extension에만 있는 메서드를 구체 타입에서 "오버라이드"하지 마라. 하고 싶으면 **프로토콜 요구사항에 추가**해라. 그러면 Witness Table에 등록되어 Dynamic Dispatch가 적용된다.

```swift
// 👍 bye()도 요구사항에 넣으면 Dynamic Dispatch
protocol Greetable {
    func hello()
    func bye()     // 요구사항에 추가!
}

extension Greetable {
    func bye() { print("프로토콜 bye") }  // 기본 구현
}

struct Korean: Greetable {
    func hello() { print("안녕하세요") }
    func bye() { print("안녕히 가세요") }  // 이제 Witness Table에 등록!
}

let person: Greetable = Korean()
person.bye()  // "안녕히 가세요" → Dynamic Dispatch!
```

#### 1.4 associatedtype

프로토콜에서 [[제네릭 기초 — <T> 문법과 타입 제약]]이다. 프로토콜 안에서 "어떤 타입을 사용할지는 채택하는 쪽에서 정해라"고 자리만 만들어놓은 것이다.

```swift
protocol Container {
    associatedtype Item  // "Item이라는 타입을 채택하는 쪽에서 정해라"
    var items: [Item] { get }
    mutating func add(_ item: Item)
}

struct Bag: Container {
    typealias Item = String  // Item = String으로 결정 (typealias 생략 가능, 추론됨)
    var items: [String] = []
    mutating func add(_ item: String) { items.append(item) }
}

struct NumberBox: Container {
    var items: [Int] = []     // Item = Int로 추론됨
    mutating func add(_ item: Int) { items.append(item) }
}
```

#### 1.5 any vs some

- `any` : Opaque Type, 정확한 타입은 숨기지만 하나의 구체 타입 → 컴파일러가 실제 타입을 알고 있어 Static Dispatch 가능.
- `some` : Existential Type, 런타임에 타입이 결정되어 Dynamic Dispatch + Existential Container 사용

```swift
// some - 컴파일러가 앎
func makePlayer() -> some Playable {
	return Song(title: "Hello", duration: 200)
	// 항상 Song을 반환. 호출자는 Playable로만 알지만, 컴파일러는 Song인 걸 앎
}

// any - 아무 Playable이나 (런타임에 결정)
func playlist() -> [any Playable] {
	return [
		Song(title: "A", duration: 100),
		Song(title: "B", duration: 200)
	] // 다른 타입들과 섞을 수 있음
}
```

|                | `some P`    | `any P`                   |
| -------------- | ----------- | ------------------------- |
| 타입 결정          | 컴파일 타임      | 런타임                       |
| Dispatch       | Static (빠름) | Dynamic (느림)              |
| 여러 타입 섞기       | 불가          | 가능                        |
| associatedtype | 사용 가능       | 사용 불가 (Swift 5.7부터 일부 가능) |
| 대표 사용처         | SwiftUI     | 프로토콜 타입 배열                |

#### 1.6 프로토콜 컴포지션 (&)

```swift
protocol Named { var name: String { get } }
protocol Aged { var age: Int { get } }

func introduce(_ person: Named & Aged) {
	print("\(person.name), \(person.age)세")
}

struct Student: Named, Aged {
	let name: String
	let age: Int
}

introduce(Student(name: "J", age: 25))
```

#### 1.7 POP가 OPP 대비 갖는 장점

| OOP (상속)    | POP (프로토콜)              |
| ----------- | ----------------------- |
| class에서만 가능 | struct/enum/class 모두 가능 |
| 단일 상속       | 다중 채택 가능                |
| 수직적 (부모-자식) | 수평적 (능력 조합)             |
| 참조 타입 강제    | 값 타입 사용 가능              |
| 불필요한 상속 계층  | 필요한 능력만 선택적 채택          |

---
## ❓ 스스로에게 물어봐

Q1. 프로토콜의 Dynamic Dispatch와 Static Dispatch

>프르토콜의 요구사항에 정의된 메서드를 프로토콜 타입으로 호출하면 witness table를 참조하는 dynamic dispatch가 일어나서 실제 구체 타입의 구현이 호출된다.
>하지만 Extension에만 정의된 메서드(요구사항이 아닌 것)를 프로토콜 타입으로 호출하면 static dispatch가 일어나서 구체 타입의 구현이 아니라 extension의 기본 구현이 호출된다.

Q2. 다음 코드의 출력은?

```swift
protocol P {
    func a()
}
extension P {
    func a() { print("P.a") }
    func b() { print("P.b") }
}
struct S: P {
    func a() { print("S.a") }
    func b() { print("S.b") }
}

let x: P = S()
x.a()
x.b()
```

>S.a
>P.b
>
>`a()`는 프로토콜 요구사항 → Dynamic Dispatch → S의 구현
>`b()`는 Extension에만 있음 → Static Dispatch → P의 기본 구현

Q3. `some Playable`과 `any Playable`의 차이를 한 문장으로

>`some`은 컴파일 타임에 하나의 구체 타입이 확정되어 빠르고, 
>`any`는 런타임에 아무 타입이나 담을 수 있어 유연하지만 느리다.