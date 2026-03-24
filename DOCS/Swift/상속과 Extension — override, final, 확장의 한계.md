#swift #inheritance #extension

>[!info] 상숙과 Extension
>상속은 class에서 코드를 재사용하는 수직적 방법이고, Extension은 기존 타입에 기능을 추가하는 수평적 방법이다. 프로토콜(POP)이 "상속의 대안"으로 등장한 이유를 이해하려면, 먼저 상속의 장점과 한계를 알아야 한다. Extension은 프로토콜의 기본 구현을 제공하는 핵심 도구이기도 하다.

## 1. 개념 정리
#### 1.1 상속

기존 클래스(부모)의 프로퍼티와 메서드를 새 클래스(자식)가 물려받는 것이다. **클래스에서만** 가능하고, struct/enum은 상속할 수 없다.

```swift
class Vehicle {
	var speed = 0
	func describe() -> String { "속도: \(speed)km/h" }
}

class ElectricCar: Vehicle {
	var battery = 100
	
	override func describe() -> String { 
		"\(super.desribe()), 배터리: \(battery)%" 
	}
}

let tesla = ElectricCar()
tesla.speed = 120
print(tesla.describe()) // "속도: 120km/h, 배터리: 100%"
```

#### 1.2 override와 final

`override`는 부모의 메서드/프로퍼티를 자식이 재정의하는 것이고, `final`은 더 이상 재정의할 수 없다고 잠그는 것이다.

```swift
class Animal {
    func sound() -> String { "..." }
    final func breathe() { print("호흡") }  // 재정의 불가
}

class Cat: Animal {
    override func sound() -> String { "야옹" }
    // override func breathe() { }  // 🚩 final이므로 불가
}

final class Singleton {  // 상속 자체 불가
    static let shared = Singleton()
    private init() {}
}

// class Child: Singleton { }  // 🚩 final class는 상속 불가
```

> [!tip] `final`은 성능에도 도움이 된다
컴파일러가 이 메서드는 오버라이드될 일이 없다고 확신하면 빠르겠죠?

#### 1.3 Extension

이미 존재하는 타입(class, struct, enum, protocol)에 **새로운 기능을 추가**하는 것이다. 원본 소스 코드 없이도 기능을 확장할 수 있다. Apple 프레임워크의 타입(String, Array 등)도 확장 가능하다.

```swift
extension Int {
    var squared: Int { self * self }
    func times(_ action: () -> Void) {
        for _ in 0..<self { action() }
    }
}

print(5.squared) // 25
3.times { print("Swift!") } // 3번 출력
```

✅ **추가할 수 있는 것**:
- 연산 프로퍼티 (computed property)
- 메서드 (instance/type)
- 새로운 이니셜라이저 (convenience만)
- 서브스크립트
- 중첩 타입
- 프로토콜 채택

❎ **추가할 수 없는 것**:
- 저장 프로퍼티 (stored property)
- deinit
- 기존 기능 override (class에서 일부 가능하지만 권장하지 않음)

##### 1.3.1 프로토콜 채택을 위한 Extension

```swift
struct Ticket {
	let movie: String
	let seat: String
}

// 별도 Extension에서 프로토콜 채택 — 코드 정리에 좋음
extension Ticket: CustomStringConvertible {
	var description: String {
		"\(moive) - \(seat)석"
	}
}

print(Ticket(movie: "인셉션", seat: "A3")) // "인셉션 - A3석"
```

##### 1.3.2 조건부 확장 (where 절)

```swift
extension Array where Element: Numeric {
	var total: Element {
		reduce(0, +)
	}
}

[1, 2, 3, 4].total // 10
[1.5, 2.5, 3.0].total // 7.-
// ["a", "b"].total // String은 Numeric이 아님
```

---
## ❓ 스스로에게 물어봐

Q1. Extension에서 저장 프로퍼티를 추가할 수 있다?

>없다. 연산 프로퍼티만 가능. 저장 프로퍼티는 인스턴스의 메모리 레이아웃을 바꾸므로 불가.

Q2. struct도 상속할 수 있다?

>없다. 상속은 클래스만 가능하고 구조체는 프로토콜로 다형성을 구현한다.
