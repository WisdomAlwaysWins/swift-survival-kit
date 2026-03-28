#swift #string #unicode

>[!info] String, 왜 알아야하나?
>"String은 왜 Int로 인덱싱이 안될까?"가 일단 궁금하고, Substring의 메모리 공유도 ..

## 1. String.Index — 왜 Int로 인덱싱 안되나?

Character가 가변 크기이기 때문이다! 이모지는 4바이트, 일반 문자는 1바이트라서 `text[3]` 이 몇 번째 바이트인지 확정할 수 없다. 그래서 `String.Index` 라는 전용 타입으로 안전하게 접근한다.

```swift
let text = "Hello👋"

// 🚩 Int 인덱싱 불가
// let first = text[0]

let first = text[text.startIndex] // H
let last = text[text.index(before: text.endIndex)] // 👋
let third = text[text.index(text.startIndex, offsetBy: 2)] // l

// 범위 추출
let range = text.startIndex..<text.index(text.startIndex, offsetBy: 5)
print(text[range]) // Hello
```

```swift
// 이모지가 왜 문제인지
let family = "👨‍👩‍👧‍👦
print(family.count) // 1 (Swift는 하나의 문자로 인식)
print(faily.unicodeScalars.count) // 7 (내부적으로 7개의 코드 포인트 조합)
// → Int 인덱싱은 "7개 중 몇 번째?"와 "1개 중 몇 번째?"가 달라서 혼란
```

#### 1.1 firstIndex로 안전하게 검색

```swift
let text = "swift Programming"

if let index = text.firstIndex(of: "P") {
	print(text[index]) // "P"
	print(text[index...]) // "Programming"
}

// 순회
for (i, char) in text.enumerated() {
	print("\(i): \(char)") // 0: S, 1: w, 2: i, ...
}
```
---
## 2. Substring — 메모리 공유

String의 일부를 잘라낸 것인데, 원본 String의 메모리를 공유한다. 효율적이지만 원본이 계속 메모리에 살아있으므로, 장기 보관하려면 `String()` 으로 변환해야 한다.

```swift
let full = "Hello, World!"
let sub = full.prefix(5) // type: Substring (원본 메모리 공유)
print(type.(of: sub)) // Substring

// 장기 보관하려면 String으로 변환
let permanent = String(sub) // 별도 메모리에 복사
```

```
full:    "Hello, World!"
        ↑    ↑
sub:     [Hello]  ← 원본의 일부를 가리킴 (메모리 공유)

String(sub): "Hello"  ← 별도 메모리에 복사 (원본과 독립)
```

>[!warning] split의 결과도 `[Substring]`
>`"a, b, c".split(separator: ",")` 의 결과는 `[Substring]`이다. 장기 보관하려면 `.map { String($0) }`으로 변환해야 한다.

---
## 3. 자주 쓰는 String 메서드

```swift
let text = "Hello, World!"

// 포함 확인
text.contains("World") // true

// 치환
text.replacingOccurrences(of: "World", with: "Swift") // "Hello, Swift!"

// 공백 제거
let padded = "    Hello     "
padded.trimmingCharacters(in: .whitespaces) // "Hello"

// 분할 (split → [Substring])
let csv = "apple.bannaa,cherry"
let fruits = csv.split(separator: ",") // ["apple", "banana", "cherry"]

// 합치기
["red", "green", "blue"].joined(separator: "-") // "red-green-blue"

// 앞/뒤 확인
text.hasPrefix("Hello") // true
text.hasSuffix("!") // true

// 앞/뒤 추출
text.prefix(5) // Hello
text.suffix(6) // "World!"
```
---
## 4. URL 인코딩

```swift
let query = "swift programming & more"

if let encoded = query.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) {
	let url = "https://api.example.com/search?q=" + encoded
	// https://api.example.com/search?q=swift%20programming%20&%20more
}
```
---
## 5. 멀티라인 문자열

```swift
let json = """
{
	"name": "J",
	"age": 25,
	"skills": ["Swift", "iOS"]
}
"""
// 따옴표 3개로 감싸면 줄바꿈 그대로 유지
```
---
## 6. Unicode와 Character

```swift
// 같은 문자도 다른 방식으로 표현 가능
let char1 = "é"  // 단일 코드 포인트
let char2 = "é"  // e + combining acute accent (2개)

print(char1 == char2)  // true (Swift가 정규화해서 비교)
print(char1.count)     // 1
```
---
## ❓ 스스로에게 물어봐

Q1. Swift의 String이 왜 Int로 인덱싱할 수 없나요?

>스위프트의 Character는 유니코드 기반으로 가변 크기입니다. 일반 문자는 1바이트지만 이모지는 여러 바이트를 차지하고, 여러 코드 포인트가 합쳐진 경우도 있다. `text[3]`이 바이트 수준에서 어느 위치인지 바로 알 수 없어서, String.index라는 전용 타입으로 안전하게 접근한다.

Q2. Substring이 뭔가요?

>String의 일부를 잘라낸 타입인데, 원본 String의 메모리를 공유한다. prefix나 split의 결과가 substring이다. 호율적이지만 원본이 메모리에 계속 남아있으므로 장기보관이 필요하다면 String()으로 변환해서 별도 메모리에 복사해야 한다.

Q3. `"👨‍👩‍👧‍👦".count`의 결과는 1이다.

>O → 스위프트는 여러 코드 포인트가 조합된 이모지도 1개의 Character로 인식한다.

Q4. `split(separator:)`의 결과 타입은 `[String]`이다.

>X → `[Substring]` 타입이다.