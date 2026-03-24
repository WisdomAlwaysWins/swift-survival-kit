#swift #arc #memory 

>[!info] ARC, 왜 알아야하는가?
>Heap에 생성된 class 인스턴스와 closure는 누군가가 해제해줘야 한다. Swift에서 그 역할을 하는 게 ARC(Automatic Reference Counting)다. ARC가 어떻게 동작하는지 모르면, 왜 메모리 누수가 생기는지, `[weak self]`를 왜 써야 하는지, delegate를 왜 weak으로 선언하는지 이해할 수 없다

## 1. 개념 정리

#### 1.1 ARC 동작 원리

**ARC**는 **Automatic Reference Couting**으로, 컴파일러가 컴파일 타임에 `retain`(RC+1)과 `release`(RC-1) 코드를 자동으로 삽입해서, Heap 객체의 참조 카운트를 관리하는 메모리 관리 방식이다. RC가 0이 되면 즉시 `deinit`이 호출되고 메모리가 해제된다.

```swift
class Playlist {
	let name: String
	init(name: String) {
		self.name = name
		print("\(name) 생성")
	}
	deinit { print("\(name) 해제") }
}

do {
	let jazz = Playlist(name: "Jazz") // RC = 1
	let copy = jazz // RC = 2
	print(copy.name)
}
// copy release → RC = 1
// jazz release → RC = 0 → deinit → "Jazz 해제"
```

| 특성    | ARC (swift)    |
| ----- | -------------- |
| 실행 시점 | 컴파일 타임에 코드에 삽입 |
| 해제 시점 | RC = 0 되는 즉시   |
| 일시정지  | 없음             |
| 순환 참조 | 직접 해결해야 함      |

#### 1.2 참조 카운트 저장 위치

![[ARC-참조카운트.png]]

#### 1.3 Strong / Weak / Unowned

```swift
class Band {
	let name: String
	init(name: String) { self.name = name }
	deinit { print("\(name) 해제") }
}

// strong (기본) — RC 증가
var ref1: Band? = Band(name: "Beatles") // RC = 1
var ref2 = ref1 // RC = 2
ref1 = nil // RC = 1
ref2 = nil // RC = 0 → "Beatles" 해산

// weak — RC 증가 안 함, 해제 시 자동 nil
var strongRef: Band? = Band(name: "Queen") // RC = 1
weak var weakRef = strongRef // RC = 1 (증가 안함)
strongRef = nil // RC = 0 → "Queen 해산"
print(weakRef) // nil (자동으로)

// unowned — RC 증가 안함, 해제 후 접근하면 크래시
class Fan {
	unowned let favBand: Band
	init(favBand: Band) { self.favBand = favBand }
}
```

|         | strong | weak          | unowned            |
| ------- | ------ | ------------- | ------------------ |
| RC 증가   | O      | X             | X                  |
| 타입      | `T`    | `T?`          | `T` (Non-Optional) |
| 대상 해제 후 | 해제 안됨  | 자동 nil        | 크래시                |
| 사용 시점   | 소유 관계  | 상호 참조, 수명 불확실 | 부모-자식, 수명 확실       |

#### 1.5 순환 참조의 5가지 패턴

>[!abstract] 순환 참조란?
>두 객체가 서로를 strong으로 가리키면, 둘 다 RC가 0이 될 수 없어서 영원히 메모리에 남는다. ARC의 최대 약점.

##### 패턴1. 클래스 간 상호 참조

```swift
class Artist {
	var manager: Manager?
	deinit { print("Artist deinit") }
}

class Manager {
	var artist: Artist? // 🚩 순환 참조
	deinit { print("Manager deinit") }
}

var a: Artist? = Artist()
var m: Manager? = Manager()
a?.manager = m // RC(m) = 2
m?.artist = a // RC(a) = 2
a = nil // RC(a) = 1
m = nil // RC(m) = 1

// 👍 이렇게 해야 순환 참조가 해걸이 되겠죠? 
class ManagerFixed {
	weak var artist: Artist?
}
```

##### 패턴2. 클로저가 self 캡처

```swift
class MusicPlayer {
	var trackName = "Song"
	var onPlay: (() -> Void)?
	
	func setup() {
		onPlay = {
			print(self.trackName)
		}
		// self → onPlay → closure
	}
	deinit { print("Player deinit") }
}

// // 👍 이렇게 해야 순환 참조가 해걸이 되겠죠? 
func setup() {
	onPlay = { [weak self] in
		guard let self else { return }
		print(self.trackName)
	}
}
```

##### 패턴3. Delegate에서 weak 빠뜨림

```swift
protocol PlayerDelegate: AnyObject {
	func didFinishPlaying()
}

class AudioPlayer { // 일을 시키는 쪽
	weak var delegate: PlayerDelegate?
	
	func play() {
		delegate?.didFinishPlaying() // 재생이 끝나면 delegate에게 알림
	}
}

class MusicVC: UIViewController, PlayerDelegate { // 일을 받는 쪽
	let player = AudioPlayer() // VC가 player를 소유 (strong)
	
	override func viewDidLoad() {
		super.viewDidLoad()
		player.delegate = self // player의 delegate에 나(VC)를 넣음
	}
	
	didFinishPlaying() {
		print("재생 완료")
	}
/*

MusicVC ──strong──▶ AudioPlayer
  RC=1   ◀──weak─── delegate(=self) ← RC 증가 안함


*/
```

>[!question] delegate를 왜 weak으로 선언하는가?
>보통 UIKit ViewController가 delegate를 채택하고, AudioPlayer의 delegate에 self를 넣는다. 이때 VC → AudioPlayer → delegate → VC로 순환이 생긴다. weak으로 하면 delegate가 RC를 증가시키지 않아서 순환이 끊긴다.

#### 1.6 캡처 리스트

```swift
// 기본 캡처 (참조로 캡처)
var volume = 50
let louder = { volume += 10 }
volume = 80
louder()
print(volume) // 90

// 값 캡처 (캡처 리스트 사용)
var bass = 50
let boost = { [base] in print(bass) }
bass = 100
boost() // 50 (캡처 시점의 값을 복사)

// weak self 캡처
let task = { [weak self] in 
	guard let self else { return }
	self.doWork()
}
```

#### 1.7 deinit

RC가 0이 되어 객체가 메모리에서 해제되기 직전에 호출되는 메서드다. 리소스 정리에 사용한다. class에만 있고 struct에는 없다. (struct는 ARC 관리 대상이 아니니까)

---
## ❓ 스스로에게 물어봐

Q1. ARC의 동작 원리를 설명해보세요

>ARC는 컴파일 타임에 retain와 release 코드를 자동으로 삽입하여 참조 카운트를 관리합니다. 객체 생성 시 RC = 1, 강한 참조 추가 시 +1, 해제 시 -1이고, 0이 되면 deinit이 호출됩니다.


Q2. ARC는 Stack 메모리도 관리하나요?

> Nope. ARC는 Heap 메모리만 관리한다. Stack은 Stack Point가 자동 관리한다.

Q3. weak 참조는 대상 객체의 RC를 증가시키나요?

>Nope. weak와 unowned 모두 RC를 증가시키지 않는다.

Q4. 다음 코드에서 해제가 출력되는가?

```swift
class Item { 
	deinit { 
		print("해제") 
	} 
} 

var a: Item? = Item() 
var b = a 
a = nil 
b = nil
```

>출력된다. a 생성 시 RC = 1, b = a 하면 RC = 2, a = nil 이면 RC = 1, b = nil이면 RC = 0

Q5. 다음 코드에서 메모리 누수가 발생하는가?

```swift
class X {
    var y: Y?
    deinit { print("X deinit") }
}
class Y {
    var x: X?
    deinit { print("Y deinit") }
}

var x: X? = X()
var y: Y? = Y()
x?.y = y
y?.x = x
x = nil
y = nil
```

>메모리 누수가 발생한다. 서로 strong으로 가리키므로 RC가 0이 될 수 없다.
>
>RC(X) = 1 + 1 - 1
>RC(Y) = 1 + 1 - 1