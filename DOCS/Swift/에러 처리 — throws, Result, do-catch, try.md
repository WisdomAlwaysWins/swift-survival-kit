#swift #error-handling

>[!info] 에러 처리, 왜 알아야하는가?
>네트워크 실패, JSON 파싱 실패, 파일 없음 — 앱에서 에러는 반드시 발생한다. Swift는 에러를 "무시"하지 못하게 설계되어 있어서, 에러 처리 문법을 모르면 코드를 쓸 수 없다. Result 타입은 비동기 콜백에서 성공/실패를 깔끔하게 전달하는 패턴으로, 자주 쓰인다.

## 1. 개념 정리
#### 1.1 Error 프로토콜 + throws

```swift
// 1. 에러 타입 정의
enum NetworkError: Error {
	case noConnection
	case timeout
	case invalidResponse(statusCode: Int)
}

// 2. 에러를 던지는 함수
func fetchData(from url: String) throw -> Data {
	guard url.starts(with: "https") else {
		throw NetworkError.noConnection
	}
	return Data() // 실제로는 네트워크 요청
}
```

#### 1.2 do-catch

```swift
do {
	let data = try fetchData(from: "https://api.example.com")
	print("성공: \(data)")
} catch MetworkError.noConnection {
	print("연결 없음")
} catch NetworkError.timeout {
	print("시간 초과")
} catch NetworkError.invalidResponse(let code) {
	print("잘못된 응답: \(code)")
} catch {
	print("기타 에러: \(error)")  // error는 자동 바인딩
}
```

### 1.3 try, try?, try!

```swift
// try — do-catch 안에서 사용
do {
	let data = try fetchData(from: "...")
} catch { print(error) }

// try? — 실패하면 nil, 성공하면 Optional 값
let optionalData = try? fetchData(from: "...")

// try! — 실패하면 크래시
let data = try! fetchData(from: "...")
```

|       | `try`       | `try?`  | `try!` |
| ----- | ----------- | ------- | ------ |
| 에러 처리 | do-catch 필수 | nil로 변환 | 크래시    |
| 반환 타입 | `T`         | `T?`    | `T`    |
| 안전성   | 높음          | 중간      | 낮음     |

#### 1.4 Result 타입

`enum Result<Success, Failure: Error> { case success(Success); case failure(Failure) }` — 성공 또는 실패를 하나의 값으로 표현하는 [[Swift-열거형-AssociatedValue-RawValue-패턴매칭|제네릭 열거형]]이다. 비동기 콜백에서 에러를 깔끔하게 전달하는 데 많이 쓴다.

```swift
func loadUser(id: Int, completion: (Result<String, NetworkError>) -> Void) {
	if id > 0 {
		completion(.success("User_\(id)"))
	} else {
		completion(.failure(.invalidResponse(statusCode: 400)))
	}
}

loadUser(id: 42) { result in
	switch result {
	case .success(let name):
		print("유저: \(name)")
	case .failure(let error):
		print("실패: \(error)")
	}
}

// get()으로 throws 변환도 가능
let result: Result<String, NetworkError> = .success("data")
let value = try? result.get() // Optional("data")
```

---
## ❓ 스스로에게 물어봐

Q1. `try?`의 반환 타입은 non-optional이다?

>Nope. 실패시 nil을 반환하므로, 옵셔널이다.

Q2. 다음 코드의 출력은?

```swift
func risky() throws -> String { return "ok" }
let result = try? risky()
print(type(of: result))
```

>`Optional<String>`  - `try?`는 항상 옵셔널로 감싼다.
