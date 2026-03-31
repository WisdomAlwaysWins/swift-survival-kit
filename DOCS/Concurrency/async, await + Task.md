#concurreny #async-await #task

>[!info] 왜 중요한가?
>Swift 5.5에서 도입된 async/await은 콜백 지옥을 해결하고, 비동기 코드를 동기 코드처럼 읽을 수 있게 만든다. 최신 프로젝트에서는 GCD 대신 async/await이 표준이 되어가고 있다.

## 1. 개념 정리
#### 1.1 콜백 지옥 → async/await

**콜백 방식의 문제점**
- 클로저 중첩으로 가독성 저하
- completion handler 호출을 빠드릴 수 있다
- throws/try와 결합 불가 → 에러를 콜백으로 전달해야함
- `[weak self]` retain cycle 관리 필요

```swift
// ═══ 콜백 지옥 (GCD 시절) ═══
func loadProfile(userId: String, completion: @escaping (Profile?) -> Void) {
	// 1. 유저 정보 가져와. 끝나면 이 클로저 실행해:
    fetchUser(userId) { user in                    // 1단계 들여쓰기
        guard let user else { completion(nil); return }
        
        // 2. 아바타 가져와. 끝나면 이 클로저 실행해:
        fetchAvatar(user.avatarURL) { image in     // 2단계 들여쓰기
            guard let image else { completion(nil); return }
            
            // 3. 게시물 가져와. 끝나면 이 클로저 실행해
            fetchPosts(userId) { posts in          // 3단계 들여쓰기
                guard let posts else { completion(nil); return }
                
                // 4. 다 모였으니 Profile 만들어서 돌려줘
                let profile = Profile(user: user, avatar: image, posts: posts)
                completion(profile)
            }
        }
    }
}

// ═══ async/await (현대적) ═══
func loadProfile(userId: String) async throws -> Profile {
    let user = try await fetchUser(userId)          // 한 줄
    let avatar = try await fetchAvatar(user.avatarURL) // 한 줄
    let posts = try await fetchPosts(userId)        // 한 줄
    return Profile(user: user, avatar: avatar, posts: posts)
}
```

#### 1.2 async/await 

>An **asynchronous function** or asynchronous method is a special kind of function or method that can be **suspended while it's partway through execution**.
>When calling an asynchronous method, execution **suspends until that method returns**. You write **await** in front of the call to mark the possible suspension point.

async 함수가 **await 지점에서 중단(suspend)**되면, 해당 스레드는 블로킹되지 않고 다른 작업을 할 수 있다. GCD의 sync처럼 스레드를 붙잡고 있는게 아니다!

```swift
func downloadImage(from url: URL) async throws → UIImage {
	let (data, response) = try await URLSession.shared.data(from: url) 
	
	guard let httpResponse = response as? HTTPURLResponse, 
				httpResponse.statusCode == 200 else { 
		throw NetworkError.invalidResponse 
	} 
	
	guard let image = UIImage(data: data) else { 
		throw NetworkError.decodingFailed 
	} 
	return image
}
```

#### 1.3 Task 

>**Task** 
>A unit of asynchronous work.

동기 컨텍스트(viewDidLoad 등)에서 async 코드를 실행하려면 Task를 만들어야 한다.

```swift
@MainActor  // ViewController는 @MainActor
class ProfileVC: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        Task {
            // ✅ 여기는 메인 스레드
            isLoading = true
            loadingSpinner.startAnimating()
            
            // await 지점 — 여기서 중단(suspend)
            // 실제 네트워크 요청은 시스템이 백그라운드에서 처리
            // 메인 스레드는 이 동안 자유 (UI 안 멈춤!)
            let profile = try await fetchProfile()
            
            // ✅ 다시 메인 스레드로 돌아옴
            nameLabel.text = profile.name
            loadingSpinner.stopAnimating()
        }
    }
}
```

>[!tip] ViewController에서 만든 Task는 @MainActor를 상속한다 
>VC가 @MainActor로 격리되어 있으므로, 그 안에서 만든 Task도 메인 스레드에서 실행된다. 
>따라서 Task 내부에서 UI 업데이트가 가능하다.

GCD 방식으로 했다면?

```swift
// GCD 방식 — 직접 스레드 관리
DispatchQueue.global().async {           // 백그라운드로 보내고
    let profile = self.fetchProfile()
    DispatchQueue.main.async {           // 다시 메인으로 보내고
        self.nameLabel.text = profile.name
    }
}
```

#### 1.4 async let — 병렬 실행

```swift
// ❌ 순차 실행 — 느림
func loadDashboard() async throws -> Dashboard {
    let user = try await fetchUser()       // 2초
    let orders = try await fetchOrders()   // 3초
    let reviews = try await fetchReviews() // 1초
    return Dashboard(user: user, orders: orders, reviews: reviews)
    // 총 6초!
}

// ✅ 병렬 실행 — 빠름
func loadDashboard() async throws -> Dashboard {
    async let user = fetchUser()       // 동시에 시작
    async let orders = fetchOrders()   // 동시에 시작
    async let reviews = fetchReviews() // 동시에 시작
    
    return try await Dashboard(user: user, orders: orders, reviews: reviews)
    // 총 3초! (가장 오래 걸리는 것 기준)
}
```

#### 1.5 Task Group — 동적 병렬 처리

```swift
func downloadAllImages(urls: [URL]) async throws -> [UIImage] {
    try await withThrowingTaskGroup(of: UIImage.self) { group in
        for url in urls {
            group.addTask {
                try await self.downloadImage(from: url)
            }
        }
        
        var images: [UIImage] = []
        for try await image in group {
            images.append(image)
        }
        return images
    }
}
```

>[!question] async let vs Task Group 차이가 뭐야? 
>- **async let**: 개수가 컴파일 타임에 정해진 경우 (3개의 API 동시 호출) 
>- **Task Group**: 개수가 런타임에 결정되는 경우 (URL 배열의 이미지 동시 다운로드)

#### 1.6 구조적 동시성 (Structured Concurrency)

>Structured concurrency APIs (including task groups and async let) always **wait for the completion of tasks** contained within their scope before returning.

부모 Task와 자식 Task가 **트리 구조**로 연결된다:
- 부모가 취소되면 → 자식도 **자동 취소**
- 자식이 모두 끝나야 → 부모가 끝남

```swift
// 구조적 (자동 취소 전파)
func loadData() async throws {
    async let a = fetchA()  // 자식 Task
    async let b = fetchB()  // 자식 Task
    // loadData가 취소되면 → a, b도 자동 취소
    let result = try await (a, b)
}

// 비구조적 (수동 관리 필요)
func startWork() {
    let task = Task {
        await doSomething()
    }
    // task.cancel()로 수동 취소해야 함
}
```

#### 1.7 작업 취소

```swift
func fetchLargeData() async throws -> Data {
    var accumulated = Data()
    
    for chunk in chunks {
        // 취소 확인 — 취소되었으면 CancellationError를 throw
        try Task.checkCancellation()
        
        // 또는 직접 확인
        if Task.isCancelled {
            return accumulated  // 중간 결과 반환
        }
        
        let data = try await download(chunk)
        accumulated.append(data)
    }
    return accumulated
}
```

---
## ❓ 스스로에게 물어봐

Q1: async/await이 GCD보다 나은 점은?

>[!faq]- 답
>1. 가독성 (콜백 지옥에서 벗어남)
>2. 에러 처리 (throws/try와 자연스럽게 결합되어 do-catch로 처리)
>3. 구조적 동시성 (부모가 최소되면 자식도 자동 취소되어 리소스 정리가 체계적)
>4. await는 스레드를 블로킹하지 않고 중단만 하므로 데드락 위험이 GCD보다 낮음

Q2. await 지점에서 무슨 일이 일어나나요?

>[!faq]- 답
>async 함수가 await에서 중단되면 스레드는 블로킹되지 않고 해제되어 시스템이 다른 작업을 스케줄링할 수 있다. 호출된 async 함수가 완료되면 결과가 원래 함수로 돌아오고 실행이 재개된다.

Q3. 다음 코드의 총 실행 시간은? (fetchA는 2초, fetchB는 3초)

```swift
async let a = fetchA()
async let b = fetchB()
let result = try await (a, b)
```

>[!faq]- 답
>약 3초