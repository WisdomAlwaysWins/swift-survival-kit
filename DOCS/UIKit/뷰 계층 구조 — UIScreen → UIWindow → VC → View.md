#uikit #view-hierarchy

>[!info] 뷰 계층 구조, 왜 알아야하는가?
>SceneDelegate에서 rootViewController를 설정하는 코드를 이해하려면 이 계층을 알아야한다. addSubview 순서를 잘못하면 크래시가 나는 것도..

## 1. UIKit 뷰 계층 구조

![[뷰 계층 구조.png]]

| 계층                 | 역할                                       |
| ------------------ | ---------------------------------------- |
| UIScreen           | 물리적 디스플레이 (보통 1개)                        |
| UIWindowScence     | Scene 단위로 멀티 윈도우 지원 (iPad)               |
| UIWindow           | 눈에 보이지 않는 최상위 컨테이너                       |
| rootViewController | 앱의 첫 화면                                  |
| UIView             | 실제 화면에 보이는 요소로 superview → subview 트리 구조 |

---
## 2. SceneDelegate에서 Window 설정

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
	var window: UIWindow?
	
	    func scene(
	    _ scene: UIScene,
		willConnectTo session: UISceneSession,
        options connectionOptions: UIScene.ConnectionOptions
    ) {
	    guard let windowScene = scene as? UIWindowScene else { return }
	    
	    let window = UIWindow(windowScene: windowScene)
	    
	    // NavigationController로 감싸기
	    let homeVC = HomeVC()
	    window.rootViewController = UINavigationController(rootViewController: homeVC)
	    
	    self.window = window
	    window.makeKeyAndVisible() // 이 window를 화면에 표시
    }
}
```

---
## 3. UIView의 superview / subview 트리

```swift
// 뷰 추가
let parentView = UIView()
let childA = UILabel()
let childb = UIButton()

view.addSubview(parentView)
parentView.addSubview(childA)
parentView.addSubvieww(childB)

// 트리 구조:
// view
// └─ parentView 
//    ├─ childA (Label) 
//    └─ childB (Button)

// 관계 확인
print(childA.superview === parentView) // true 
print(parentView.subviews) // [childA, childB]
```

#### 3.1 Subview의 순서가 중요한 이유

**`addSubview` → translates = false → 제약 설정** 순서를 어기면 런타임 에러가 난다!

```swift
// ❌ 잘못된 순서 — superview에 추가 전에 제약 설정
childView.translatesAutoresizingMaskIntoConstraints = false childView.topAnchor.constraint(equalTo: parentView.topAnchor).isActive = true // → superview가 nil이라 크래시!

// 올바른 순서
parentView.addSubview(childView) // 1. 먼저 추가
childView.translatesAutoresizingMaskIntoConstraints = false // 2. translates 끄기
NSLayoutConstraint.activate([ // 3. 제약 설정 
	childView.topAnchor.constraint(equalTo: parentView.topAnchor, constant: 10),
	childView.leadingAnchor.constraint(equalTo: parentView.leadingAnchor, constant: 10), 
])
```

>[!abstract] `translatesAutoresizingMaskIntoConstraints`가 뭔데?
>옛날 방식인 Auto Resizing Mask를 Auto Layout 제약으로 자동 변환할지 설정하는 것이다. 코드로 뷰를 만들면 기본값이 true라서, 내가 만든 제약과 자동 생성 제약이 충돌한다. 코드로 Auto Layout을 쓸 때 반드시 false로 꺼야한다!

---
## ❓ 스스로에게 물어봐

Q1. UIKit의 뷰 계층 구조 설명

>최상위에 UIScreen (물리적 화면)
>UIWindowScene이 Scene 단위를 관리 (멀티 윈도우, iPad)
>UIWindow (눈에 보이지 않는 컨테이너)
>rootViewController
>subview
