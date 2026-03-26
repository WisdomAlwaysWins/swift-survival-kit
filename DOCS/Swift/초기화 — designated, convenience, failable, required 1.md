#swift #initialization

>[!info] 초기화, 왜 알아야하는가?
>초기화는 "모든 프로퍼티가 사용 전에 반드시 값을 갖도록 보장"하는 Swift의 안전 장치다. 상속 구조에서 초기화 위임 규칙을 모르면 컴파일 에러에 막히고, `required init?(coder:)`가 왜 필요한지 설명하지 못한다.

## 1. 개념 정리
#### 1.1 Designated vs Convenience

- **Designated**: 모든 프로퍼티를 초기화하는 주 생성자로 상위 클래스의 Designated를 호출해야 함.
- **Convenience**: 보조 생성자로 같은 클래스의 다른 init을 호출해야함. `convenience` 키워드 필요.

```swift
class Beverage {
	let name: String
	let size: String
	
	// Designated - 모든 프로퍼티 직접 초기화
	init(name: String, size: String) {
		self.name = name
		self.size = size
	}
	
	// Convenience - 다른 init 호출
	convenience init(name: String) {
		self.init(name: name, size: "Regular") // 같은 클래스의 Designated 호출
	}
}
```

##### 1.1.1 초기화 위임 규칙 2가지

1. Designated init은 반드시 **상위 클래스의 Designated init**을 호출
2. Convenience init은 반드시 **같은 클래스의 다른 init**을 호출

```swift
class Latte: Beverage {
    let hasMilk: Bool
    init(name: String, size: String, hasMilk: Bool) {
        self.hasMilk = hasMilk
        super.init(name: name, size: size)  // 규칙 1: 부모의 Designated 호출
    }

    convenience init() {
        self.init(name: "라떼", size: "Tall", hasMilk: true)  // 규칙 2: 같은 클래스의 init 호출
    }
}
```

#### 1.2 Failable Initializer (init?)

```swift
struct Temperature {
    let celsius: Double

    init?(celsius: Double) { // 실패할 수 있어 .. 그럼 nil을 반환해야겠지?
        guard celsius >= -273.15 else { return nil }  // 절대영도 이하면 실패
        self.celsius = celsius
    }
}

let valid = Temperature(celsius: 25)    // Optional(Temperature)
let invalid = Temperature(celsius: -300) // nil
```

#### 1.3 required init

모든 서브 클래스가 반드시 해당 init을 구현해야한다는 표시.

```swift
class CustomView: UIView {
	let label = UILabel()
	
	override init(frame: CGFloat) {
		super.init(frame: frame)
		setup()
	}
	
	required init?(coder: NSCoder) { // Storyboard 지원 필수
		super.init(coder: coder)
		setup()
	}
	
	func setup() { addSubview(label) }
}
```

#### 1.4 Memberwise Initializer

Struct에만 자동 생성되는 생성자로 모든 저장 프로퍼티를 파라미터로 받는다. 직접 init을 정의하면 사라진다. (Extension에 넣으면 유지 가능)

```swift
// ═══ 기본: 자동으로 Memberwise Init이 생김 ═══
struct Coffee { 
	var name: String 
	var price: Int 
	var isIced: Bool 
}

// 직접 init을 안 만들어도 자동으로 생성됨 → 모든 저장 프로퍼티가 파라미터로 들어감
let latte = Coffee(name: "라떼", price: 5600, isIced: true)

// 기본값이 있으면 파라미터 생략 가능
struct Tea {
	var name: String
	var price: Int = 4000
	var isIced: Bool = false
}

let greenTea = Tea(name: "녹차")
let icedTea = Tea(name: "아이스티", price: 3500, isIced: true) // 전부 지정도 가능

// ═══ 직접 init을 정의하면 Memberwise가 사라짐! ═══
struct Juice {
	var name: String
	var price: Int
	
	init(name: String) { // 직접 정의
		self.name = name
		self.price = 5000 // 가격 고정
	}
}

let juice = Juice(name: "오렌지")

// ═══ Extension에 init을 넣으면 Memberwise가 유지됨! ═══
struct Smoothie {
	var name: String
	var price: Int
}

extension Smoothie {
	init(name: String) {
		self.name = name
		self.price = 6000
	}
}

let s1 = Smoothie(name: "망고", price: 7000) // Memberwise 유지
let s2 = Smoothie(name: "딸기") // extension init도 사용 가능
```

---
## ❓ 스스로에게 물어봐

Q1. Convenience init은 상위 클래스의 init을 직접 호출할 수 있다?

>없다. 같은 클래스의 다른 Init만 호출 가능. 상위 클래스 호출은 designated의 역할이다.

Q2. Struct의 Memberwise Initializer는 직접 init을 정의해도 유지된다?

>유지되지 않는다. 직접 init하면 사라진다. extension에 Init을 넣으면 유지 가능하다.
