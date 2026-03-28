>[!info] 열거형, 왜 중요한가?
>열거형은 "정해진 선택지 중 하나"를 표현하는 타입이다. 스위프트의 열거형은 다른 언어와 달리 각 케이스에 데이터를 붙일 수 있고 (Associated Value) 이 덕분에 네트워크 응답, API 엔드포인트, 상태 관리 등 매우 자주 쓰인다. 그리고 Optional이 내부적으로 `enum Optional<Wrapped> { case none, case some(Wrapped) }` 이므로, 열거형을 모르면 Optional의 본질을 이해할 수 없다.

## 1. 개념 정리
#### 1.1 기본 열거형

열거형은 가능한 값들을 미리 정의해놓은 타입이다. "이 변수에는 이 값들 중 하나만 들어갈 수 있다"는 걸 컴파일 타임에 보장한다. 값 타입이므로 Stack에 저장된다.

```swift
enum Direction {
	case north
	case south
	case east
	case west
}

var heading = Direction.north
heading = .east // 타입이 추론되면 . 으로 접근 가능
```

#### 1.2 원시값 (Raw Value)

각 케이스에 미리 **정해진 고정 값** (`String`, `Int`, `Double` 등)을 부여하는 것이다. 모든 케이스가 같은 타입의 원시값을 가진다. 원시값으로부터 열거형을 생성할 수 있는데, 이때 **실패할 수 있으므로 Optional을 반환**한다.

```swift
enum Difficulty: String {
	case easy = "쉬움"
	case medium = "보통"
	case hard = "어려움"
}

let level = Difficulty.hard
print(level.rawValue) // "어려움"


// Raw Value로부터 생성 (실패 가능 → 옵셔널)
let fromRaw = Difficulty(rawValue: "보통")
let invalid = Difficulty(rawValue: "최상")

// Int Raw Value는 자동 증가
enum Priority: Int {
	case low = 0
	case medium // 자동으로 1
	case high // 자동으로 2
}

print(Priority.high.rawValue) // 2
```

#### 1.3 연관값 (Associated Value)

각 케이스에 런타임에 결정되는 데이트를 붙이는 것이다. Raw Value와 달리 케이스마다 다른 타입, 다른 개수의 데이터를 가질 수 있다. "이 케이스일 때 추가적인 정보가 뭔지"를 함께 담는 것!

```swift
enum PaymentReusult {
	case success(amount: Intk, receipt: String)
	case failure(errorCode: Int, message: String)
	case pending
}

let result = PaymentResult.success(amount: 15000, receipt: "RCP-001")
```

>[!question] Raw Value vs Associated Value
>한 열거형에서 둘을 동시에 사용할 수 없다!
>- **Raw Value**: 컴파일 타임에 정해진 고정 값으로 모든 케이스가 같은 타입을 가짐 (.rawValue)
>- **Asscicated Value**: 런타임에 결정되는 데이터로 케이스마다 다른 타입/개수 가능. Switch로 추출.
>

#### 1.4 패턴 매칭

```swift
// switch로 연관값 추출
func handlePayment(_ result: PaymentResult) {
    switch result {
    case .success(let amount, let receipt):
        print("결제 완료: \(amount)원, 영수증: \(receipt)")
    case .failure(let code, let message):
        print("결제 실패 [\(code)]: \(message)")
    case .pending:
        print("처리 중...")
    }
}

// if case로 특정 케이스만 매칭
func checkAmount(_ result: PaymentResult) {
    if case .success(let amount, _) = result {
        print("결제 금액: \(amount)")
    }
}

// guard case로 조기 종료
func processSuccess(_ result: PaymentResult) {
    guard case .success(let amount, let receipt) = result else {
        print("성공 아님")
        return 
    }
    print("처리: \(amount)원, \(receipt)")
}
```

#### 1.5 CaseIterable

열거형의 모든 케이스를 배열로 접근할 수 있게 해주는 프로토콜이다. `allCase`라는 타입 프로퍼티가 자동 생성된다. 

**연관값(Associated Value**)이 있는 케이스가 하나라도 있으면 **사용할 수 없다!**

```swift
enum Season: CaseIterable {
	case spring, summer, fall, winter
}

for season in Season.allCases {
	print(season) // spring, summer, fall, winter
}

print(Season.allCases.count) // 4
```

#### 1.5 API EndPoint 관리

1. 타입 안전: 오타 방지 (문자열 "/users" 같은 실수 없음)
2. 자동 완성: Xcode에서 .userList, .userDetail 등 바로 뜸
3. 한 곳에서 수정: 경로가 바뀌면 enum만 수정
4. Associated Value: id, query 같은 파라미터를 타입 안전하게 전달

```swift
enum ShopAPI {
	case productList(page: Int)
	case productDetail(id: String)
	case addToCart(productId: String, quantity: Int)
	case checkout
	
	var path: String {
		switch self {
		case .productList: return "/products"
		case .productDetail: return "/products/\(id)"
		case .addToCart: return "/cart"
		case .checkout: return "/checkout"
		}
	}
	
	var method: String {
		switch self {
		case .productList, .productDetail: return "GET"
		case .addToCart: return "POST"
		case .checkout: return "POST"
		}
	}
}

let ai = ShopAPI.productDetail(id: "ABC-123")
print(api.path) // "/products/ABC-123"
print(api.method) // "GET"
```
---
## 2. 메모리 동작

열거형은 값 타입이므로 일반적으로 Stack에 저장된다.

![[메모리-열거형.png]]

---
## 3. indirect enum (재귀 열거형)

indirect는 열거형의 연관값에 자기 자신 타입을 넣을 수 있게 하는 키워드다. 트리 구조 같은 재귀적 데이터를 표현할 때 사용한다. 내부적으로 해당 케이스를 Heap에 저장하여 무한 크기 문제를 해결한다.

enum은 값 타입이라 크기가 컴파일 타입에 확정되어야한다. 자기 자신을 포함하면 크기가 무한 확장되므로 불가능하다! `indirect` 를 붙이면 해당 케이스를 **Heap에 저장(포인터 참조)** 하여 크기가 확정된다.

```swift
// 이진 트리 표현
indirect enum BinaryNode {
	case leaf(Int)
	case node(left: BinaryNode, value: Int, right: BinaryNode)
}

let tree = BinaryNode.node(
	left: .heaf(1),
	value: 2,
	right: .node(
		left: .leaf(3),
		value: 4,
		right: .leaf(5)
	)
)

//        2 
//       / \ 
//      1   4
//         / \ 
//        3   5
```

```swift
// 산술 표현식
indirect enum ArithExpr {
	case number(Int)
	case add(ArithExpr, ArithExpr)
	case multiply(ArithExpr, ArithExpr)
}

// (2 + 3) * 4
let expr = ArithExpr.multiply(
	.add(.number(2), .number(3)),
	.number(4)
)

func evaluate(_ expr: ArithExpr) -> Int {
	switch expr {
	case .number(let n): return n
	case .add(let l, let r): return evaluate(l) + evaluate(r)
	case .multiply(let l, let r): return evaluate(l) * evaluate(r)
	}
}

print(evaluate(expr)) // 20
```

---
## ❓ 스스로에게 물어봐

Q1. Raw Value와 Associated Value의 차이는?

>Raw Value는 컴파일 타임에 정해진 고정 값으로, 모든 케이스가 같은 타입입니다.
>Associated Value는 런타임에 결정되는 데이터로, 케이스마다 다른 타입과 개수를 가질 수 있습니다.
>
>Raw Value는 .rawValue 프로퍼티로 접근하고, 역으로 init(rawValue:)로 생성 가능하지만 Optional을 반환합니다.
>Associated Value는 switch나 if case로 패턴 매칭을 통해 추출해야 합니다.
>
>이 둘은 한 열거형에서 동시에 사용할 수 없습니다.
