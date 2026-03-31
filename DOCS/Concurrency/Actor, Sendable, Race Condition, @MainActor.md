#concurreny #actor #senable #thread-safety

>[!info] 왜 중요한가?
>Actor는 Swift 6에서 데이터 경쟁을 **컴파일 타임에** 방지하는 핵심 도구다. Sendable은 "이 타입은 스레드 간에 안전하게 전달할 수 있다"는 보증이다. Race Condition과 Deadlock은 동시성의 두 가지 주요 문제이다.

## 1. 개념 정리
#### 1.1 Race Condition

여러 스레드가 **같은 데이터에 동시에 접근**하면서, 최소 하나가 **쓰기**를 할 때 결과가 예측 불가능해지는 상황이다.

```swift
// ❌ Race Condition 발생!
var balance = 1000

DispatchQueue.global().async {
    let current = balance    // 1000 읽음
    sleep(1)
    balance = current - 500  // 500으로 설정
}

DispatchQueue.global().async {
    let current = balance    // 1000 읽음 (A가 아직 안 썼으니까!)
    sleep(1)
    balance = current - 300  // 700으로 설정
}

// 실제: 500 또는 700 (누가 나중에 쓰느냐에 따라!)
```

#### 1.2 Actor

>**Actors** 
>allow only **one task to access their mutable state at a time**, which makes it safe for code in multiple tasks to interact with the same instance of an actor.

Actor는 한 번에 하나의 접근만 허용하는 참조 타입이다. class와 비슷하지만, Actor의 프로퍼티와 메서드에 접근하려면 반드시 await를 써야한다.

```swift
actor BankAccount {
    var balance: Int
    
    init(balance: Int) { self.balance = balance }
    
    func withdraw(_ amount: Int) -> Bool {
        guard balance >= amount else { return false }
        balance -= amount  // Actor 내부에서는 동기적으로 안전하게 접근
        return true
    }
    
    func deposit(_ amount: Int) {
        balance += amount
    }
}

let account = BankAccount(balance: 1000)

// Actor 외부에서 접근 시 반드시 await
Task {
    let success = await account.withdraw(500)  // await 필수!
    let current = await account.balance        // await 필수!
}
```

#### 1.3 @MainActor

>**MainActor** 
>A singleton actor whose executor is equivalent to the main dispatch queue.

**메인 스레드에서 실행되도록 보장**하는 글로벌 Actor다.

```swift
@MainActor
class ProfileViewModel: ObservableObject {
    @Published var name = ""
    @Published var isLoading = false
    
    func loadProfile() async {
        isLoading = true  // ✅ 메인 스레드 보장
        let profile = await fetchProfile()
        name = profile.name  // ✅ 메인 스레드 보장
        isLoading = false
    }
}
```

>[!tip] @MainActor는 `DispatchQueue.main.async`를 대체한다 
>GCD 시절에는 매번 `DispatchQueue.main.async { }` 안에 UI 코드를 넣었다. @MainActor를 ViewModel이나 VC에 붙이면 컴파일러가 메인 스레드 실행을 **보장**한다.

#### 1.4 isolated / nonisolated

```swift
actor DataStore {
    var items: [String] = []
    
    // 기본: isolated — Actor 보호 하에 실행
    func addItem(_ item: String) {
        items.append(item)  // await 없이 안전하게 접근
    }
    
    // nonisolated — Actor 보호 밖에서 실행 (await 불필요)
    // Actor의 mutable 프로퍼티에 접근 불가
    nonisolated func description() -> String {
        "DataStore"  // items에 접근하지 않으므로 격리 불필요
    }
    
    // nonisolated let — 불변이므로 격리 불필요
    nonisolated let id = UUID()
}
```

#### 1.5 Sendable

> A type whose values can safely be passed across concurrency domains is known as a **sendable** type.

"이 타입의 값을 **다른 스레드로 안전하게 보낼 수 있다**"는 프로토콜이다.

```swift
// ✅ 값 타입은 자동으로 Sendable (프로퍼티도 전부 Sendable이면)
struct Message: Sendable {
    let id: UUID
    let text: String
}

// ✅ Actor는 자동으로 Sendable
actor ChatRoom { ... }

// ⚠️ class는 수동으로 Sendable 준수 필요
final class Config: Sendable {  // final + 불변 프로퍼티만
    let apiKey: String
    let baseURL: String
    init(apiKey: String, baseURL: String) {
        self.apiKey = apiKey
        self.baseURL = baseURL
    }
}

// ❌ 이건 Sendable 불가 — var 프로퍼티가 있는 class
class MutableConfig {
    var apiKey: String  // var → 여러 스레드에서 동시 수정 가능 → 위험
    init(apiKey: String) { self.apiKey = apiKey }
}
```

>[!warning] Swift 6의 strict concurrency Swift 6부터는 컴파일러가 Sendable을 **강제**한다. 
>Sendable 아닌 타입을 Task 경계로 보내면 컴파일 에러가 난다. `@unchecked Sendable`은 개발자가 직접 안전성을 보장할 때만 사용한다 (Lock으로 보호된 경우 등).

---
## ❓ 스스로에게 물어봐

Q1: Actor가 뭐고 왜 필요한가요?

>[!faq]- 답
>Actor는 한 번에 하나의 접근만 허용하는 참조 타입이다. 내부적으로 serial queue처럼 동작하여 동시 접근을 직렬화한다.

Q2. Race Condition이 뭔가요?

>[!faq]- 답
>여러 스레드가 같은 데이터에 동시에 접근하면서 최소 하나가 쓰기를 할 때 발생하는 상황.
>Actor로 데이터를 격리하거나, Serial Queue로 접근을 직렬화하거나, Lock으로 보호해야한다.

Q3. @MainActor의 역할은?

>[!faq]- 답
>메인 스레드에서 실행되도록 보장하는 글로벌 Actor. GCD의 DispatchQueue.main.async를 대체한다.

