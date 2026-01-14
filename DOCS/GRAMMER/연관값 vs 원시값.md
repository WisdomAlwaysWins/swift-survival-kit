
>[!question] 헷갈리는 포인트
>1. 연관값을 저장 후 변경 불가
>2. 연관값과 원시값은 동시에 사용 불가

>[!info] 옵셔널은 사실 연관값을 가진 열거형 !
>```Swift
>enum Optional {
>	case none
>	case some(Wrapped)
>}
>
>// 그래서 이렇게 쓸 수 있음
>let num: Int? = 5
>if case let .some(value) = num {
>	print(value) // 5
>}
>```

|     | 연관값 (Associated Values) | 원시값 (Raw Values) |
| --- | ----------------------- | ---------------- |
| 선언  | `case text(String)`     | `case one = 1`   |
| 타입  | case 마다 다른 타입 가능        | 모든 case 가 같은 타입  |
| 값   | 인스턴스 생성 시 저장            | 선언 시 고정          |
| 변경  | 가능 (새 인스턴스 생성)          | 불가능              |
| 추출  | `switch`, `if case`     | `.rawValue`      |

```Swift
// 연관값 - 인스턴스마다 다른 값 저장
enum Message {
	case text(String)
}

let msg1 = Message.text("안녕")
let msg2 = Message.text("반갑다")

msg1 = .text("헬로")

// 원시값 - 항상 같은 값
enum Level: Int {
	case easy = 1
	case hard = 2
}

let level = Level.easy

```

