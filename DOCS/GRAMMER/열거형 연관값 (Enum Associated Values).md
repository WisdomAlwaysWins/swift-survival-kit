
#### 개념 

* 열거형의 각 case에 **서로 다른 타입의 값을 함께 저장**할 수 있는 기능

#### 기본 문법

```Swift
enum Message {
	case text(String) // 문자열 메시지
	case photo(String) // 사진 URL
	case video(String, Int) // 비디오 URL, 길이(초)
}
```

#### 사용

```Swift
let message1 = Message.text("안녕하세요")
let message2 = Message.photo("image.jpg")
let message3 = Message.video("video.mp4", 180)
```

---
#### 값 추출하기

1. switch 문으로 추출

```Swift
switch message1 {
	case .text(let content):
		print("텍스트: \(content)")
	case .photo(let url):
		print("사진: \(url)")
	case .video(let url, let duration):
		print("비디오: \(url), \(duration)초")
}
```

2. `let`을 앞으로 빼기 (모든 값이 let 일 때)

```Swift
switch message1 {
	case let .text(content):
		print("텍스트: \(content)")
	case let .photo(url):
		print("사진: \(url)")
	case let .video(url, duration):
		print("비디오: \(url), \(duration)초")
}
```

3. `if case`로 특정 case만 추출

```Swift
if case let .text(content) = message1 { // 대입 연산자 라는 걸 기억해 ..
	print("텍스트 메시지: \(content)")
}
```

---

#### 실전 예제

```Swift
enum Result {
	case success(Int)
	case failure(String)
}

let result = Result.success(200)

switch result {
	case .success(let code):
		print("성공 \(code)")
	case .failure(let error):
		print("실패 \(error)")
}
```