# 🇰🇷 만국박람회

1. 프로젝트 기간: 2021.12.06 - 2021.12.17

<br>

## 🗂 목차
- [📱 구현 화면](#-구현-화면)
- [📃 구현 내용](#-구현-내용)
- [🚀 Trouble Shooting](#-Trouble-Shooting)
- [🤔 고민한 점](#-고민한-점)
- [⌨️ 키워드](#-키워드)


<br>

## 📱 구현 화면

| Text Size| 메인화면 | 출품작 리스트 | 출품작 상세화면 |
| :---: | :---: | :---: | :---: |
| 기본 | ![image](https://user-images.githubusercontent.com/90945013/162420106-c05e11db-c4f3-42a1-932e-d485100baa11.png) | ![image](https://user-images.githubusercontent.com/90945013/162420145-d8b65d7e-b554-4685-af2d-371a887e3bde.png) | ![image](https://user-images.githubusercontent.com/90945013/162420164-524c47db-599f-4910-9538-19834655ce0c.png) |
| Dynamic Types 적용 | ![image](https://user-images.githubusercontent.com/90945013/162420640-be68f894-2476-46ac-af9e-9fd822e80def.png) | ![image](https://user-images.githubusercontent.com/90945013/162420662-57a664d5-9e02-4a31-875d-4bde01053a9e.png) | ![image](https://user-images.githubusercontent.com/90945013/162420681-c742bf2c-6cb7-428e-b86c-f86acf53a240.png) |

<br>

## 📃 구현 내용

### MVC + UIKit
- MVC 아키텍처를 사용하여 설계
- UIKit과 StoryBoard 통해 뷰를 구현

### JSON
- Codable을 채택하여 JSON 데이터와 매칭할 모델 타입 구현
- CodingKey 프로토콜을 사용하여 스네이크 케이스 또는 축약형인 JSON 키 값을 스위프트의 네이밍에 맞게 변환
- JSON 파싱이 잘 되고 있는지에 대한 테스트 코드 작성

### VIew
- 뷰를 구성하는 과정에서의 성능 및 가독성을 높이기 위해 하나의 화면마다 하나의 StroyBoard로 분리
- TableView의 Delegate와 DataSource의 역할에 대한 이해를 바탕으로 적절하게 구현
- Navigation Controller를 활용하여 화면전환하도록 구현
- performSegue와 prepare매서드를 통해 ViewController 사이의 데이터를 전달하도록 구현
- AutoLayout을 적용하여 다양한 기기에서 지원할 수 있도록 구현
- Label, Text에 대한 Word Wrapping / Line Wrapping / Line Break 방식의 이해를 바탕으로 사용자가 보기에 편하도록 구현

### Accessibility
- 접근성(Accessibility)의 개념과 필요성을 인지하고 다양한 사용자가 사용할 수 있도록 구현
- Dynamic Types를 통해 다양한 사용자의 텍스트 접근성 향상

<br>

## 🚀 Trouble Shooting


### 1. DataSource에서 Alert 띄우기
#### 문제점
DataSource에서 리스트를 가져오지 못하거나 가져온 데이터의 count가 0이어서 보여줄 것이 없을 때 얼럿을 띄워주고 싶었으나  DataSource에서는 Alert을 띄울 수 없었다.

#### 원인
Alert을 만들고 present를 하는 과정에서 UIViewController가 아니면 present를 할 수가 없었다.

#### 해결방안
Delegate 패턴을 사용하여 DataSource에서 빈셀을 반환하게 될 경우 UITableViewController에서 대신 Alert을 처리할 수 있도록 구현하였다.

showAlert 메서드를 UIViewController의 Extension에서 구현 하였다.
Delegate프로토콜인 AlertDelegate프로토콜을 사용하여 UIViewController의 Extension에서 구현해 놓은 showAlert 메서드를 요구사항으로 정의하였다.

```swift
// Alert.swift
extension UIViewController {
    func showAlert(alertMessage: AlertMessage, buttonMessage: AlertMessage) {
	// …
    }
}

// EntryDataSource.swift
protocol AlertDelegate: UIViewController {
    func showAlert(alertMessage: AlertMessage, buttonMessage: AlertMessage)
}
```

### 2. Delegate 패턴 사용 시 메모리 누수
#### 문제점
- Delegate 패턴을 사용할 때 메모리 누수가 발생한 것을 확인 할 수 있었다.
<p align=center><img src="https://user-images.githubusercontent.com/20634806/145971091-40e97313-71cf-4f72-aa5d-a66644d51d04.png" width=300/></p>
<p align=center><img src="https://user-images.githubusercontent.com/20634806/145971141-99e28ace-ac95-445f-bac1-62b49e10d3d2.png" width=300/></p>
#### 원인
- Delegate 패턴을 사용할 때 두 객체가 서로를 강한참조하고 있어 강한참조순환이 발생해 메모리 누수가 발생하였다.

#### 해결방안
- delegate를 weak으로 선언하여 강한순환참조를 해결하였다.
- delegate 패턴을 사용할 때 메모리누수에 대해서 주의해야한다.

```swift
weak var delegate: AlertDelegate?
```

### 3. Hugging vs Compression Resistance Priority
#### 문제점
- 아래의 사진처럼 "방문객:" label과 "48,130,330명" label 사이에 공백이 발생했다.
![](https://user-images.githubusercontent.com/71127966/146341703-3496f4b9-af8d-4d8c-b2a3-c413a8d9b6b5.png)

#### 원인
- 내부의 우선순위 연산의 의한 결과값으로 생각이 된다.   

#### 해결방안
- 여러 숫자를 대입해 보았을 떄 950이라는 숫자가 되어야 문제를 해결할 수 있었다.   
- 보통 250, 750, 1000 이렇게 기본적으로 구분을 많이 하고 있다고 알고 있다.
- 근데 사실 1000은 내부적으로 950만 되어도 가능한 것은 아닐까? 추측을 해본다.

![](https://user-images.githubusercontent.com/71127966/146341912-5af01317-6bdf-44cd-9504-a0bf2c67deed.png)

### 4. 날짜(기간)에 대한 AcessibilityLabel 설정법
#### 문제점
- VoiceOver를 설정하는 도중 날짜에 대한 부분을 어색하게 문제가 발생했다.
- 개최 기간 부분의 날짜를 "천구백년 사월 십사일 일구공공 일일 일이"로 읽어 준다.
![](https://user-images.githubusercontent.com/71127966/146344534-01371eb7-8dd6-45b4-a790-6e62180d3382.png)

#### 원인
- AcessibilityLabel를 설정하지 않았더니 label에 적혀있는대로 1900. 04. 14 - 1900. 11. 12를 읽어서 "천구백년 사월 십사일 일구공공 일일 일이"로 읽게 되었다.

#### 해결방안
- AcessibilityLabel를 좀 더 친절하게 날짜와 한글을 함께 적어주었다.
> 1900.04.14부터 1900.11.12까지

![](https://user-images.githubusercontent.com/71127966/146344534-01371eb7-8dd6-45b4-a790-6e62180d3382.png)

<br>

## 🤔 고민한 점

### 1. 에셋 파일 추가하는 커밋의 스타일은 docs가 괜찮을까?
- 이것은 같이 개발하는 개발자들과 맞추는 것이 맞다고 생각하여 팀원과 상의 후에 커밋스타일을 docs로 정하여 사용하기로 했다.

### 2. CodingKey를 사용해야 할까?
- JSON 파일에 명시된 키와 프로젝트 내부에서 사용할 프로퍼티명이 동일한 경우에는 CodingKey를 사용하지 않아도 되어서 CodingKey를 채택한 enum을 만들어주지 않았다.
- JSON 파일에 명시된 키와 프로젝트 내부에서 사용할 프로퍼티명이 동일 하지 않은 경우는 카멜케이스와 스네이크 케이스를 오가는 것이면 keyDecodingStrategy, keyEncodingStrategy로 해결하려고 하였지만 축약어가 있어 이것은 [Swift API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/)에서 지향하는 네이밍에 맞지 않아 CodingKeys 프로토콜을 사용하여 네이밍을 바꾸었다.

### 3. 에셋에서 @1x, @2x, @3x 스케일을 나타내는 것들이 있는데, 스케일에 따른 이미지가 필요한 이유는 무엇일까?

아이폰 기기 해상도에 따라 픽셀 밀도가 다르다.

- 아이폰 4 이전 -> 일반 디스플레이
- 아이폰 4 ~ 5 -> 레티나 디스플레이
- 아이폰 6 ~ 7 -> 레티나 HD 디스플레이

표준 해상도 이미지 배율은 1.0 이며 @1x 이미지 라고 부른다. ( standard-resolution display 에서는 1px == 1pt )
만약 이미지가 이것만 존재한다면 고해상도 디스플레이에 대응하기 위해 업스케일링을 거쳐야 한다.
이런 일이 빈번해진다면, 앱의 성능과 품질에 영향을 줄 수 있다.

그래서 애플은 다양한 해상도에 효과적으로 대응하기 위해, 1x, 2x, 3x 스케일에 따른 이미지를 요구하며
사용자가 앱스토어에서 앱을 다운받을 때는, 사용자의 기기에 맞는 이미지만 다운로드 한다.

<p align=center><img src = "https://user-images.githubusercontent.com/71127966/145138567-177504a5-93b8-4fb6-bcc2-a914ef7f4997.png" width = 300 /></p>
[이미치 참조: H.I.G 문서](https://developer.apple.com/design/human-interface-guidelines/ios/icons-and-images/image-size-and-resolution/)

### 4. JSONParser를 enum으로 선언한 이유
- 처음에 struct로 할지 enum으로 할지 고민을 했다.   
- struct를 사용하게 되면 몇개월 뒤의 나 또는 다른 개발자가 실수로 인스턴스를 생성할 수도 있다고 생각하여
실수를 미연에 방지하고자 인스턴스를 생성할 수 없는 case없는 enum으로 선언하였습니다.

### 5. 오류처리
JSON파일을 불러오지 못했을 때 어떻게 처리를 해야할지 고민을 많이 했다.
처음에는 Error 열거형 타입을 사용하여 오류처리를 하려고 하였다.

하지만 JSON을 파싱할 때 마다 do-try-catch를 사용하게 되면 코드가 가독성이 떨어지기도 하고 결정적으로 prepare메서드는 override 메서드이기 때문에 오류를 throws 할 수 없었고 tableView 메서드도 UITableViewDataSource 프로토콜에 정의되어 있지 않아서 오류를 throws 할 수 없었다.
그래서 오류 처리를 Alert을 발생시키고 빈셀을 반환하는 방식으로 구현했다.

### 6. 프로토콜을 사용할 수 있는 타입 제한
Alert을 띄우기 위한 present메서드는 UIViewController 타입에서만 사용할 수 있기 때문에
프로토콜을 구현할 때 UIViewController만 사용할 수 있도록 아래와 같이 코드를 작성하였다.
클래스만 채택할 수 있게 AnyObject를 사용할 수 있도록 하는 것을 한번 더 생각하여 UIViewController를 쓰면 UIViewController타입을 상속하는 모든 타입들만 채택할 수 있게 만들었다.

```swift 
protocol AlertDelegate: UIViewController {
    func showAlert(alertMessage: AlertMessage, buttonMessage: AlertMessage)
}
```

### 7. 네임스페이스 관리
네임스페이스를 별도 파일로 관리하는 것이 좋다고 생각하여 JSONFileName과 AlertMessage를 만들어 관리하도록 했다.
identifier의 경우는 cell의 identifier도 있고 seuge의 identifier 등 여러 UI요소에 identifier가 있어 쓰임새를 나눠서 관리할지 고민 하다가 결국 각 클래스에 static let으로 정의했었다.

그런데 리뷰어의 피드백을 받고 나서 더 좋은 방법이 있다는 것을 알게 되었다.  
reuse identifier를 클래스명으로 지정하고 나서 extension이나 프로토콜+기본구현을 통해 reuse identifier를 제공할 수 있었다.

그래서 UITableViewCell extension을 통해 UITableViewCell에서는 resuse identifier를 편하게 관리할 수 있도록 구현하였다.

```swift
extension UITableViewCell {
    static var reuseIdentifier: String {
        return String(describing: self)
    }
}
```

### 8. 스토리보드 분리
스토리보드를 사용하여 UI를 그릴 떄에는 하나의 뷰컨에 하나의 스토리보드를 사용하도록 구현했다. 
왜냐하면 하나의 스토리보드 안에 여러개의 뷰컨이 있으면 첫째로 스토리보드를 조작할 떄 느려지는 현상이 있었다.    
또한, 스토리보드를 나누어 작업하게 되면 더 보기가 쉽고 깔끔해진다.


<br>

## ⌨️ 키워드
- `UITableView`
	- `UITableView Delegate`, `UITableViewDataSource`
- `JSON`
	- `CodingKey`, `Codable`, `Encodable`, `Decodable`, `JSONDecoder`, `JSONEncoder`
- `AssetData`
- `Delegate Pattern`
- `Navigation Controller`
- `Acessibility`
	- `Dynamic Types`, `VoiceOver`, `AcessibilityLabel`
- `AutoLayout`
	- `Constraints`, `Priority`, `Content hugging`, `Compression resistance`
- `Word Wrapping`, `Line Wrapping`, `Line Break `
