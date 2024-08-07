# Favorite Shopping - 나만의 쇼핑 앱

<p align="center">
  <img src="https://github.com/constdreamcoder/SaSACRecap/assets/95998675/3525e2b0-b3db-40cd-af49-f4ec06abf3c7" align="center" width="100%">
</p>

<br/>

## Favorite Shopping

- 서비스 소개: 원하는 상품을 검색할 수 있는 앱
- 개발 인원: 1인
- 개발 기간: 24.01.21 ~ 24.01.24(총 4일)
- 개발 환경

  - 최소버전: iOS 15
  - Portrait Orientation 지원
  - 라이트 모드 지원

  <br/>

## 💪 주요 기능

- 상품 검색 기능
- 선호 상품 찜 기능
- 프로필 캐릭터 선택 / 수정 기능

  <br/>

## 🛠 기술 소개

- SnapKit을 통한 Code-Based UI로 전환
- Kingfisher를 통한 외부 이미지 다운로드
- Alamofire를 통한 네트워크 코드 간소화
- Swift Concurrency를 활용한 직관적인 네트워크 코드 구성
- Local Notificaition를 통한 유저 알림 구성
- 네이버 쇼핑 API를 통한 상품 검색
- prefetch를 이용한 Cursor 기반 Pagination 구현

  <br/>

## 💻 구현 내용

### 1. SnapKit을 통한 Code-Based UI로 전환

<details>
<summary><b>StoryBoard UI<b></summary>
<div markdown="1">

<img src="https://github.com/constdreamcoder/SaSACRecap/assets/95998675/c4a7799f-e122-4502-a7a9-1589f534498e" align="center" width="100%">

</div>
</details>

<details>
<summary><b>SnapKit UI<b></summary>
<div markdown="1">

```swift
// 온보딩 화면에서 Snapkit으로 구성한 UI 코드
func configureConstraints() {
    appLogoImageViewContainer.addSubview(appLogoImageView)

    [
        appLogoImageViewContainer,
        onboardingImageView,
        startButton
    ].forEach { view.addSubview($0) }

    appLogoImageViewContainer.snp.makeConstraints {
        $0.top.horizontalEdges.equalTo(view.safeAreaLayoutGuide)
        $0.bottom.equalTo(onboardingImageView.snp.top)
    }

    appLogoImageView.snp.makeConstraints {
        $0.center.equalToSuperview()
        $0.width.equalToSuperview().multipliedBy(0.53)
        $0.height.equalTo(appLogoImageView.snp.width).multipliedBy(0.45)
    }

    onboardingImageView.snp.makeConstraints {
        $0.center.equalTo(view.safeAreaLayoutGuide)
        $0.width.equalTo(view.safeAreaLayoutGuide.snp.width).multipliedBy(0.9)
        $0.height.equalTo(onboardingImageView.snp.width)
    }

    startButton.snp.makeConstraints {
        $0.horizontalEdges.equalToSuperview().inset(24.0)
        $0.bottom.equalTo(view.safeAreaLayoutGuide.snp.bottom).offset(-16.0)
        $0.height.equalTo(50.0)
    }
}
```

</div>
</details>

<br/>

### 2. 닉네임 유효성 검사 에러 처리

<details>
<summary><b>닉네임 유효성 검사에 대한 커스텀 에러 및 에러 메세지 열거형</b></summary>
<div markdown="1">

```swift
enum NicknameValidationError: Error {
    case notInBetweenTwoAndTenLetters
    case containUnallowedSpecialCharacters
    case containNumbers

    var errorMessage: String {
        switch self {
        case .notInBetweenTwoAndTenLetters:
            return "2글자 이상 10글자 미만으로 설정해주세요"
        case .containUnallowedSpecialCharacters:
            return "닉네임에 @, #, $, %는 포함할 수 없어요"
        case .containNumbers:
            return "닉네임에 숫자는 포함할 수 없어요"
        }
    }

    var errorMessageTextColor: UIColor {
        return .red
    }
}

```

</div>
</details>

<details>
<summary><b>닉네임 유효성 검사 메서드 및 에러 메세지 표시 메서드</b></summary>
<div markdown="1">

```swift
// 닉네임 유효성 검사
private func validateNickname(text: String) throws {
    guard text.count >= 2 && text.count < 10
    else {
        throw NicknameValidationError.notInBetweenTwoAndTenLetters
    }

    guard !text.contains("@") && !text.contains("#")
                && !text.contains("$") && !text.contains("%")
    else {
        throw NicknameValidationError.containUnallowedSpecialCharacters
    }

    guard text.filter({ $0.isNumber }).count < 1
    else {
        throw NicknameValidationError.containNumbers
    }
}

// 에러 메세지 표시 정의
private func showErrorMessage(error: NicknameValidationError) {
    errorMessageLabel.text = error.errorMessage
    errorMessageLabel.textColor = error.errorMessageTextColor
}
```

</div>
</details>

<details>
<summary><b>유효성 검사 에러 타입에 따른 에러 메세지 표시 메서드</b></summary>
<div markdown="1">

```swift
// 유효성 검사 에러 타입에 따른 에러 메세지 표시
private func checkNicknameValidationAndShowErrorMessage(text: String) {
    do {
        try validateNickname(text: text)
        errorMessageLabel.text = "사용할 수 있는 닉네임이에요"
        errorMessageLabel.textColor = Colors.pointColor
    } catch {
        switch error {
        case NicknameValidationError.notInBetweenTwoAndTenLetters:
            print("2글자 이상 10글자 미만으로 설정해주세요")
            showErrorMessage(error: NicknameValidationError.notInBetweenTwoAndTenLetters)
        case NicknameValidationError.containUnallowedSpecialCharacters:
            print("닉네임에 @, #, $, %는 포함할 수 없어요")
            showErrorMessage(error: NicknameValidationError.containUnallowedSpecialCharacters)
        case NicknameValidationError.containNumbers:
            print("닉네임에 숫자는 포함할 수 없어요")
            showErrorMessage(error: NicknameValidationError.containNumbers)
        default:
            print("알 수 없는 오류입니다.")
        }
    }
}
```

</div>
</details>

<br/>

### 3. Swift Concurrency를 활용한 네트워크 코드 구성

<details>
<summary><b>Swift Concurrency 네트워크 코드</b></summary>
<div markdown="1">

```swift
// 네트워크 에러 정의
enum NetworkError: Error {
    case unknownError
}

final class ShoppingManager {

    static let shared = ShoppingManager()

    private init() {}

    // 상품 검색(Swift Concurrency 활용)
    func fetchShoppingResults(
        keyword: String,
        sortingStandard: SortingStandard = .byAccuracy,
        start: Int = 1,
        display: Int = 30
    ) async throws -> SearchResultsModel {
        if let query = keyword.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) {

            // 통신 Base URL 구성
            let urlString = "https://openapi.naver.com/v1/search/shop.json?query=\(query)&display=\(display)&sort=\(sortingStandard.rawValue)&start=\(start)"

            // Header 구성
            let headers: HTTPHeaders = [
                "X-Naver-Client-Id": APIKeys.clientID,
                "X-Naver-Client-Secret": APIKeys.clientSecret
            ]

            // 네트워크 요청을 위한 dataTask 구성
            let dataTask = AF.request(urlString, method: .get, headers: headers)
                .validate(statusCode: 200..<300)
                .serializingDecodable(SearchResultsModel.self)

            // 네트워크 요청 및 응답 수신
            let result = await dataTask.result

            // 응답 내용 반환
            switch result {
            case .success(let success):
                return success
            case .failure(let failure):
                throw failure
            }
        }
        throw NetworkError.unknownError
    }
}
```

</div>
</details>

<br/>

### 4. Local Notification 구성을 통한 쇼핑 목록 관리 알림 제공

<details>
<summary><b>Local Notification 구성</b></summary>
<div markdown="1">

```swift
func configureLocalNotifications() {
    let content = UNMutableNotificationContent()
    content.title = "쇼핑 리스트 관리!!"
    content.body = "쇼핑 리스트를 관리해주세요!"

    var dateComponents = DateComponents()
    dateComponents.hour = 15

    let trigger = UNCalendarNotificationTrigger(dateMatching: dateComponents, repeats: true)
    let request = UNNotificationRequest(identifier: "\(Date())", content: content, trigger: trigger)

    UNUserNotificationCenter.current().add(request) { error in
        if let error = error {
            print("Notification Error: ", error)
        }
    }
}
```

</div>
</details>

<details>
<summary><b>Local Notification</b></summary>
<div markdown="1">

<img src="https://github.com/constdreamcoder/SwiftPrac/assets/95998675/f3d1e5e5-9cb1-4136-b2b2-89572ec2a92a" align="center" width="200">

</div>
</details>

<br/>

## 🔥 트러블 슈팅

### 1. 리터럴한 형태의 UserDefaults Key

문제상황

- 리터럴한 형태로 UserDefaults Key 사용

    <details>
    <summary><b>리터럴한 형태로 사용된 UserDefaults Key</b></summary>
    <div markdown="1">

  ```swift
  override func viewWillAppear(_ animated: Bool) {
      super.viewWillAppear(animated)

      // profileImageView 이미지 업데이트
      let profileImageName = UserDefaults.standard.string(forKey: "CurrentProfileImageName") ?? "no_image"
      profileImageView.image = UIImage(named: profileImageName)

      // nicknameLabel 텍스트 업데이트
      nicknameLabel.text = UserDefaults.standard.string(forKey: "Nickname") ?? ""

      // heartCountLabel 텍스트 업데이트 및 AttributedString 적용
      guard let heartPressedList = UserDefaults.standard.dictionary(forKey: "HeartPressedList") else { return }

      // ...
  }
  ```

    </div>
    </details>

문제 원인 파악

- 반복적인 UserDefaults Key 작성으로 유지보수 시 휴먼 에러 발생 가능성 상승

해결방법

- 열거형을 이용하여 UserDefaults Key를 따로 정의 및 관리

    <details>
    <summary><b>열거형으로 정의된 UserDefaults Key</b></summary>
    <div markdown="1">

  ```swift
  enum UserDefaultsKeys: String, CaseIterable {
      case nickname = "Nickname"
      case userLoginState = "UserLoginState"
      case heartPressedList = "HeartPressedList"
      case recentKeywordList = "RecentKeywordList"
      case selectedProfileImageName = "SelectedProfileImageName"
      case currentProfileImageName = "CurrentProfileImageName"
  }
  ```

    </div>
    </details>

<br/>

### 2. HTML 태그 처리

문제상황

- 네이버 쇼핑 API 호출 시 응답에 HTML 태그가 포함된 문자열이 반환

   <img src="https://github.com/constdreamcoder/SwiftPrac/assets/95998675/725aedf8-f29b-4c6d-b3c3-ce9bba122122" align="center" width="200">

해결방법

- NSAttributedString을 활용하여 HTML 태그값 제거 및 HTML 인코딩된 문자열을 일반 텍스트로 변환하여 사용자가 쉽게 읽을 수 있도록 합니다.

    <details>
    <summary><b>열거형으로 정의된 UserDefaults Key</b></summary>
    <div markdown="1">

  ```swift
  var htmlTagEraser: String {
    // 문자열을 UTF-8 인코딩된 Data로 변환
    guard let data = self.data(using: .utf8) else { return self }

    // NSAttributedString의 초기화에 사용할 옵션 정의
    let options: [NSAttributedString.DocumentReadingOptionKey: Any] = [
        .documentType: NSAttributedString.DocumentType.html, // 문서 유형을 HTML로 설정
        .characterEncoding: String.Encoding.utf8.rawValue // 문자 인코딩을 UTF-8로 설정
    ]

    do {
        // NSAttributedString을 사용하여 HTML 데이터를 디코딩하여 일반 문자열로 변환
        let attributed = try NSAttributedString(data: data, options: options, documentAttributes: nil)
        return attributed.string // 변환된 문자열을 반환
    } catch {
        return self // 변환에 실패하면 원래 문자열을 반환
    }
  }
  ```

    </div>
    </details>
