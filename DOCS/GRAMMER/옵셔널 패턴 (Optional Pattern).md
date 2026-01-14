
>[!question] 헷갈리는 포인트
>1. Optional Chaining 의 결과도 항상 Optional

#### 개념

* 옵셔널 값을 **안전하게 추출(unwrap)하고 사용**하기 위한 다양한 패턴

#### 옵셔널 기본

```Swift
var name: String? = "J"
var age: Int? = nil

// 옵셔널은 값이 있거나(some), 없거나(nil)
enum Optional {
	case none // nil
	case some(wrapped)
}
```

---

#### 1. if let - 옵셔널 바인딩

```Swift
let name: String? = "J"

if let unwrapped = name {
	print("이름 \(unwrapped)") // 안전하게 사용
} else {
	print(" 이름 없음")
}

if let name {
	print(name)
}
```

#### 2. guard let - Early Return

```Swift
func greet(name: String?) {
	guard let name = name else {
		print("이름이 없습니다")
		return
	}
	
	print("안녕하세요, \(name)님")
}
```

>[!question] `if let` vs `gaurd let`
>```Swift
>// if let - 스코프 안에서만 사용
>func checkAge(age: Int?) {
>	if let age {
>		print(age)
>	}
>	// 여기서는 age 사용 불가
>}
>```
>```Swift
>func checkAge(age: Int?) {
>	guard let age = age else { return }
>	print(age)
>	// 여기서도 age 사용 가능
>}
>```

#### 3. Nil Coalescing Operator (??)

```swift 
let name: String? = nil 
let displayName = name ?? "Guest" 
print(displayName) // "Guest" 

let age: Int? = 25 
let displayAge = age ?? 0 
print(displayAge) // 25 
```

```swift 
let first: String? = nil 
let second: String? = nil 
let third: String? = "J" 

let result = first ?? second ?? third ?? "Unknown" 
print(result) // "J" 
```

#### 4. Optional Chaining (?.)

```Swift
class Person {
	var name: String
	var pet: Pet?
	
	init(name: String) {
		self.name = name
	}
}

class Pet {
	var name: String
	
	init(name: String) {
		self.name = name
	}
}

let person = Person(name: "J")
let petName = person.pet?.name // nil

person.pet = Pet(name: "멍멍이")

let petName2 = person.pet?.name // "멍멍이"
```

```swift
let name: String? = "hello"
let upper = name?.uppercased() // Optional("HELLO")

let empty: String? = nil
let upper2 = empty?.uppercased() // nil
```

```swift
class Address {
	var street: String
	
	init(street: String) {
		self.street = street
	}
}

class Company {
	var address: Address?
}

class Person {
	var company: Company?
}

let person = Person()
let street = person.company?.address?.street // nil
```

#### 5. if case let - 패턴 매칭

```swift
let num: Int? = 5

if case let .some(value) = num {
	print(value)
}

if case let value? = num { // 위와 동일하게 동작
	print(value)
}
```

```swift
let num: Int? = nil

if case .none = num {
	print("값이 없습니다")
}


if case nil = num { // 위와 동일하게 동작
	print("값이 없습니다")
}
```

#### 6. switch - 옵셔널 처리

```swift
let age: Int? = 25

switch age {
case let value?:
	print("나이 \(value)")
case nil:
	print("나이 정보 없음")
}
```

