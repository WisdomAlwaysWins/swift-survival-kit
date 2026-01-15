날짜 : 2026-01-15 (수)

>[!question] 
>map과 compactMap 함수의 차이는 뭘까

#### Description
---
배열의 각 요소를 변환할 때 사용하는 두 가지 고차함수.

`map`은 모든 요소를 변환한 결과를 그대로 유지하며, 변환 결과가 nil이어도 배열에 포함시킨다.
반면 `compactMap`은 변환 후 nil인 요소를 자동으로 제거하고 옵셔널을 언래핑해준다.

예를 들어, 문자열 배열을 숫자로 변환할 때 "abc" 같이 변환이 불가능한 값은 nil이 되는데, map을 쓰면 `[Int?]` 타입의 옵셔널 배열이 되지만, compactMap을 쓰면 실패한 변환은 제거하고 `[Int]` 타입의 깔끔한 배열을 얻을 수 있다. 

#### 개념
---
##### map

```Swift
func map(_ transform: (Element) -> T) -> [T]
```

- 모든 요소에 변환 함수 적용 
- 결과에 nil이 있어도 그대로 유지
- 배열 크기 유지

##### compactMap

```Swift
func compactMap(_ transform: (Element) -> T?) -> [T]
```

- 모든 요소에 변환 함수 적용 
- **nil 자동 제거** 
- **옵셔널 자동 언래핑** 
- 배열 크기 줄어들 수 있음

|        | map          | compactMap |
| ------ | ------------ | ---------- |
| nil 처리 | nil 유지       | nil 자동 제거  |
| 반환 타입  | `[Type?]` 가능 | `[Type]`   |
| 크기     | 항상 동일        | 줄어들 수 있음   |

#### 활용
---
1. 옵셔널 배열 언래핑

```Swift
let numbers: [Int?] = [1, nil, 3, nil, 5] 

// map - 그대로 
numbers.map { $0 } // [Optional(1), nil, Optional(3), nil, Optional(5)] 

// compactMap - nil 제거 
numbers.compactMap { $0 } // [1, 3, 5]
```