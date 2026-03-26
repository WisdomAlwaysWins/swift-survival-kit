#ios #os #sandbox

>[!info] iOS 계층 구조는 왜 알아야하는가?
>iOS 앱이 동작하는 환경을 이해하는 것으로, 우리가 쓰는 UIKit, SwiftUI는 최상위 계층일 뿐이고 그 아래에 미디어 처리, 데이터 서비스, 운영체제 커널이 쌓여 있다.

## 1. 개념 정리

#### 1.1 iOS 4계층 구조

iOS는 피라미트 형태로 4개의 계층으로 구성되어 있다. 아래로 갈수록 하드웨어에 가깝고, 위로 갈수록 개발자가 직접 쓰는 고수준의 API다. 각 계층은 아래 계층의 기능을 사용해서 더 높은 수준의 기능을 제공한다.

```swift
HIGH LEVEL (개발자가 주로 작업하는 곳)
┌───────────────────────────────────┐
│      Cocoa Touch Layer            │  UIKit, SwiftUI, MapKit, Push
├───────────────────────────────────┤
│      Media Layer                  │  AVFoundation, Core Animation, Metal
├───────────────────────────────────┤
│      Core Services Layer          │  Foundation, Core Data, Core Location
├───────────────────────────────────┤
│      Core OS Layer                │  Darwin Kernel, Security, Accelerate
└───────────────────────────────────┘
LOW LEVEL (하드웨어에 가까운 곳)
```

##### 1.1.1 Cocoa Touch Layer (최상위)

앱의 UI와 사용자 상호작용을 담당하는 최상위 계층으로, 개발자가 가장 많이 쓰는 계층이다.

| 프레임워크              | 역할            | 실전 예시                         |
| ------------------ | ------------- | ----------------------------- |
| UIKit              | 화면 구성, 이벤트 처리 | UIViewController, UITableView |
| SwiftUI            | 선언형 UI        | View, @State, @Binding        |
| MapKit             | 지도            | MKMapView                     |
| Push Notifications | 알림            | UNUserNotificationCenter      |
| Multitasking       | 멀티태스킹         | Background Modes              |

##### 1.1.2 Media Layer

그래픽, 오디오, 비디오 처리. 시각적 / 청각적 경험을 담당한다.

| 프레임워크          | 역할        | 실전 예시          |
| -------------- | --------- | -------------- |
| AVFoundation   | 오디오 / 비디오 | 카메라 촬영, 음악 재생  |
| Core Animation | 애니메이션     | UIVew.animate  |
| Core Graphics  | 2D 그래픽    | 도형 그리기, 이미지 처리 |
| Metal          | GPU 직접 접근 | 고성능 3D 렌더링     |
| Vision         | 컴퓨터 비전    | 얼굴 인식, 바코드 스캔  |

##### 1.1.3 Core Services Layer

앱의 기본 기능들 — 데이터 저장, 네트워크, 위치, 연락처 등. **권한(Permission)** 과 관련된 프레임워크가 많다.

| 프레임워크         | 역할                                | 실전 예시         |
| ------------- | --------------------------------- | ------------- |
| Foundation    | 기본 타입 데이터 (String, Array, Data 등) | -             |
| Core Data     | 로컬 데이터베이스                         | 복잡한 로컬 데이터 관리 |
| Core Location | GPS, 나침반                          | 위치 기반 서비스     |
| CloudKit      | iCloud 동기화                        | 기기 간 데이터 동기화  |
| HealthKit     | 건강 데이터                            | 운동량, 심박수      |

>[!question] 왜 Core Services 프레임워크들은 권한이 필요해?
>이 계층의 프레임워크들은 사용자의 **민감한 데이터** (위치, 연락처, 건강 정보)나 하드웨어(GPS)에 접근하기 때문이다. `Info.plist`에 사용 목적을 적고, 런타임에 사용자 동의를 받아야 한다.

##### 1.1.4 Core OS Layer (최하위)

하드웨어와 직접 소통하는 운영체제 레벨. Darwin 커널 (UNIX 기반) 위에 구축되어 있다.

| 프레임워크         | 역할                 | 실전 예시         |
| ------------- | ------------------ | ------------- |
| Darwin Kernal | 프로세스 / 메모리 / 파일 관리 | 직접 접하지 않음     |
| Security      | 암호화, keychain      | 토큰 저장, SSL    |
| Accelerate    | 수학 연산 최적화          | 이미지 필터, 신호 처리 |

#### 1.2 Sandbox 모델

각 앱이 **독립된 파일 시스템 공간** 을 갖는 보안 모델이다. 앱 A는 앱 B의 파일에 접근할 수 없다. 마치 각 앱이 자기만의 "모래 상자(sandbox)" 안에서 놀고 있는 것처럼, 다른 앱의 모래 상자에는 접근할 수 없다.

```
App의 Sandbox 디렉토리 구조:

AppName.app/
├── Documents/          ← 사용자가 생성한 데이터 (iCloud 백업 O)
├── Library/
│   ├── Caches/         ← 캐시 데이터 (시스템이 삭제 가능, 백업 X)
│   ├── Preferences/    ← UserDefaults 저장 위치
│   └── Application Support/  ← 앱이 관리하는 데이터
├── tmp/                ← 임시 파일 (시스템이 수시로 삭제)
└── SystemData/         ← 시스템 관리 영역
```

##### 1.2.1 각 디렉토리의 용도

| 디렉토리                | 용도                   | iCloud 백업 | 삭제 가능성           |
| ------------------- | -------------------- | --------- | ---------------- |
| Documents           | 사용자가 만든 파일, 중요 데이터   | O         | 사용자만 삭제          |
| Library/Caches      | 다시 다운로드 가능한 캐시       | X         | 시스템이 디스크 부족 시 삭제 |
| Library/Perferences | UserDefaults(.plist) | O         | 앱 삭제 시           |
| tmp                 | 임시 파일                | X         | 시스템이 수시로 삭제      |

```swift
// Documents 디렉토리 경로 얻기
let documentsPath = FileManager.default.urls(
    for: .documentDirectory, in: .userDomainMask
).first!

// Caches 디렉토리 경로 얻기
let cachesPath = FileManager.default.urls(
    for: .cachesDirectory, in: .userDomainMask
).first!

// tmp 디렉토리
let tmpPath = FileManager.default.temporaryDirectory
```

#### 1.3 App Group — 앱 간의 데이터 공유

같은 개발자의 앱들이 **Sandbox 경계를 넘어 데이터를 공유**할 수 있게 해주는 기능이다. 메인 앱과 위젯, 메인 앱과 Extension 사이에서 데이터를 주고받을 때 사용한다. 공유 UserDefaults와 공유 파일 컨테이너를 제공한다.

```swift
// App Group 설정 후 공유 UserDefaults
let sharedDefaults = UserDefaults(suiteName: "group.com.myapp.shared")
sharedDefaults?.set("공유 데이터", forKey: "sharedKey")

// 다른 앱(같은 그룹)에서 읽기
let value = sharedDefaults?.string(forKey: "sharedKey")
```

#### 1.4 KeyChain — 민감 정보 저장

**암호화된 저장소**로, 비밀번호, 토큰, 인증서 같은 민감한 정보를 안전하게 저장하는 곳이다. UserDefaults와 달리 앱을 삭제해도 Keychain 데이터는 유지될 수 있고, 하드웨어 레벨(Secure Enclave)에서 암호화된다.

```swift
import Security

// Keychain에 저장
func saveToken(_ token: String, for account: String) {
    let data = token.data(using: .utf8)!
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: account,
        kSecValueData as String: data
    ]

    SecItemDelete(query as CFDictionary)  // 기존 값 삭제
    SecItemAdd(query as CFDictionary, nil)
}

// Keychain에서 읽기
func loadToken(for account: String) -> String? {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: account,
        kSecReturnData as String: true
    ]

    var result: AnyObject?
    SecItemCopyMatching(query as CFDictionary, &result)
    guard let data = result as? Data else { return nil }
    return String(data: data, encoding: .utf8)
}
```

#### 1.5 데이터 저장소 선택 가이드

| 저장소       | 용도                  | 용량           | 보안          | 앱 삭제 시 |
| ------------ | --------------------- | -------------- | ------------- | ---------- |
| UserDefaults | 설정값, 간단한 데이터 | 작음 (MB 이하) | 낮음 (plist)  | 삭제됨     |
| Keychain     | 토큰, 비밀번호        | 작음           | 높음 (암호화) | 유지 가능  |
| FileManager  | 파일, 이미지          | 큼             | 중간          | 삭제됨     |
| Core Data    | 구조화된 데이터       | 큼             | 중간          | 삭제됨     |

---
## ❓ 스스로에게 물어봐

Q1. iOS 4계층 구조는?

>iOS는 Core OS, Core Services, Media, Cocoa Touch 4 계층으로 구성됩니다. 
>
>- Core OS는 Darwin 커널 기반으로 프로세스 관리, 보안, 메모리 관리를 담당합니다. 
>- Core Services는 Foundation, Core Data, Core Location 등 앱의 기본 기능을 제공합니다. 
>- Media는 AVFoundation, Core Animation, Metal 등 멀티미디어 처리를 담당합니다. 
>- Cocoa Touch는 UIKit, SwiftUI 등 UI와 사용자 상호작용을 담당하며, 개발자가 가장 많이 작업하는 계층입니다.

Q2. Keychain과 UserDefaults의 차이는?

>UserDefaults는 plist 파일로 저장되어 암호화되지 않습니다. 간단한 설정값에 적합합니다. Keychain은 하드웨어 레벨(Secure Enclave)에서 암호화되어 비밀번호, 토큰 같은 민감한 정보에 적합합니다.
>또한 앱을 삭제하면 UserDefaults는 함께 삭제되지만, Keychain 데이터는 유지될 수 있습니다.
