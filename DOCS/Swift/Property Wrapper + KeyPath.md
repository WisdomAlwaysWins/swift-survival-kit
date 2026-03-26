## ❓ 스스로에게 물어봐

Q1. sto#swift #property-warpper #keypath

> [!info]
> SwiftUI의 `@State`, `@Binding`, `@ObservedObject`가 전부 Property Wrapper다. 이게 뭔지 모르면 SwiftUI 코드를 읽을 수 없다. KeyPath는 프로퍼티 자체를 값으로 다루는 개념으로, KVO와 SwiftUI 바인딩의 기반이 된다.

## 1. Property Wrapper

프로퍼티에 접근할 때 **중간에 로직을 끼워넣는 매커니즘**이다. getter / setter를 감싸서, 값을 읽거나 쓸 때 자동으로 추가 작업(검증, 변환, 저장 등)을 수행한다.

```swift
@propertyWrapper
struct Clamped {
	private var value: Int
	let range; ClosedRange<Int>
	
	var wrappedValue: Int {
		get { value }
		set { value = min(max(newValue, range.lowerBound), range.upperBound) }
	}
	
	init(wrappedValue: Int, range: ClosedRange<Int>) {
		self.range = range
		self.value = min(max(wrappedValue, range.lowerBound), range.upperBound)
	}
}

struct Player {
	@Clamped(range: 0...100) var hp: Int = 100
}

var player = Player()
player.hp = 150
print(player.hp) // 100 (range 초과분 잘림)
player.hp = -10
print(player.hp) // 0
```

#### 1.1 컴파일러가 하는 변환

```swift

// 우리가 쓰는 코드
@Clamped(range: 0...100) var hp: Int = 100

// 컴파일러가 실제로 만드는 코드
private var _hp = Clamped(wrappedValue: 100, range: 0...100)

var hp: Int {
    get { _hp.wrappedValue }
    set { _hp.wrappedValue = newValue }
}
```

#### 1.2 projectedValue ($접근)

```swift
@propertyWrapper
struct Validated {
    private var value: String = ""
    var wrappedValue: String {
        get { value }
        set { value = newValue }
    }

    var projectedValue: Bool {  // $ 접근 시 반환
        !value.isEmpty
    }
}

struct Form {
    @Validated var email: String = ""
}

var form = Form()
form.email = "j@test.com"
print(form.$email)  // true (projectedValue — 비어있지 않으니 유효)
```

> [!abstract] SwiftUI에서의 Property Wrapper
> `@State`의 wrappedValue는 실제 값, projectedValue는 `Binding`이다.
> 그래서 `$count`로 접근하면 `Binding<Int>`가 나와서 자식 뷰에 전달할 수 있다.

---
## 2. KeyPath

**프로퍼티 자체를 값으로** 다루는 것이다. "어떤 타입의 어떤 프로퍼티"를 `\Type.property` 형태로 표현한다. 프로퍼티에 접근하는 "경로"를 변수에 담을 수 있다.

```swift
struct Album {
	var title: String
	var trackCount: Int
}

// KeyPath 생성
let titlePath: KeyPath<Album, String> = \Album.title
let countPath: \Album.trackCound // 타입 추론 가능

// KeyPath로 접근
let album = Album(title: "Midnights", trackCound: 13)
print(album[keyPath: titlePath]) // "Midnight"
print(album[keyPath: countPath]) // 13

// 고차함수에서 활용
let albums = [
	Album(title: "A", trackCount: 10),
	Album(title: "B", trackCount: 15),
    Album(title: "C", trackCount: 8)
]

let titles = albums.map(\.title) // ["A", "B", "C"]
let sorted = albums.sorted(by: \.trackCount) // trackCount 기준 정렬 (Swift 5.9+)
```

---
## ❓ 스스로에게 물어봐

Q1. Property Wrapper의 `$` 접근은 `wrappedValue`를 반환한다. 

>X → proejctedValue를 반환한다.

Q2. `\Album.title`의 타입은?

>`KeyPath<Album, String>` Albums의 String 타입 프로퍼티에 대한 경로
