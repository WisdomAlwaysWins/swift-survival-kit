#swift #memory 

>[!info] 메모리 영역, 왜 중요할까?
>"왜 class는 느리고 struct는 빠른가요?", "재귀가 왜 터지나요?" — 이 질문들의 답은 전부 메모리 구조에 있다. 프로세스가 실행될 때 메모리가 어떻게 나뉘고, 각 영역에 무엇이 저장되는지 알아야 Swift의 거의 모든 동작이 "왜 그렇게 되는지" 설명할 수 있다.

## 1. 개념 정리
#### 1.1 프로세스 메모리의 4가지 영역

>[!abstract] 프로세스 메모리
>앱이 실행되면 운영체제가 그 앱에게 독립적인 메모리 공간을 할당한다. 이 공간이 4개의 영역으로 나뉘어 서로 다른 목적으로 사용된다. iOS에서 앱 하나가 받는 메모리는 보통 수백 MB ~ 몇 GB 범위다.

```
HIGH ADDRESS (큰 주소)
┌─────────────────────────────┐
│         Stack               │  ← 함수 호출 시 생성, 반환 시 소멸
│         ↓↓↓ (아래로 성장)   │     지역 변수, 파라미터, 리턴 주소
│                             │
│      (free space)           │  ← Stack과 Heap 사이의 빈 공간
│                             │
│         ↑↑↑ (위로 성장)     │
│         Heap                │  ← 동적 할당 (class, closure 인스턴스)
│                             │     ARC가 관리
├─────────────────────────────┤
│         Data                │  ← 전역 변수, static 변수
│                             │     앱 시작 시 생성, 종료 시 소멸
├─────────────────────────────┤
│         Code (Text)         │  ← 컴파일된 기계어 명령어
│                             │     읽기 전용
└─────────────────────────────┘
LOW ADDRESS (작은 주소)
```

#### 1.2 각 영역 상세

##### 1.2.1 Code (Text) 영역

컴파일된 **기계어 명령어**(함수의 실행 코드)가 저장되는 영역이다. **읽기 전용**이라 런타임에 수정할 수 없다. 앱이 실행되면 모든 함수의 코드가 여기에 올라간다.

```swift
func greet() { print("hello") }  // greet의 기계어 → Code 영역
func calculate(_ x: Int) -> Int { return x * 2 }  // calculate의 기계어 → Code 영역
```

##### 1.2.2 Data 영역

**전역 변수**와 **static 변수**가 저장되는 영역이다. 앱이 시작될 때 초기화되고, 앱이 종료될 때 해제된다. 앱 실행 내내 메모리에 살아있으므로 어디서든 접근 가능하다.

```swift
var appLaunchCount = 0           // 전역 변수 → Data 영역

struct Settings {
    static var darkMode = false  // static 변수 → Data 영역
}
```

##### 1.2.3 Heap 영역

**동적으로 할당되는 메모리** 영역이다. `class` 인스턴스와 `closure`가 여기에 생성된다. 개발자(정확히는 ARC)가 해제를 책임져야 하고, Stack보다 할당/해제가 느리다. 아래에서 위로 성장한다.

```swift
class Order {
    var item: String
    var price: Int
    init(item: String, price: Int) {
        self.item = item
        self.price = price
    }
}

let myOrder = Order(item: "커피", price: 4500)
// Order 인스턴스 → Heap에 생성
// myOrder (포인터) → Stack에 저장
```

##### 1.2.4 Stack 영역

**함수 호출 시 생성되는 스택 프레임**이 쌓이는 영역이다. 지역 변수, 파라미터, 리턴 주소가 저장된다. 위에서 아래로 성장하고, 함수가 반환되면 프레임이 자동으로 사라진다. 매우 빠르지만 크기가 제한적이다 (iOS에서 1~2MB).

```swift
func checkout() {
    let quantity = 3          // Stack
    let unitPrice = 4500      // Stack
    let total = quantity * unitPrice  // Stack
    print(total)
}
// checkout() 반환 → quantity, unitPrice, total 전부 자동 소멸
```

>[!tip] 스택 프레임의 동작은 [[함수 — 1급 객체와 스택 프레임]] 에서 다뤘다.

#### 1.3 Swift 코드에서 각 영역에 무엇이 저장되는가?

```swift
// ── Code 영역 ──
// 아래 함수들의 컴파일된 기계어가 Code에 저장됨
func processOrder() { ... }
func calculateDiscount() { ... }

// ── Data 영역 ──
var totalOrderCount = 0              // 전역 변수
struct API {
    static let baseURL = "https://..."  // static 상수
}

// ── Stack 영역 ──
func makeReceipt() {
    let date = Date()               // Stack (struct이므로 값 직접 저장)
    let orderNumber = 1042          // Stack
    let discount = 0.1              // Stack
}

// ── Heap 영역 ──
class Customer {
    var name: String
    init(name: String) { self.name = name }
}

func registerCustomer() {
    let customer = Customer(name: "J")
    // customer (포인터) → Stack
    // Customer 인스턴스 → Heap
}
```

---
## 2. 힙 할당이 느린 이유

WWDC 2016 "Understanding Swift Performance"의 내용 참고.

| 비용             | 설명                                                        |
| -------------- | --------------------------------------------------------- |
| 빈 공간 탐색        | Heap에서 적절한 크기의 비어있는 블록을 찾아야함                              |
| 스레드 동기화 (Lock) | 여러 스레드가 동시에 Heap을 사용하므로 Lock이 필요                          |
| 초기화 비용         | Stack는 Stack Point 이동으로 끝나지만, Heap은 메모리 클리어 + 메타데이터 설정 필요 |
| 참조 카운팅         | Heap 객체는 RC를 관리해야함                                        |
| 캐시 미스          | Stack은 메모리가 연속적이라 CPU                                     |
>[!question] 스택은 왜 빠른데 .. ?
>스택 할당은 스택 포인터(SP)를 줄이는 것만으로 끝난다. 정수 하나를 대입하는 비용과 같다. 해제도 SP를 올리는 것만으로 끝. 탐색도 없고, Lock도 없고, RC도 없다.

---
## 스택 오버플로우

스택의 크기는 유한하다. 함수 호출마다 스택 프레임이 쌓이는데, 재귀가 너무 깊으면 Stack 전체를 소진해서 앱 크래시가 발생한다.

```swift
func forever() { // 🚩 무한 재귀
	forever() // 매 호출마다 스택 프레임에 추가되어 결국 터짐
}

func countdown(_ n: Int) {
	if n == 0 { return }
	countdown(n - 1)
]

countdown(1_000_000) // Stack Overflow
```

해결책:
- 재귀를 반복문 (for / while)으로 변환
- Tail Recursion 활용 (마지막 연산이 재귀 호출이면 컴파일러가 최적화 가능)

>[!abstract] 꼬리 재귀
>일반 재귀는 함수가 호출될 때마다 스택 프레임이 쌓인다. (재귀 호출 이후에도 할 일이 남아있으니까) n이 100만이면 100만 개의 스택 프레임이 쌓이는 꼴.
>```swift
>func sum(_ n: Int) -> Int {
>	if n == 0 { return 0 }
>	return n + sum(n - 1) // sum(n - 1)이 돌아오면 n + 를 해야함
>}
>```
>Tail Recursion(꼬리 재귀)은 재귀 호출이 함수의 맨 마지막 동작인 경우다. 재귀 호출 이후에 할 일이 아무 것도 없으니, 이전 프레임을 유지할 필요가 없다. 프레임 1개를 교체해가면서 재활용하는 것.
>```swift
>func sum(_ n:Int, acc: Int = 0) -> Int {
>	if n == 0 { return acc }
>	return sum(n - 1, acc: acc + n)
>}
>```

---
## ❓ 스스로에게 물어봐

Q1. 메모리의 4영역을 설명해주세요.

>Code, Data, Heap, Stack 4개의 영역으로 나뉩니다. 
>Code 영역은 컴파일된 명령어가 저장되는 읽기 전용 영역입니다. 
>Data 영역은 전역 변수와 static 변수가 저장되며 앱이 실행되는 내내 유지됩니다.
>Heap 영역은 class 인스턴스나 closure 같은 참조 타입이 동적으로 할당되는 곳이고 ARC가 관리합니다.
>Stack 영역은 함수 호출 시 생성되는 스택 프레임에 지역 변수와 파라미터가 저장되며, 함수 반환 시 자동 해제됩니다.
>
>Stack은 연속적 메모리라 매우 빠르고, Heap은 분산되어있어 빈 공간 탐색, Lock 경합, 캐시 미스 등으로 느립니다.