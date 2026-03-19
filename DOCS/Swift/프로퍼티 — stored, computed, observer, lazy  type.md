#swift #property #memory 

>[!info] 프로퍼티, 왜 중요할까?
>프로퍼티는 클래스와 구조체가 데이터를 저장하고 계산하는 방식 그 자체다. "이 데이트가 메모리에 실제로 저장되는건지, 매번 계산되는건지", "값이 바뀔 때 뭔가 자동으로 실행되게 하려면 어떻게 하는지", "무거운 데이터를 나중에 초기화하려면 어떻게 하는지"와 같은 질문의 답이 전부 프로퍼티와 관련이 있다.
>또한 `let` struct 인스턴스는 프로퍼티를 못 바꾸는데 `let` class 인스턴스는 바꿀 수 있다. 이 차이를 설명하려면 프로퍼티가 메모리에 어떻게 존재하는지를 알아야한다.

## 1. 개념 정리
#### 1.1 저장 프로퍼티 (Stored Property)

인스턴스의 메모리 공간에 실제로 값을 저장하는 프로퍼티다. 

`var`이면 변경 가능, `let`이면 변경 불가. 
클래스와 구조체에서 사용할 수 있고, 열거형에서는 사용할 수 없다.

```swift
struct Bird {
	var name: String // 저장 프로퍼티 (var → 변경 가능)
	var weight: Double // 저장 프로퍼티 (let → 변경 불가능)
}

var toucan = Bird(name: "투카니", weight: 1.2)
toucan.name = "차카니" // var 프로퍼티니까 변경 가능
//toucan.weight = 1.0 // let 프로퍼티니까 변경 불가
```

저장 프로퍼티의 메모리 구조:

1. 구조체인 경우 (값 타입)
	![[스택 - 구조체.png]]
2. 클래스인 경우 (참조 타입)
	![[스택 - 클래스.png]]

>[!question] 왜 `let`인데 클래스는 프로퍼티를 바꿀 수 있는가?
>- **struct의 `let`**: "이 값 자체가 변경 불가" → 프로퍼티를 바꾸면 값 전체가 바뀌는 것이므로 불가 
>- **class의 `let`**: "이 참조(포인터)가 변경 불가" → 포인터가 가리키는 주소는 고정이지만, 그 주소에 있는 **내용물**은 바꿀 수 있음
>
>비유하면, 
>- struct의 `let`은 "이 종이에 적힌 내용을 수정하지 마"
>- class의 `let`은 "이 주소로 가는 길을 바꾸지 마 (집 안 인테리어는 마음대로 해)"

---
#### 1.2 연산 프로퍼티 (Computed Property)

```swift
struct Rectangle {
	// 저장 프로퍼티 → 메모리에 값 저장
	var weight: Double
	var height: Double
	
	// 연산 프로퍼티 → 메모리에 값 저장하지 않고 접근할 때마다 계산
	var area: Double { 
		return weight * height
	}
}

let rectangle = Rectangle(width: 5, height: 10)
print(rectangle.area) // 이 순간에 값이 계산됨
print(rectangle.area) // 또 계산
```

값을 저장하지 않고, **매번 계산해서 값을 만들어내는** 프로퍼티다. 겉보기에는 프로퍼티처럼 `.area`로 접근하지만, 내부적으로는 **함수(getter/setter)가 실행**되는 것이다. 메모리에 값을 저장하지 않으므로 인스턴스의 메모리 크기에 영향을 주지 않는다.

##### 1.2.1 getter와 setter

```swift
struct Weight {
	var kg: Double // 저장 프로퍼티
	var pound: Double { // 연산 프로퍼티
		get {
			return kg * 2.205 // 읽을 때, kg → pound 변환
		}
		set {
			kg = newValue / 2.205 // 쓸 때, pound → kg 변환
		}
	}
}

var myWeight = Weight(kg: 70)
print(myWeight.pound) // 154.35 - getter 호출
myWeight.pound = 176 // setter 호출 (newValue = 176)
print(myWeight.weight) // 79.8... - kg가 변경됨
```

##### 1.2.2 읽기 전용 연산 프로퍼티

```swift
struct Bottle {
	var liters: Double
	
	var gallons: Double { // get만 있으면 읽기 전용 → get 키워드 생략 가능
		liters * 0.264
	}
}

let bottle = Bottle(liters: 40)
print(bottle.gallons) // 10.56
//bottle.gallons = 20 // 🚩 읽기 전용이므로 쓰기 불가
```

##### 1.2.3 언제 stored vs computed를 써야하나?

- **다른 저장 프로퍼티로부터 계산할 수 있는 값**이면 → 연산 프로퍼티 
- **독립적으로 존재해야 하는 값**이면 → 저장 프로퍼티

```swift
struct Product {
	var brand: String
	var model: String
	var displayName: String // brand가 바뀌면 이것도 수동으로 바꿔야함
}

struct Product {
	var brand: String
	var model: String
	var displayName: String { // 항상 최신 값으로 자동으로 나옴
		"\(brand) \(model)"
	}
}
```

---
#### 1.3 프로퍼티 옵저버 (Property Observer)

저장 프로퍼티의 값이 바뀌기 직전과 바뀐 직후에 자동으로 실행되는 코드 블록이다. "값이 바뀔 때 뭔가 하고 싶다"면 옵저버를 쓴다!

`willSet` (바뀌기 전)과 `didSet` (바뀐 후) 두 가지가 있다.

```swift
class BatteryMonitor { 
	var level: Int = 100 { 
		willSet { 
			print("배터리: \(level)% → \(newValue)%로 변경 예정") 
		} 
		didSet { 
			if level < 20 { 
				print("⚠️ 배터리 부족!") 
			} 
			print("배터리: \(oldValue)% → \(level)%로 변경 완료") 
		} 
	} 
}

let phone = BatteryMoniter()
phone.level = 15
// 출력:
// 배터리: 100% → 15%로 변경 예정
// ⚠️ 배터리 부족!
// 배터리: 100% → 15%로 변경 완료
```

>[!warning] 초기화 시에는 호출되지 않는다!
>프로퍼티 옵저버는 초기화 과정에서는 호출되지 않고, 초기화 이후 값이 변경될 때만 호출된다.
>```swift
>class Score {
>	var point: Int = 0 {
>		didSet { print("점수 변경!") }
>	}
>}
>
>let s = Score() // 아무것도 출력 안됨
>s.point = 50 // "점수 변경!" 출력
>```

---
#### 1.4 지연 저장 프로퍼티 (Lazy Stored Property)

처음 접근할 때 비로소 초기화되는 저장 프로퍼티다. 인스턴스를 만들 때가 아니라, 실제로 그 프로퍼티를 사용하는 순간에 메모리에 만들어진다. "안 쓸 수도 있는 무거운 것"을 미리 만들어두는 낭비를 막기 위해 사용한다.

```swift 
class PhotoEditor {
	let fileName: String
	lazy var filterEngine = FilterEngine()
	
	init(fileName: String) {
		self.fileName = fileName
		print("PhotoEditor 생성됨")
	}
}
let editor = PhotoEditor(fileName: "stopsun.png") 
// 출력: "PhotoEditor 생성됨" → 이 시점에 filterEngine은 아직 생성 안됨

let engine = editor.filterEngine()
// 이 시점에서 FilterEngine()이 처음으로 생성됨!
```

##### 1.4.1 lazy var의 규칙

1. 반드시 `var` 로 선언
	- `let` 은 인스턴스 생성 시 값이 확정되어야 하므로 지연이 불가
2. 반드시 기본값 (초기화 표현식)이 필요
	- 생성자에서 초기화하지 않으므로 기본값이 있어야함
3. 스레드 안전하지 않음
	- 여러 스데드가 동시에 접근하면 여러 번 초기화될 수 있음

##### 1.4.2 lazy var에 클로저를 사용하는 패턴

```swift
class DashboardView {
	var columnCount: Int = 3
	
	lazy var gridWidth: Int = {
		return columnCount * 120
	}() // 클로저를 즉시 실행해서 값을 넣음
}
```

---
#### 1.5 타입 프로퍼티 (Type Property)

인스턴스가 아닌 타입 자체에 속하는 프로퍼티로 모든 인스턴스가 공유하는 하나의 값이다. 
`static` 키워드를 붙여서 선언하고 메모리의 데이터 영역에 저장된다.

```swift
struct AppConfig {
	var theme: String // 인스턴스 프로퍼티 - 각 인스턴스마다 다른 값
	static var appVersion = "2.1.0" // 타입 프로퍼티 - 모든 인스턴스가 공유
	static var buildName = 347 // 타입 프로퍼티
}

let config = AppConfig(theme: "dark")
```

##### 1.5.1 Singleton 패턴

앱 전체에서 딱 하나의 인스턴스만 존재하도록 보장하는 디자인 패턴이다. `static let shared`로 유일한 인스턴스를 만들고, `private init()`으로 외부에서 새 인스턴스를 만들지 못하게 막는다.

```swift
class NetworkManager {
	static let shared = NetworkManager()
	private init() {}
	
	var baseURL = ""
	
	func fetch(endpoint: String) {
		print("GET \(baseURL)/\(endpoint)")
	}
}

Networkmanager.shared.fetch(endpoint: "home")
```

##### 1.5.2 static vs class 타입 프로퍼티

오버라이딩을 위해서는 `class` 를 붙일 것!

```swift
class Vehicle {
	static var wheels = 4 // 오버라이드 불가 
	class var description: String { // 오버라이드 가능 
		return "탈것" 
	}
}

class Motorcycle: Vehicle {
	// override static var wheels = 2 // 🚩불가
	override class var description: String { // 👍 가능
		return "오토바이"
	}
}
```

>[!warning] `class var`는 연산 프로퍼티만 가능하다!
>`class var x = 10`처럼 저장 프로퍼티로는 선언할 수 없다. 오버라이드 가능한 타입 프로퍼티는 반드시 연산 프로퍼티여야한다.

---
## 2. 프로퍼티 5종 비교

| 종류             | 메모리 저장        | 선언                              | 핵심                    |
| -------------- | ------------- | ------------------------------- | --------------------- |
| 저장 (Stored)    | ✅ 인스턴스에 저장    | `var name: String`              | 실제 값을 보관              |
| 연산 (Computed)  | 저장 안함         | `var area: Double { ... }`      | 매번 계산 (getter/setter) |
| 옵저버 (Observer) | ✅ 저장 프로퍼티에 붙임 | `var x: Int { willSet/didSet }` | 값 변경 감지               |
| 지연 (Lazy)      | ✅ 접근 시 저장     | `lazy var engine = Engine()`    | 처음 접근할 때 초기화          |
| 타입 (Type)      | ❕Data 영역에 저장  | `static var version = "1.0"`    | 모든 인스턴스가 공유           |

---
## 3. 메모리 동작

연산 프로퍼티는 Heap에 공간을 차지하지 않고 코드 영역에만 함수로 존재한다.

![[메모리동작_연산프로퍼티.png]]

---
## ❓ 스스로에게 물어봐

Q1. stored property와 computed property의 차이는?

>저장 프로퍼티는 값을 인스턴스 메모리에 실제로 저장합니다. 계산 프로퍼티는 값을 저장하지 않고 호출할 때마다 매번 getter 함수가 실행되어 계산됩니다.
>사용 기준은, 다른 저장 프로퍼티로부터 계산할 수 있는 값이면 계산 프로퍼티를, 독립적으로 존재해야하는 값이면 저장 프로퍼티를 씁니다.

Q2. lazy var는 무엇이며 언제 쓰나요?

>lazy var는 처음 접근할 때 초기화되는 저장 프로퍼티입니다.
>무거운 리소스의 지연로딩, 다른 프로퍼티에 의존하는 초기화일 때 사용합니다.
>하지만, 반드시 var로 선언해야하며 멀티스레드 환경에서 thread-safe하지 않습니다. 그리고 반드시 기본값을 필요로 합니다.

Q3. let struct와 let class에서 프로퍼티 변경이 다르게 동작하는 이유는?

>struct는 값 타입이므로 let으로 선언하면 그 인스턴스 덩어리 자체를 변경 불가능하게 만듭니다.
>class는 참조 타입이므로 let으로 선언하면 참조(포인터)가 변경 불가능한 것이지, 포인터가 가리키는 힙의 내용물은 변경 가능합니다. 즉 let이 잠그는 것은 "어디를 가리키느냐"이지, "가리키는 곳의 내용"이 아닙니다.
