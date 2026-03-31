#uikit #tableview

>[!info] TableView, 왜 중요한가?
>iOS 앱에서 목록 화면은 거의 100% TableView 또는 CollectionView로 만든다. 카카오톡 채팅 목록, 설정 앱, 연락처 — 전부 TableView다.

## 1. UITableView란

Apple 공식 문서에서는 이렇게 정의한다:

![[UITableView.png]]

```
┌──────────────────────┐
│  채팅방 1            │  ← Row 0 (셀)
├──────────────────────┤
│  채팅방 2            │  ← Row 1 (셀)
├──────────────────────┤
│  채팅방 3            │  ← Row 2 (셀)
├──────────────────────┤
│  채팅방 4            │  ← Row 3 (셀)
├──────────────────────┤
│         ...          │
└──────────────────────┘
        ↕ 스크롤
```

각 행에는 앱 콘텐츠의 일부분이 포함된다. 연락처 앱은 각 연락처의 이름을 별도의 행에 표시하고, 설정 앱은 사용 가능한 설정 그룹을 행으로 표시한다. 하나의 긴 목록으로 구성할 수도 있고, 관련 행을 섹션(Section) 형태로 그룹화할 수도 있다.

일반으로 UINavigationController와 함께 사용한다. 테이블에서 행을 탭하면 상세 화면으로 push 하는 계층적 탐색이 가장 흔한 패텬이다.

---
## 2. TableView 생성하기

##### [Creating a table view](https://developer.apple.com/documentation/uikit/uitableview#Creating-a-table-view)
[`init(frame: CGRect, style: UITableView.Style)`](https://developer.apple.com/documentation/uikit/uitableview/init\(frame:style:\))
Creates and returns a table view with the specified frame and style.

[`init?(coder: NSCoder)`](https://developer.apple.com/documentation/uikit/uitableview/init\(coder:\))
Creates a table view object from data in an unarchiver.

frame과 style을 지정해서 만든다. frame은 보통 Auto Layout으로 잡으니 패스. 그러면 style은 뭘까?

- .plain : 가장 기본적인 스타일로 스크롤 시 section header가 상단에 고정된다
- .grouped : 각 섹션이 고유한 행들의 그룹으로 구분되는 스타일 (설정 앱)
- .insetGrouped : grouoped랑 비슷하지만, 섹션이 둥근 모서리로 처리된 스타일

```swift
// 생성 예시
let tableView = UITableView(frame: .zero, style: .plain)

// 또는
let settingsTable = UITableView(frame: .zero, style: .insetGrouped)
```

---
## 3. DataSource — "데이터를 줘"

TableView를 만들려면 딱 2가지 질문에 답해야 한다. 이 2가지가 UITableViewDataSource 프로토콜의 필수 메서드다!

1. **`tableView(_:numberOfRowsInSection:)`** : 각 섹션의 몇 개의 행이 있는지 리턴한다.
2. **`tableView(_:cellForRowAt:)`** : 특정 위치에 표시할 셀을 만들어서 리턴한다.

```swift
class FruitListVC: UIViewController, UITableViewDataSource {
    
    let tableView = UITableView()
    let fruits = ["사과", "바나나", "체리", "딸기", "포도"]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        tableView.dataSource = self  // "데이터는 내가 줄게"
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        
        view.addSubview(tableView)
        tableView.frame = view.bounds
    }
    
    // 질문 1: "몇 개야?"
    func tableView(_ tableView: UITableView,
                    numberOfRowsInSection section: Int) -> Int {
        return fruits.count  // 5개
    }
    
    // 질문 2: "각 칸에 뭘 보여줄 거야?"
    func tableView(_ tableView: UITableView,
                    cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = fruits[indexPath.row]
        return cell
    }
}
```

>[!abstract] `indexPath`가 뭔데? 
>**몇 번째 섹션의 몇 번째 줄**인지를 나타내는 값이다. `indexPath.section` = 섹션 번호 (0부터) `indexPath.row` = 줄 번호 (0부터) 섹션이 1개뿐이면 `indexPath.row`만 쓰면 된다

---
## 4. register + dequeueReuseableCell

- **`register(_:forCellReuseIdentifier:)`** : 이 identifier로 이 셀의 클래스를 쓸 것이라고 미리 등록하는 것이다. viewDidLoad에서 1번만 호출한다.

```swift
tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
//                  ^^^^^^^^^^^^^^^^                               ^^^^
//                  어떤 클래스로 만들지                           이름표
```

- **`dequeueReusableCell(withIdentifier:for:)`** : 재사용 큐에서 특정 셀을 하나 꺼내달라는 요청

---
## 5. Delegate — "이벤트를 알려줘"

- didSelectRowAt : 탭했을 때
- heightForRowAt : 셀 높이
- 스와이프 액션 

---
## 6. 셀 재사용 원리

화면에 보이는 셀만 메모리에 만들고, 스크롤로 화면 밖으로 나간 셀을 **재사용 큐(reuse queue)** 에 넣어서 다시 쓴다. 1만 개의 데이터가 있어도 화면에 보이는 10~15개의 셀만 메모리에 존재한다.

```
화면에 보이는 셀: [0] [1] [2] [3] [4]

                 ↓ 스크롤
                 
재사용 큐: [0] ←── 화면 밖으로 나간 셀이 큐에 들어감

화면에 보이는 셀: [1] [2] [3] [4] [5]
                              ↑
            큐에서 꺼낸 셀을 [5]의 데이터로 설정
```

#### 6.1 prepareForReuse

셀이 재사용 큐에서 꺼내질 때 호출된다. 이전 데이터의 흔적을 **초기화**하는 곳이다. 이걸 안 하면 스크롤 시 이전 셀의 이미지, 체크 마크 등이 남아있는 버그가 생긴다.

```swift
class ProductCell: UITableViewCell {
    let productImage = UIImageView()
    let nameLabel = UILabel()
    
    override func prepareForReuse() {
        super.prepareForReuse()
        productImage.image = nil          // 이전 이미지 제거!
        nameLabel.text = nil              // 이전 텍스트 제거
        productImage.cancelImageLoad()    // 비동기 이미지 로딩 취소!
    }
}
```

