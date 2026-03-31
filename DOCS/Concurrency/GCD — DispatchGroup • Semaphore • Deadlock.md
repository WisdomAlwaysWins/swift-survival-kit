#concurrent #gcd #deadlock

>[!info] 왜 중요한가?
>여러 이미지를 동시에 다운로드하고 전부 끝나고 UI를 갱신하려면 DispatchGroup이 필요하다.
>DispatchQueue.main.sync가 왜 데드락을 일으키는지 모르면 앱 만들 때 앱을 멈추게 만들어버릴 수 있겠죠?

## 1. 개념 정리
#### 1.1 DispatchGroup — 여러 작업 동기화

> **DispatchGroup** 
> A group of tasks that you monitor as a single unit.

여러 비동기 작업을 하나의 그룹으로 묶고, 전부 끝날때 알림을 받는 도구다.

```swift
let group = DispatchGroup()

// 작업 1
group.enter()
downloadImage(url: url1) { image in
	iamges.append(image)
	group.leave()
}

// 작업 2
group.enter()
downloadImage(url: url2) { image in
	iamges.append(image)
	group.leave()
}

// 작업 3
group.enter()
downloadImage(url: url3) { image in
	iamges.append(image)
	group.leave()
}

// 전부 끝나면 실행
group.notify(queue: .main) {
	self.collectionView.reloadData()
	print("이미지 \(image.count)개 로딩 완료")
}
```

>[!warning] `enter()`와 `leave()`의 개수가 반드시 같아야 한다
>enter > leave이면 notify가 영원히 호출 안 되고, leave > enter이면 크래시. 
>`defer { group.leave() }`로 보장하는 게 좋다.
>```swift
>// ✅ defer로 보장
>group.enter()
>downloadImage(url: url1) { image in
>	defer { group.leave() }  // 이 클로저가 어떻게 끝나든 반드시 leave 실행
>	
>	guard let image else {
>		return  // 여기서 return해도 → defer가 leave() 호출해줌 ✅
>	}
>	images.append(image)
>	// 정상 종료해도 → defer가 leave() 호출해줌 ✅
>}
>```

#### 1.2 DispatchSemaphore — 동시 접근 수 제한

>**DispatchSemaphore** 
>An object that controls access to a resource across multiple execution contexts through use of a traditional counting semaphore.

동시에 접근할 수 있는 스레드 수를 제한하는 도구다.

- wait() : 들어간다 (칸 하나 차지, value - 1)
- signal() : 나온다 (칸 하나 비움, value + 1)

```swift
let semaphore = DispatchSemaphore(value: 3) // 최대 3개 동시 다운로드

for url in imageURLs {
	DispatchQueue.global().async {
		semaphore.wait() // 카운터 - 1 (0이면 대기)
		defer { semaphore.signal() } // 카운터 + 1
		let image = downloadSync(url) 
		processImage(image)
	}
}
```

#### 1.3 DispatchWorkItem — 취소 가능한 작업

```swift
var searchWorkItem: DispatchWorkItem?

func search(query: String) {
    searchWorkItem?.cancel()  // 이전 검색 취소
    
    let workItem = DispatchWorkItem {
        let results = performSearch(query)
        DispatchQueue.main.async {
            self.updateResults(results)
        }
    }
    
    searchWorkItem = workItem
    DispatchQueue.global().asyncAfter(deadline: .now() + 0.3, execute: workItem)
    // 0.3초 debounce — 타이핑이 멈추면 검색 실행
}
```

#### 1.4 DispatchQueue.main.sync — 데드락!

```swift
// ❌ 데드락! (메인 스레드에서 실행 중일 때)
DispatchQueue.main.sync {
    print("이건 절대 실행 안 됨")
}

override func viewDidLoad() {      // ┐ 
	super.viewDidLoad()            // │ 
	                            // │ 이게 한 뭉텅이 
	DispatchQueue.main.sync {      // │ (하나의 작업) 
		self.label.text = "완료"   // │ 
	}                              // │ 
	                            // │ 
	print("끝")                    // │
 }                                 // ┘
```

1. 현재 메인 스레드에서 코드 실행 중
2. main.sync → "이 블록을 메인 큐에 넣고, 끝날 때까지 기다려"
3. 메인 큐는 Serial → 현재 실행 중인 작업이 끝나야 다음 작업 실행
4. 하지만 현재 작업은 sync 블록이 끝나기를 기다리고 있음
5. sync 블록은 현재 작업이 끝나기를 기다리고 있음
6. → 서로를 영원히 기다리며 멈춤 → 데드락

```
메인 스레드: "sync 블록 끝나면 계속 할게" → 기다림 ...
main Queue: "현재 작업 끝나면 sync 블록 실행할게" → 기다림 ...
```

>[!abstract] 이해가 안되면 비유를 해보자.
>메인스레드를 1인 사장님이 하는 미용실이라고 생각해보자.
>
>```
>예약 목록: [A 손님 커트] ← 지금 진행 중
>
>↓ 커트 중, 새 예약이 들어왔다 (main.sync)
>
>예약 목록: [A 손님 커트 (진행중)] → [염색 (대기)]
>sync이니까 염색이 끝나야 A 손님 커트의 다음 단계로 넘어갈 수 있음
>근데, A 손님 커트가 끝나야 예약 목록에서 다음 (염색)을 시작할 수 있음
>
>염색을 시작하려면 → 커트가 끝나야함
>커트를 끝내려면 → 염색이 끝나야함
→  데드락 💀
>```

#### 1.5 DeadLock 발생 조건 (모두 만족)

1. 상호 배제 (Mutual Exclusion) : 자원은 1개 스레드만 사용
2. 점유 대기 (Hold and Wait) : 자원을 점유한 채로 다른 자원 대기
3. 비선점 (No preemption) : 다른 스레드의 자운을 뺏을 수 없음
4. 순환 대기 (Circular Wait) : A→B→C→A 순환 대기

```swift
override func viewDidLoad() { // 메인 미용실 미용사가 커트 중 

	DispatchQueue.main.sync { // "메인 미용실에 예약 넣고 기다릴게" 
		self.label.text = "완료" 
	} 
}
```

>[!tip] 해결방법
>DispatchQueue.async를 쓰면 된다. async는 기다리지 않으므로 데드락이 발생하지 않는다.
>백그라운드 스레드에서는 main.sync를 사용해도 안전하다 (메인이 아닌 스레드가 기디라는 것이므로)
>```swift
>DispatchQueue.global().async { // 백그라운드 미용실에서 일 시작 
>	let result = heavyWork() 
>	
>	DispatchQueue.main.sync { // "메인 미용실에 예약 넣고 기다릴게" 
>		self.label.text = result // "메인 미용실 미용사인데, 앞에 아무도 없네? 바로 해줄게"
>	} 
>	print("끝") 
>}
>```

>[!warning] 기다리는 애랑, 작업해야하는 애랑 같은 놈이면 데드락이다.ᐟ.ᐟ

---
## ❓ 스스로에게 물어봐

Q1: DispatchQueue.main.sync는 왜 데드락이 발생하나요?

>[!faq]- 답
>메인 스레드에서 main.sync를 호출하면 메인 큐에 블록을 넣고 "끝날 때까지 기다린다"라고 하는데, 메인 큐는 Serial이라 현재 작업 중인 작업이 끝나야 다음 작업을 실행할 수 있다. 현재 작업은 sync가 끝나기를, sync는 현재 작업이 끝나기를 기다리며 영원히 멈춘다.

Q2: DispatchGroup은 어떻게 동작하나요?

>[!faq]- 답
>각 작업 시작 전에 enter() 호출하고, 완료 시 leave() 호출한다. 쌍이 맞으면 notify 에 등록한 클로저가 실행된다. 현대 Swift에서는 TaskGroup이 이를 대체한다.
