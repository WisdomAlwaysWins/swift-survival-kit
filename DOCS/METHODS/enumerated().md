
날짜 : 2026-01-15 (목)

>[!question] 
>배열을 순회할 때 인덱스와 요소를 어떻게 동시에 사용할까?


#### Description
---
배열의 인덱스와 요소를 튜플 형태로 반환하는 메서드로, 배열을 순회하면서 "몇 번째 요소인지"와 *"실제 값이 무엇인지"* 를 동시에 알아야 할 때 사용한다.

일반적인 for-in 루프는 값만 가져오지만, `enumerated()`를 사용하면 0부터 시작하는 순서 정보를 함께 얻을 수 있다. `map`, `filter`와 같은 고차함수와 조합하여 인덱스 기반 변환 작업을 수행할 때 특히 유용하다.

#### 개념
---
##### 기본 문법

```Swift
array.enumerated() → EnumeratedSequence
// 반환: [(offset: Int, element: Element)]
```

- 각 요소를 `(offset, element)` 튜플로 반환
	- offset은 0부터 시작하는 인덱스
	- element는 배열의 실제 값

##### 기본 예제

```Swift
let fruits = ["사과", "바나나", "오렌지"]

for (index, fruit) in fruits.enumerated() {
	print("\(index) : \(fruit)")
}
```

#### 활용
---
1.  map 과 함께 사용
```Swift
let fruits = ["사과", "바나나", "오렌지"] 

let result = fruits.enumerated().map { (offset, element) in 
	return "\(offset + 1)번: \(element)" 
}
// ["1번: 사과", "2번: 바나나", "3번: 오렌지"]
```

2. 짧게 쓰기
```Swift
// $0.offset, $0.element로 접근
let result = fruits.enumerated().map {
	"\($0.offset + 1)번: \($0.element)"
}

// 이름 붙이기 (더 읽기 쉬움) 
let result = fruits.enumerated().map { index, fruit in 
	"\(index + 1)번: \(fruit)" 
}
```


