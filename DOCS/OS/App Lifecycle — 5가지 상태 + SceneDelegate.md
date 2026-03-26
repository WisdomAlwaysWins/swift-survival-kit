>[!info] 앱 라이프라이클, 왜 중요할까?
>앱이 백그라운드로 갈 때 데이터를 저장하고, 포그라운드로 돌아올 때 UI를 갱신하는 것은 App Lifecycle을 이해해야 가능하다.

## 1. 개념 정리

#### 1.1 앱의 5가지 상태

>[!abstract] App State
iOS 앱은 항상 다음 5가지 상태 중 하나에 있다. 상태 전환마다 시스템이 특정 메서드를 호출해주고, 개발자는 그 메서드 안에서 적절한 작업을 해야 한다.


![[LifeCycle.png]]

| 상태          | 설명                        | 코드 실행   | UI 표시 |
| ----------- | ------------------------- | ------- | ----- |
| Not Running | 앱이 실행되지 않은 상태             | X       | X     |
| Inactive    | 포그라운드지만 이벤트를 받지 않는 과도기 상태 | O (제한적) | O     |
| Active      | 포그라운드에서 사용자와 상호작용 중       | O       | O     |
| Background  | 백그라운드에서 코드 실행 중 (약 5초 제한) | O (제한적) | X     |
| Suspended   | 메모리에 있지만 코드 실행 안 함        | X       | X     |

> [!question] Inactive 상태는 언제 발생해?
> Active로 가기 전/후의 **과도기 상태**다. 전화가 오거나, 앱 전환기가 열리거나, 제어센터를 내리거나, 알림 배너가 뜰 때 잠깐 Inactive가 된다. 사용자의 터치 이벤트를 받지 않는다.

#### 1.2 iOS 12 이전 (참고만 해라)

SceneDelegate가 없었고, **AppDelegate 하나가 앱 전체 + UI 생명주기를 모두 관리**했다. `applicationDidBecomeActive`, `applicationDidEnterBackground` 등의 메서드가 AppDelegate에 있었다. 레거시 프로젝트에서 볼 수 있지만, 지금은 SceneDelegate 방식이 표준이므로 "옛날에는 이랬다" 정도만 알면 충분하다.

#### 1.3 iOS 13+ — SceneDelegate 도입

iOS 13부터 iPad에서 **멀티 윈도우(Split View)** 를 지원하게 되었다. 같은 앱의 창이 여러 개 열릴 수 있으니, "앱 전체"의 생명주기와 "각 창(Scene)"의 생명주기를 분리해야 했다

- **AppDelegate**: 앱 전체 수준 — 앱 시작, 원격 알림 등록 등 (1개) 
- **SceneDelegate**: 각 창(Scene) 수준 — UI 표시, 포그라운드/백그라운드 전환 (여러 개 가능) 

```swift
// AppDelegate — 앱 전체 (iOS 13+)
@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        print("앱 전체 초기화 — Firebase, 푸시 알림 등록")
        return true
    }
    
    // Scene 설정을 반환
    func application(_ application: UIApplication,
                     configurationForConnecting connectingSceneSession: UISceneSession,
                     options: UIScene.ConnectionOptions) -> UISceneConfiguration {
        return UISceneConfiguration(name: "Default", sessionRole: connectingSceneSession.role)
    }
}

// SceneDelegate — 각 창(Scene)별
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?
    
    // Scene 생성 — 초기 화면 설정
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
               options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = scene as? UIWindowScene else { return }
        let window = UIWindow(windowScene: windowScene)
        window.rootViewController = UINavigationController(rootViewController: HomeVC())
        window.makeKeyAndVisible()
        self.window = window
    }
    
    func sceneDidBecomeActive(_ scene: UIScene) {
        print("Scene 활성화 — 이 창의 타이머 재개")
    }
    
    func sceneWillResignActive(_ scene: UIScene) {
        print("Scene 비활성화 — 이 창의 작업 일시정지")
    }
    
    func sceneDidEnterBackground(_ scene: UIScene) {
        print("Scene 백그라운드 — 이 창의 데이터 저장")
    }
    
    func sceneWillEnterForeground(_ scene: UIScene) {
        print("Scene 포그라운드 복귀 — 이 창의 UI 갱신")
    }
}
```

#### 1.4 iOS 12 이전 vs iOS 13+ 비교

| 항목 | iOS 12 이전 | iOS 13+ |
|------|-----------|---------|
| UI 생명주기 관리 | AppDelegate | SceneDelegate |
| 앱 전체 관리 | AppDelegate | AppDelegate (축소) |
| 초기 화면 설정 | AppDelegate | SceneDelegate |
| 멀티 윈도우 | 불가 | 가능 (iPad) |
| `window` 프로퍼티 위치 | AppDelegate | SceneDelegate |

### 1.5 SwiftUI의 App 프로토콜
```swift
import SwiftUI

@main
struct MyApp: App {
    @Environment(\.scenePhase) var scenePhase
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: scenePhase) { oldPhase, newPhase in
            switch newPhase {
            case .active:
                print("활성화 — UI 갱신")
            case .inactive:
                print("비활성화 — 임시 저장")
            case .background:
                print("백그라운드 — 중요 데이터 저장")
            @unknown default:
                break
            }
        }
    }
}
```

> [!abstract] `scenePhase`가 뭔데?
> SwiftUI에서 앱의 상태 변화를 감지하는 Environment 값이다. `.active`, `.inactive`, `.background` 3가지 상태를 제공한다. UIKit의 SceneDelegate 메서드들을 대체한다.

### 1.6 각 상태 전환에서 해야 할 작업

| 전환                          | 해야 할 작업                     |
| --------------------------- | --------------------------- |
| **앱 시작**                    | DB 초기화, 원격 설정 로드, 푸시 알림 등록  |
| **Active**                  | 타이머 재개, UI 갱신               |
| **Active → Inactive**       | 게임 일시정지, 진행 중 작업 임시 저장      |
| **Inactive → Background**   | 중요 데이터 저장, 네트워크 정리 (5초 제한!) |
| **Background → Foreground** | 데이터 새로고침, UI 갱신, 토큰 갱신 확인   |
| **메모리 경고**                  | 캐시 삭제, 불필요한 이미지 해제          |

---
## ❓ 스스로에게 물어봐

Q1: 앱의 5가지 상태

>[!faq]- 답
>1. Not Running은 앱이 실행되지 않은 상태입니다. 
>2. Inactive는 포그라운드지만 이벤트를 받지 않는 과도기 상태로, 전화가 오거나 앱 전환기가 열릴 때 발생합니다.
>3. Active는 포그라운드에서 사용자와 상호작용하는 정상 상태입니다. 
>4. Background는 백그라운드에서 약 5초간 코드를 실행할 수 있는 상태입니다. 
>5. Suspended는 메모리에는 있지만 코드가 실행되지 않는 상태로, 메모리가 부족하면 시스템이 종료시킵니다.

Q2: SwiftUI에서 앱 생명주기를 어떻게 감지하나요?

>[!faq]- 답
>@Environment(\.scenePhase)를 사용합니다. .active, .inactive, .background 3가지 상태를 제공하고 .onChange(of: scenePhase)로 상태 변화를 감지할 수 있습니다.

Q3. iOS 13+에서 초기 화면(rootViewController)은 AppDelegate에서 설정한다.

> [!faq]- 정답
> **X** — SceneDelegate의 `scene(_:willConnectTo:options:)`에서 설정한다.

Q4. applicationWillTerminate는 iOS 13+에서 항상 호출된다.

> [!faq]- 정답
> **X** — 호출이 보장되지 않는다. 중요한 저장은 didEnterBackground에서 해야 한다.

**Q5.** 사용자가 앱을 사용 중(Active)에 홈 버튼을 누르면, UIKit에서 어떤 메서드가 순서대로 호출되는가?

> [!faq]- 정답
> 1. `sceneWillResignActive` (Active → Inactive)
> 2. `sceneDidEnterBackground` (Inactive → Background)
>
> 이후 시간이 지나면 Suspended로 전환되지만, 이때는 메서드 호출 없이 시스템이 자동 처리.

**Q6.** 사용자가 백그라운드에 있던 앱을 다시 탭하면, 어떤 메서드가 순서대로 호출되는가?

> [!faq]- 정답
> 1. `sceneWillEnterForeground` (Background → Inactive)
> 2. `sceneDidBecomeActive` (Inactive → Active)



