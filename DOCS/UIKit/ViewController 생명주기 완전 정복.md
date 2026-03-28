#uikit #viewcontroller #lifecycle 

>[!info] ViewController, 왜 중요한가?
>"viewDidLoad에서 API를 호출해도 되나요?", "viewWillAppear와 viewDidAppear 중 어디서 애니메이션을 시작하나요?" — 생명주기를 모르면 이런 기본 질문에 답할 수 없다...

## 1. 개념 정리

#### 1.1 생명주기 메서드 순서

![[viewContoller-생명주기.png]]

#### 1.2 각 메서드의 역할

| 메서드                     | 호출 시점             | 해야 할 작업                                      | 호출 횟수 |
| ----------------------- | ----------------- | -------------------------------------------- | ----- |
| `viewDidLoad`           | 뷰가 메모리에 로드        | UI 초기 설정, Delegate/DataSource 연결, 1회성 API 호출 | 1회    |
| `viewWillAppear`        | 화면에 나타나기 직전       | 데이터 새로고침, NavigationBar 설정                   | 매번    |
| `viewDidLayoutSubviews` | Auto Layout 계산 완료 | frame 기반 레이아웃 조정 (cornerRadius 등)            | 매번    |
| `viewDidAppear`         | 화면에 완전히 나타남       | 애니메이션 시작, 타이머 시작                             | 메번    |
| `viewWillDisappear`     | 화면에서 사라지기 직전      | 키보드 닫기, 편집 모드 종료, 데이터 임시 저장                  | 매번    |
| `viewDidDisappear`      | 화면에서 완전히 사라짐      | 타이머 해제, 관찰자 제거, 리소스 정리                       | 매번    |

>[!question] viewDidLoad에서 frame이 확정되지 않는 이유는?
>`viewDidLoad` 시점에는 뷰가 메모리에 로드만 됐을 뿐, Auto Layout이 아직 계산되지 않았다. frame이 확정되는 시점은 `viewDidLayoutSubviews`다. 그래서 `viewDidLoad`에서 `view.frame.width`를 읽으면 예상과 다른 값이 나올 수 있다.

```swift
class ProfileVC: UIViewController {
	let avatarView = UIView()
	
	override func viewDidLoad() {
		super.viewDidLoad()
		view.addSubview(avatarView)
		// 여기서 avatarView.frame.width는 아직 0일 수 있음
	}
	
	override func viewDidLayoutSubviews() {
		super.viewDidLayoutSubviews()
		// 여기서 frame이 확정됨
		avatarView.layer.cornerRadius = avatarView.frame.width / 2
	}
}
```

#### 1.3 Navigation push / pop 시 생명 주기

```
A(현재) → B(push)

A: viewWillDisappear
B: viewDidLoad (첫 push 시 1회만)
B: viewWillAppear
A: viewDidDisappear
B: viewDidAppear

B → A(pop, B에서 뒤로 가기)

B: viewWillDisappear
A: viewWillAppear      ← A의 viewDidLoad는 호출 안 됨! (이미 메모리에 있음)
B: viewDidDisappear
A: viewDidAppear
B: deinit              ← B가 메모리에서 해제
```

>[!warning] pop할 때 돌아가는 VC의 `viewDidLoad`는 호출하지 않는다!
>A는 Navigation Stack에 남아있었으므로 이미 메모리에 있다. `viewWillAppear`부터 호출된다. 데이터 갱신이 필요하면 `viewWillAppear`에서 해야 한다.

#### 1.4 Modal persent / dismiss 시 생명 주기

```
A(현재) → B(present, fullScreen)

A: viewWillDisappear
B: viewDidLoad
B: viewWillAppear
A: viewDidDisappear
B: viewDidAppear

B → A(dismiss)  

B: viewWillDisappear
A: viewWillAppear
B: viewDidDisappear
A: viewDidAppear
B: deinit
```

>[!question] pageSheet present에서는 A가 사라지지 않는데?
>`modalPresentationStyle = .pageSheet`(기본)이면 A가 뒤에 보이므로 `viewWillDisappear`가 **호출되지 않는다**. `.fullScreen`이나 `.overFullScreen`에서만 A의 disappear가 호출된다.

#### 1.5 super 호출이 필수인 이유

```swift
override func viewWillAppear(_ animated: Bool) {
	super.viewWillAppear(animated) // 반드시 호출!
	// UIKit 내부 동작(NavigationBar 업데이트 등)이 super에 있음
}
```

>[!question] super를 안 부르면?
>UIKit 내부에서 수행하는 작업(네비게이션 바 갱신, 회전 처리, 탭바 업데이트 등)이 누락되어 예상치 못한 버그가 발생한다. Apple 문서에서 모든 생명주기 메서드에서 super 호출을 필수로 명시하고 있다.

---
## 2. 실전 코드 보기

```swift
class OrderListVC: UIViewController {
	var order: [Order] = []
	
	override func viewDidLoad() {
		super.viewDidLoad()
		// 1회성 설정
		tableView.delegate = self
		tableView.dataSource = self
		tableView.register(OrderCell.self, forCellReuseIdentifier: "cell")
		title = "주문 목록"
	}
	
	override func viewWillAppear(_ animated: Bool) {
		super.viewWillAppear(animated)
		// 매번 최신 데이터 로드 (다른 화면에서 돌아올 때도)
		fetchOrders()
		navigationController?.setNavigationBarHidden(false, animated: animated)
	}
	
	override func viewDidAppear(_ animated: Bool) {
		super.viewDidAppear(animated)
		// 화면이 완전히 보인 후 애니메이션 시작
		animateNewOrderBadge()
	}
	
	override func viewDidLayoutSubviews() {
		super.viewDidLayoutSubviews()
		// frame 확정 후 cornerRadius 적용
		headerView.layer.cornerRadius = headerView.frame.height / 2
	}
	
	override func viewWillDisappear(_ animated: Bool) {
		super.viewWillDisappear(animated)
		// 키보드 닫기
		view.endEditing(true)
	}
}
```
---
## ❓ 스스로에게 물어봐

Q1. ViewController 생명주기 순서

>viewDidLoad → viewWillAppear → viewDidLayoutSubviews → viewDidAppear 순으로 나타나고, viewWillDisappear → viewDidDisappear 순서로 사라진다. 
>viewDidLoad는 뷰가 메모리에 처음 로드될 때 1회만 호출되고, 나머지는 화면이 나타나거나 사라질 때 매번 호출된다. 

Q2. viewDidLoad에 frame을 읽으면 안되는 이유는?

>viewDidLoad 시점에는 AutoLayout이 아직 계산되지 않아 프레임이 확정되지 않는다. 프레임 기반 작업은 viewDidLayoutSubviews에서 해야한다.

Q3. push / pop 시 생명주기

>A : 현재 화면
>B : push 되는 화면
>
>A: viewWillDisappear
>B: viewDidLoad
>B: viewWillAppear
>A: viewDidDisappear
>B: viewDidAppear
>
>B : pop
>
>B: viewWillDisappear
>A: viewWillAppear
>B: viewDidDisappear
>A: viewDidAppear
>B: deinit