#swift #type-casting

>[!info] 타입 캐스팅, 왜 알아야하는가?
>상속 구조에서 부모 타입으로 담긴 객체를 실제 자식 타입으로 다시 꺼내려면 타입 캐스팅이 필요하다. UIKit에서 `tableView.dequeueReusableCell`이 UITableViewCell을 반환하는데, 실제로는 커스텀 셀이니까 다운캐스팅해서 써야 한다.

## 1. 개념 정리

```swift
class Media {
	let title: String
	init(title: String) { self.title = title }
}

class Movie: Media {
	let director: String
	init(title: String, director: String) {
		self.director = director
		super.init(title: title)
	}
}

class Music: Media {
	let artist: String
	init(title: String, artist: String) {
		self.artist = artist
		super.init(title: title)
	}
}
```

#### 1.1 is — 타입 확인

```swift
let item: Media = Movie(title: "인셉션", director: "놀란")
print(item is Movie) // true
print(item is Music) // false
```

#### 1.2 as — 업캐스팅 (자식 → 부모)

```swift
let movie = Movie(title: "인셉션", director: "놀란")
let media = movie as Media // 항상 성공 (as 성공)
```

#### 1.3 as? — 조건부 다운캐스팅

```swift
let items: [Media] = [
	Movie(title: "인셉션", director: "놀란"),
	Music(title: "BIRDS OF FEATHER", artist: "Billie Eilish")
]

for item in items {
	if let movie = item as? Movie {
		print("영화 : \(movie.director)")
	} else if let music = item as? Music {
		print("음악: \(music.artist))
	}
}
```

#### 1.4 as! — 강제 다운캐스팅

```swift
let definiteMovie: Media = Movie(title: "테넷", director: "놀란")
let movie = definiteMovie as! Movie  // 확실할 때만! 실패하면 크래시
```

#### 1.5 Any vs AnyObject

- **Any**: 모든 타입 (struct, class, enum, 함수, 클로저 전부)
- **AnyObject**: class 타입만

```swift
var anything: [Any] = [1, "Hello", true, Movie(title: "A", director: "B")]
var objects: [AnyObject] = [Movie(title: "A", director: "B")]
// objects.append(1) // 🚩 Int는 struct이므로 AnyObject에는 못 담음
```

---
## ❓ 스스로에게 물어봐

Q1. `as?` 는 실패 시 nil을 반환한다 → O

Q2. Any는 class 타입만 담을 수 있다 → X, 모든 타입을 담을 수 있음