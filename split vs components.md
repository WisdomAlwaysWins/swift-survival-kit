
날짜 : 2026.01.15 (목)

>[!question]
>문자열을 구분자로 나눌 때 split과 components의 차이는 뭔가요?

#### Description
----
문자열을 특정 구분자로 분리하는 두 가지 메서드. 

**`split`** 은 Swift 표준 라이브러리의 String 메서드로, `Substring` 배열을 반환하며 빈 문자열 제거 옵션과 최대 분할 횟수 제어 등 다양한 옵션을 제공한다. 

**`components(separatedBy:)`** 는 Foundation 프레임워크의 NSString 메서드로, `String` 배열을 반환하며 사용법이 단순하지만 옵션이 적다. 

두 메서드의 가장 큰 차이는 빈 문자열 처리인데, **`split`** 은 기본적으로 빈 문자열을 제거하지만 **`components`** 는 포함시킨다. 

#### 개념
----
##### `split()`
```Swift
func split(
	separator: Charactor, // 구분자
	maxSplits: Int = Int.max, // 최대 분할 횟수
	omittingExmptySubnsequeneces: Bool = true // 빈 문자열 제거
) → [Substring]
```

- Swift 표준 라이브러리
- ***Substring 배열 반환***
- 기본적으로 빈 문자열 제거
- 다양한 옵션 제공

##### `components(separtedBy:)`
```Swift
func components(separatedBy separator: String) -> [String]
```

* Foundation 프레임워크 필요
* ***String 배열 반환***
* 빈 문자열 포함
* 옵션 적음


|        | split            | components           |
| ------ | ---------------- | -------------------- |
| 프레임워크  | Swift 표준         | Foundation           |
| 반환 타입  | `[Substring]`    | `[String]`           |
| 빈 문자열  | 기본 제거            | 포함                   |
| 구분자 타입 | Charater, String | String, CharacterSet |
| 옵션     | 많음               | 적음                   |



