#concurrency #gcd 

>[!info] 왜 중요한가요?
>"메인 스레드에서 블로킹하면 앱이 멈춘다"는 것을 이해하려면 동기/비동기, Serial/Concurrent 개념을 알아야한다. GCD는 iOS에서 비동기 작업을 처리하는 가장 기본적인 도구이다.

## 1. 개념 정리
#### 1.1 동기 vs 비동기

- 동기 (Sync) : 작업이 끝날 때까지 기다린다.
	- ex. 카페에서 음료 나올 때까지 카운터 앞에서 기다리기
- 비동기 (Async) : 작업을 요청하고 바로 다음 일을 한다.
	- ex. 진동벨 받고 자리로 가서 다른 일 하고 있기

```swift
// 동기 - 메인 스레드 블로킹
let data = try Data(contentsOf: url) // 다운로드 끝날 때까지 멈춤
updateUI(with: data) // 다운로드 후에야 실행

// 비동기 - 메인 스레드 자유
DispatchQueue.global().async {
	let data = try? Data(contentsOf: url) // 백그라운드에서 다운로드
	DispatchQueue.main.async {
		self.updateUI(with: data) // 완료 후 메인에서 UI 갱신
	}
}
print("요청 완료") // 즉시 실행 (기다리지 않음)
```

#### 1.2 DispatchQueue

>**DispatchQueue** An object that manages the execution of tasks serially or concurrently on your app's main thread or on a background thread.

큐 자체가 스레드는 아니다. 큐는 작업을 받아 시스템이 관리하는 스레드 풀에 배분한다. 개발자가 스레드를 직접 생성/관리할 필요가 없다는 게 GCD의 핵심이다.

```
개발자가 하는 일 : 작업을 큐에 넣는다
시스템(GCD)이 하는 일 : 스레드 풀에서 적절한 스레드를 골라 작업을 실행한다.
```

#### 1.3 Serial Queue vs Concurrent Queue

- Serial Queue : 작업을 한번에 하나씩 순서대로 실행 (순서 보장)
- Concurrent Queue : 작업을 동시에 여러 개 진행. (순서 보장하지 않음, 빠름)

```
Serial Queue:
[작업1 ████] → [작업2 ██] → [작업3 ██]    총 8초

Concurrent Queue:
[작업1 ████]
[작업2 ██  ]                                총 4초
[작업3 ██  ]
```

#### 1.4 sync vs async

Apple 문서에서는 `async(execute:)`를 이렇게 설명한다.

>Schedules a work item for execution and **returns immediately**.

`sync(execute:)`는

>Submits a work item for execution on a dispatch queue and **waits until that block finishes executing**.

**"return immediately"**  vs "**waits until finishes"**. 이것이 async과 sync의 본질적 차이다.

#### 1.5 4가지 조합

| 조합                 | 순서 보장 | 메인 블로킹 | 병렬  | 권장  |
| ------------------ | ----- | ------ | --- | --- |
| Serial + Sync      | O     | O      | X   | X   |
| Serial + Async     | O     | X      | X   | X   |
| Concurrent + Sync  | X     | O      | X   | X   |
| Concurrent + Async | X     | X      | O   | O   |

```swift
// 가장 많이 쓰이는 조합
DispatchQueue.global().async { // Concurrent + Async
	let result = heavComputation()
	DispatchQueue.main.async { // 메인 스레드에서 UI 업데이트
		self.label.text = result
	}
}
```

#### 1.6 DispatchQueue의 3종류

##### 1.6.1 메인 큐 (Main Queue)

>The dispatch queue associated with the main thread of the current process.

```swift
DispatchQueue.main // Serial, 메인스레드, UI 전용, 앱에 1개 존재
```

##### 1.6.2 글로벌 큐 (Global Queue)

```swift
DispatchQueue.global() // Concurrent, 백그라운드 스레드
DispatchQueue.global(qos: .userInitiated) // QoS 지정 가능
```

##### 1.6.3 커스텀 큐 (Custom Queue)

```swift
// Apple 문서의 init 시그니처:
// init(label: , qos: , attributes: , autoreleaseFrequency: , target: )

let serialQueue = DispatchQueue(label: "com.myapp.database")
// attributes에 아무것도 안 넣으면 Serial 이 기본값

let concurrentQueue = DispatchQueue(label: "com.myapp.cache", attributes: .concurrent)
// .concurrent를 명시해야 Concurrent Queue
```

#### 1.7 QoS (Quality of Service)

| QoS              | 우선순위 | 용도                      |
| ---------------- | ---- | ----------------------- |
| .userInteractive | 0    | 애니메이션, 즉각 응답 필요         |
| .userInitiated   | 1    | 사용자가 시작한 작업 (버튼 탭 후 결과) |
| .default         | 2    | 기본 값                    |
| .utility         | 3    | 프로그레스바 있는 긴 작업          |
| .background      | 4    | 사용자가 모르는 작업 (백업, 동기화)   |

#### 1.8 UI는 왜 메인 스레드에서 업데이트 해야하나?

> UIKit classes should be used only from an app's main thread or main dispatch queue.

UIKit의 렌더링 시스템은 스레드 안전하지 않다. 메인 스레드가 Run Loop의 업데이트 사이클에서 화면을 갱신하는데, 다른 스레드에서 UI를 건드리면 이 사이클의 타이밍과 어긋나서 화면이 깨지거나 크래시가 발생한다.

```
Main Run Loop 사이클:
1. 이벤트 수신 (터치, 타이머)
2. 이벤트 헨들러 실행
3. UI 업데이트
4. 화면 렌더링
5. 다음 사이클 대기
   
→ 전부 메인 스레드에서 돌아간다!
```

---
## ❓ 스스로에게 물어봐

Q1: Serial Queue와 Concurrent Queue의 차이는?

>[!faq]- 답
>Serial Queue는 한번에 하나의 작업만 실행하여 순서가 보장된다.
>Concurrent Queue는 병렬로 여러 작업을 동시에 실행하여 빠르지만 순서는 보장되지 않는다.
>DispatchQueue.main은 Serial Queue이고, DispatchQueue.global은 Concurrent Queue이다.
>커스텀 큐는 Serial이 기본이며 .concurrent 로 설정할 수도 있다.

Q2. Sync와 Async의 차이는?

>[!faq]- 답
>sync는 작업이 완료될 때까지 호출 스레드를 블로킹하고
>async는 작업을 큐에 넣고 즉시 반환하여 호출 스레드가 다른 일을 계속 할 수 있다.

Q3. 메인 스레드에서 UI를 업데이트하는 이유는?

>[!faq]- 답
>UIKit은 thread-safe하지 않다. 메인 스레드의 run loop의 업데이트 사이클에서 화면을 갱신하는데, 여러 스레드가 동시에 UI를 변경하면 메모리 손상, 렌더링 깨짐, 크래시 발생한다.

Q4. 다음 코드에서 A, B, C의 출력 순서는?

```swift
print("A")
DispatchQueue.global().async { print("B") }
print("C")
```

>[!faq]- 답
>A → C → B
>async 이므로 B는 큐에 넣고 바로 C를 실행
