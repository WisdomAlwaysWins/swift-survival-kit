**Q1.** 동기(sync)는 작업이 끝날 때까지 기다리고, 비동기(async)는 기다리지 않는다. (O/X)

>→ O

**Q2.** DispatchQueue.global()은 Serial Queue이다. (O/X)

>→ X
>concurrent queue로 여러 작업을 동시에 실행한다. serial queue는 DispatchQueue.main

**Q3.** 커스텀 큐 `DispatchQueue(label: "test")`의 기본값은 Concurrent이다. (O/X)

>→ X
>기본값은 serial이고, attributes를 .concurrent로 명시해야한다.

**Q4.** QoS `.background`는 `.userInitiated`보다 우선순위가 높다. (O/X)

>→ X
>userInteractive > userInit > default > utility > background

**Q5.** 비동기(async)로 작업을 보내면 메인 스레드가 멈추지 않는다. (O/X)

>→ O
>async는 작업을 큐에 넣고 즉시 반환하므로 메인 스레드가 멈추지 않고 다른 작업을 스케줄링한다.

---

**Q6.** 출력 순서는?

```swift
print("A")
DispatchQueue.global().async { print("B") }
print("C")
```

>→ A C B

**Q7.** 출력 순서는?

```swift
let serial = DispatchQueue(label: "test")
print("1")
serial.async { print("2") }
serial.async { print("3") }
print("4")
```

>→  1 4 2 3
>async라서 2, 3이 나중에 나오고 serial이라서 2, 3의 순서가 보장된다

**Q8.** 출력 순서는?

```swift
let serial = DispatchQueue(label: "test")
print("A")
serial.sync { print("B") }
print("C")
serial.async { print("D") }
print("E")
```

>→ A B C E D
>sync는 기다리니까 A 다음 B가 오고 async는 안기다리니까 C 다음 E

---

**Q9.** 데드락이 발생하는가?

```swift
DispatchQueue.main.async {
    print("Hello")
}
```

>발생하지 않음
>async는 기다리지 않음.ᐟ.ᐟ

**Q10.** 데드락이 발생하는가?

```swift
// 메인 스레드에서 실행
DispatchQueue.main.sync {
    print("Hello")
}
```

>발생한다. 
>메인 스레드에서 main.sync → 같은 serial queue의 뭉텅이 안에서 sync

**Q11.** 데드락이 발생하는가?

```swift
DispatchQueue.global().async {
    DispatchQueue.main.sync {
        print("Hello")
    }
}
```

>발생하지 않는다.
>기다리는 놈과 실행하는 놈이 다른 스레드이고 메인큐는 비어있어서 바로 실행하고 백그라운드가 결과를 받아 계속 진행 가능.

**Q12.** 데드락이 발생하는가?

```swift
let serial = DispatchQueue(label: "test")
serial.async {
    serial.sync {
        print("Hello")
    }
}
```

>발생한다.
>같은 serial queue 안에서 sync 걸었음

---
**Q13.** DispatchGroup에서 `enter()` 3번, `leave()` 3번 호출하면 `notify`가 실행된다. (O/X)

>O
>enter과 leave가 짝을 이루면 실행됨.

**Q14.** DispatchSemaphore(value: 0)에서 wait()을 먼저 호출하면 영원히 블로킹된다. (O/X)

>O
>value가 0 이니까 -1을 할 수 없다.

**Q15.** 다음 코드에서 "완료"가 출력되는가?

```swift
let group = DispatchGroup()

group.enter()
DispatchQueue.global().async {
    sleep(1)
    // leave() 호출을 깜빡함!
}

group.notify(queue: .main) {
    print("완료")
}
```

>완료가 출력되지 않음. enter와 leave가 짝을 이루지 못해서.

---

**Q16.** `Task.cancel()`을 호출하면 작업이 즉시 중단된다. (O/X)

>X
>Task.cancel()은 취소 신호만 보내는거지, 작업이 즉시 멈추는 것은 아니다.

**Q17.** `await` 지점에서 해당 스레드는 블로킹된다. (O/X)

>X

**Q18.** 다음 코드의 총 실행 시간은? (fetchA는 2초, fetchB는 3초)

```swift
let a = try await fetchA()
let b = try await fetchB()
```

>약 5초

**Q19.** 다음 코드의 총 실행 시간은? (fetchA는 2초, fetchB는 3초)

```swift
async let a = fetchA()
async let b = fetchB()
let result = try await (a, b)
```

>약 3초

**Q20.** async let은 구조적 동시성이고, Task { }는 비구조적 동시성이다. (O/X)

>X

---

**Q21.** Actor 외부에서 Actor의 var 프로퍼티에 await 없이 접근할 수 있다. (O/X)

>X
>Actor가 데이터 경쟁을 컴파일 타임에 방지하는 방법!

**Q22.** struct는 기본적으로 Sendable이다. (O/X)

>O
>struct는 값 타입이라 전달될 때 복사된다. 공유 상태가 없으니까 기본적으로 Sendable이다

**Q23.** var 프로퍼티가 있는 class도 Sendable로 선언할 수 있다. (O/X)

>X 
>class가 Sendable이려면 **final + 모든 프로퍼티가 let**이어야 한다. var가 있으면 여러 스레드에서 동시에 수정할 수 있으니까 안전하지 않다.

**Q24.** @MainActor를 붙인 클래스의 메서드는 백그라운드 스레드에서 실행될 수 있다. (O/X)

>X
>@MainActor를 **클래스에** 붙이면, 그 클래스의 모든 메서드/프로퍼티가 메인 스레드에서 실행된다. 백그라운드에서 실행될 수 없다. `nonisolated`를 명시적으로 붙인 메서드만 예외.

**Q25.** Actor는 내부적으로 Concurrent Queue처럼 동작한다. (O/X)

>X
>Actor는 **Serial Queue**처럼 동작한다. 한 번에 하나의 접근만 허용해서 데이터 경쟁을 방지한다. Concurrent였으면 동시 접근을 허용하니까 보호가 안 된다.


---

**Q26.** 이 코드에 문제가 있는가? 있다면 뭔가?

```swift
DispatchQueue.global().async {
    let image = self.downloadImage()
    self.imageView.image = image
}
```

>UI 업데이트를 메인 스레드가 아닌 곳에서 실행 중

**Q27.** 이 코드에 문제가 있는가? 있다면 뭔가?

```swift
var count = 0

DispatchQueue.global().async { count += 1 }
DispatchQueue.global().async { count += 1 }
DispatchQueue.global().async { count += 1 }

print(count)
```

>여러 스레드에서 count에 접근해서 수정 중

**Q28.** 이 코드에서 `[weak self]`가 필요한 곳과 불필요한 곳을 구분하세요.

```swift
class MyVC: UIViewController {
    func doWork() {
        // A
        [1,2,3].forEach { num in
            self.process(num)
        }
        
        // B
        URLSession.shared.dataTask(with: url) { data, _, _ in
            self.handleData(data)
        }
        
        // C
        UIView.animate(withDuration: 0.3) {
            self.view.alpha = 0
        }
        
        // D
        Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
            self.tick()
        }
    }
}
```

>B, D

---

**Q29.** 출력 순서는?

```swift
print("1")
DispatchQueue.global().async {
    print("2")
    DispatchQueue.main.async {
        print("3")
    }
    print("4")
}
print("5")
```

>1 5 2 4 3

**Q30.** 출력 순서는?

```swift
let serial = DispatchQueue(label: "test")

serial.async {
    print("A")
    sleep(2)
    print("B")
}

serial.async {
    print("C")
}

print("D")
```

>D A B C 

**Q31.** 출력 순서는? (주의: concurrent)

```swift
let concurrent = DispatchQueue(label: "test", attributes: .concurrent)

concurrent.async { print("A"); sleep(2); print("B") }
concurrent.async { print("C"); sleep(1); print("D") }
concurrent.async { print("E") }

print("F")
```

>F가 먼저고, ACE의 순서는 보장되지 않는다.

---
**Q32.** 데드락이 발생하는가?

```swift
let serialA = DispatchQueue(label: "A")
let serialB = DispatchQueue(label: "B")

serialA.async {
    serialB.sync {
        print("Hello")
    }
}
```

>발생하지 않음

**Q33.** 데드락이 발생하는가?

```swift
DispatchQueue.main.async {
    DispatchQueue.main.sync {
        print("Hello")
    }
}
```

>발생

**Q34.** 데드락이 발생하는가?

```swift
DispatchQueue.global().sync {
    print("Hello")
}
```

>발생하지 않음

---
**Q35.** 이 코드에 문제가 있는가?

```swift
func fetchData() async throws -> Data {
    let data = try await URLSession.shared.data(from: url)
    return data.0
}

// 호출
let data = try await fetchData()  // viewDidLoad 안에서 직접 호출
```

>Task 블록 필요

**Q36.** 총 실행 시간은? (각 fetch는 1초)

```swift
func loadAll() async throws {
    async let a = fetch1()
    let b = try await fetch2()
    async let c = fetch3()
    let results = try await (a, c)
}
```

>2초

**Q37.** 이 코드에서 Task 취소가 제대로 동작하는가?

```swift
let task = Task {
    for i in 0..<1000 {
        print(i)
        try await Task.sleep(nanoseconds: 100_000_000)
    }
}

task.cancel()
```

>동작한다. 
>try await이 있는 suspension point에서 취소가 감지 된다.

**Q38.** 이 코드에서 Task 취소가 제대로 동작하는가?

swift

```swift
let task = Task {
    for i in 0..<1000 {
        print(i)
        Thread.sleep(forTimeInterval: 0.1)
    }
}

task.cancel()
```

>아니오. 
>코드 안에서 취소를 확인할 수 있는 지점이 없다.

---
**Q39.** 이 코드는 컴파일되는가?

```swift
actor Counter {
    var count = 0
    
    func increment() {
        count += 1
    }
}

let counter = Counter()
counter.increment()
```

>X
>await로 호출해야함

**Q40.** 이 코드는 컴파일되는가?

```swift
actor Counter {
    var count = 0
    
    func increment() {
        count += 1
    }
    
    func doubleIncrement() {
        increment()
        increment()
    }
}
```

>O
>내부는 직렬로 동작

**Q41.** 이 코드의 문제는?

```swift
actor BankAccount {
    var balance = 1000
    
    func transfer(to other: BankAccount, amount: Int) async {
        guard balance >= amount else { return }
        balance -= amount
        await other.deposit(amount)
    }
    
    func deposit(_ amount: Int) {
        balance += amount
    }
}
```

>balance -= amount와 await other.deposit(amount) 사이에 다른 Task가 balance를 변경한다면 ..

---
**Q42.** [weak self] 필요한가?

```swift
Task { [weak self] in
    let data = try await fetchData()
    self?.updateUI(data)
}
```

>→ 권장
>중간에 화면을 나가면 fetchData를 중단할 수 있음

**Q43.** [weak self] 필요한가?

```swift
class ViewModel {
    var onUpdate: (() -> Void)?
    
    func setup() {
        onUpdate = {
            print(self.name)
        }
    }
}
```

>클로저가 캡처되므로 필요함

**Q44.** [weak self] 필요한가?

```swift
NotificationCenter.default.addObserver(
    forName: .someNotification,
    object: nil,
    queue: .main
) { _ in
    self.refresh()
}
```

>클로저방식이라서 self를 강하게 잡고 있으면 vc가 해제되지 않아서 필요함

---
**Q45.** 이 코드의 문제점을 **모두** 찾으세요 (3개 이상).

```swift
class ImageListVC: UIViewController {
    var images: [UIImage] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let urls = getImageURLs()
        
        for url in urls {
            DispatchQueue.global().async {
                let data = try! Data(contentsOf: url)
                let image = UIImage(data: data)!
                self.images.append(image)
                self.collectionView.reloadData()
            }
        }
    }
}
```

>1. serial queue인 viewDidLoad() 메서드에서 Task 없이 바로 비동기 작업
>2. UI 업데이트를 메인 스레드가 아닌 곳에서 진행
>3. 백그라운드에서 reloadData()

**Q46.** 위의 코드를 async/await으로 올바르게 재작성하세요.

```swift
class ImageListVC: UIViewController {
    var images: [UIImage] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()

        Task {
	        let urls = getImageURLs()
	        
	        let downloadedImages = await withTaskGroup(of UIImage?.self) { group in 
				for url in urls {
		            group.addTask {
			            guard let (data, _) = try? await URLSession.shared.data(from: url),
				            let image = UIImage(data: data) else {
					        return nil    
						}
						return image
		            }
	            }
	            var results: [UIImages] = []
	            for await image in group {
		            if let image {
			            results.append(image)
		            }
	            }
	            return results
	        }
	        images = downloadImages
	        collectionView.reloadData()
        }
    }
}
```

