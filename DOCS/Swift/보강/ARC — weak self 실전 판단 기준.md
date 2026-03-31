#swift #arc #memory #weak-self

>[!info]
>기존 [[ARC — RC 동작, 순환 참조 5가지 패턴, weak와 unowned]]의 보강 파트다. ARC 기초(RC 추적, strong/weak/unowned, 순환 참조 5패턴)는 기존 노트를 참고하고, 이 노트에서는 **[weak self]** 언제 써야하고 언제 안 써도 되는지 판단 기준만 다룬다.

## 1. 핵심 원칙: "이 클로저가 self를 오래 잡고 있을 수 있는가?"

이걸 판단하는 기준이 @escaping vs non-escaping 이다.

판단 플로우 차트
```
클로저에서 self를 쓴다
        │
        ▼
이 클로저가 @escaping인가?
        │
   ┌────┴────┐
   NO        YES
   │          │
   ▼          ▼
[weak self]  self가 이 클로저를
안 써도 됨    "소유"하고 있는가?
              (프로퍼티에 저장, 또는 self → 무언가 → 클로저)
              │
         ┌────┴────┐
         NO        YES
         │          │
         ▼          ▼
    써도 되고     [weak self]
    안 써도 됨     필수!
    (선택적)     (안 쓰면 retain cycle)
```

---
## 2. 케이스별 상세 — non-escaping

#### 2.1 self가 클로저를 소유 (retain cycle)

```swift
class ViewController: UIViewController {
	var onComplete: (() -> Void)? // self가 클로저를 소유 (프로퍼티로)
	
	func setup() {
	
		onComplete = {
			print(self.title)
		} // self → onComplete (클로저) → self => 순환!
		
		onComplete = {  [weak self] in
			guard let self else { return }
			print(self.title)
		}
	}
}
```

```
❌ 순환 참조 발생:
  
  ViewController ──(strong)──→ onComplete (클로저)
       ↑                            │
       └────────(strong: self)──────┘
       
       RC가 영원히 0이 되지 않음 → 메모리 누수
```

```
✅ [weak self]로 해결:
  
  ViewController ──(strong)──→ onComplete (클로저)
       ↑                            │
       └────────(weak: self)────────┘  ← RC 증가 안 함!
       
       VC = nil → RC: 0 → 해제! → 클로저도 함께 해제!
```

>[!warning] 이것이 가장 흔한 retain cycle 패턴.
>`self.프로퍼티 = { self.무언가 }` 의 형태를 본다면 즉시 순환 참조를 의심할 것.

#### 2.2 URLSession.dataTask

**VC가 pop되어 화면에서 사라져도, 네트워크 요청이 끝날 때까지 VC가 메모리에 남아있다는 것**이다. 불필요한 메모리 유지 + 해제된 화면에 데이터를 업데이트하려는 의미 없는 작업이 발생한다.

```swift
class ProfileVC: UIViewController {

    func fetchData() {
        // ❌ retain cycle은 아니다 — 하지만 문제가 있다
        URLSession.shared.dataTask(with: url) { data, _, _ in
            self.update(data)
        }
        // URLSession → 클로저 → self (단방향, 순환 아님)
        // 하지만! 네트워크 요청 끝날 때까지 self가 해제 안 됨
        
        // ✅ 해결
		URLSession.shared.dataTask(with: url) { [weak self] data, _, _ in
		    guard let self else { return }
		    self.update(data)  // VC가 이미 해제됐으면 아무것도 안 함
		}.resume()
    }
}
```

#### 2.3 DispatchQueue (상황에 따라 다름)

```swift
DispatchQueue.main.async {
	self.label.text = "완료"
} 
// main.async는 즉시 실행되고 클로저가 사라져서 self를 오래 잡고 있지 않는다.
// @escaping이긴 하지만, 실질적인 지연이 거의 없다.
```

```swift
DispatchQueue.main.asyncAfter(deadline: .now() + 5) { [weak self]
	self?.label.text = "5초"
}
// 5초 동안 클로저가 self를 잡고 있다. 그 사이 VC가 pop되면 불필요하게 5초동안 메모리를 차지한다.
```

#### 2.4 Delegate

소유권 방향이 VC → CustomView (strong), CustomView → VC (delegate) 이므로, delegate가 string이면 순환 참조가 발생한다.

```swift
protocol MyDelegate: AnyObject { }  // AnyObject 필수!

class MyView {
    weak var delegate: MyDelegate?   // weak 필수!
}
```