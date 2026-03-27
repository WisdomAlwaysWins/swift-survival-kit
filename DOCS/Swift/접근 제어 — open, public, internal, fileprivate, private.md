#swift #access-control

>[!info] 접근 제어, 왜 알아야 하는가?
>접근 제어는 "누가 이 코드에 접근할 수 있는가"를 결정한다. `private init()`으로 Singleton을 만들고 `private(set)`으로 외부에서 읽기만 가능하게 하게, 모듈 경계에서 `public`과 `open`을 구분하는 것이 중요하다!

## 1. 5가지 접근 수준 (넓은 순서)

| 수준            | 범위                           |
| ------------- | ---------------------------- |
| open          | 모듈 외부에서 상속 / 오버라이딩 가능        |
| public        | 모듈 외부에서 접근 가능, 상속 / 오버라이딩 불가 |
| internal (기본) | 같은 모듈 내에서만                   |
| fileprivate   | 같은 파일 내에서만                   |
| private       | 같은 선언(스코프)                   |

```swift
// private(set) - 외부에서는 읽기만, 쓰기는 내부에서만
struct BankAccount {
	private(set) var balance: Int = 0 // 외부에서 balance를 읽을 수 있지만 직접 수정 불가
	
	mutating func deposit(_ amount: Int) {
		balance += amoun // 내부에서만 변경
	}
}

var account = BankAccount()
print(account.balance) // 읽기 가능
// account.balance = 1000 // 🚩 쓰기 불가
account.deposit(5000) // 메서드를 통해서만 변경
```

>[!abstact] 모듈이란?
>**import해서 사용할 수 있는 하나의 배포 단위**다. 내 앱 프로젝트 전체가 하나의 모듈이고, `import UIKit`으로 가져오는 UIKit이 또 하나의 모듈이다. 접근 제어에서 "모듈 경계"란 이 import 기준을 말한다.
>```
>모듈(Module) = 하나의 배포 단위 
>├─ 내 앱 프로젝트 코드 전체 = 1개의 모듈 
>├─ UIKit = 1개의 모듈 
>├─ Foundation = 1개의 모듈 
>├─ SwiftUI = 1개의 모듈 
>├─ SnapKit (SPM/CocoaPods) = 1개의 모듈 
>└─ Alamofire = 1개의 모듈
>```

> [!tip] 내 앱만 만들 때는 internal(기본)이면 충분하다
> 접근 제어가 진짜 중요해지는 건 **라이브러리/프레임워크를 만들어서 배포할 때**다. 어떤 API를 외부에 공개할지, 상속을 허용할지를 결정해야 하니까. 앱 내부에서는 `private`과 `fileprivate`로 불필요한 접근을 막는 게 주 용도.

---
## 2. 기본 원칙

#### 2.1 타입의 내부 멤버는 타입 자체의 접근 수준을 넘을 수 없다.

```swift
internal class SomeClass {
	open var name = "이름" // 🚩 의미 없음
	public var age = 25 // 🚩 의미 없음
	var score = 100 // 기본값
	fileprivate var hidden = 0
	private var secret = ""
}
// 클래스가 internal이면 멤버가 아무리 open/public이어도 internal까지만 접근 가능
```

#### 2.2 내부 멤버의 기본값은 internal이다 (타입이 더 넓어도)

```swift
open class OpenClass {
	var name = "이름" // internal → 타입이 open이라고 멤버가 자동으로 open이 되진 않는다.
}
```

#### 2.3 변수의 타입은 변수보다 접근 수준이 같거나 높아야한다.

```swift
private struct SecretData { var value: Int }

// var data: SecretData = SecretData(value: 42) // 🚩 컴파일 에러
private var data: SecretData = SecretData(value: 42) // ✅ 변수도 private으로 맞춰야
```

---
## ❓ 스스로에게 물어봐

Q1. Swift의 기본 접근 수준은 public이다.

>X → internal로 같은 모듈 내에서만 접근 가능

Q2. open과 public의 차이는?

>open은 모듈 외부에서 상속 / 오버라이드 가능. public은 모듈 외부에서 접근은 가능하지만 상속 / 오버라이드 블가능

