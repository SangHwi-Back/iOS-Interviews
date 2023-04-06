# Async/Await

## Meet Async/Await

예를 들어 UIImage 에 fetchThumbnail 이라는 함수가 있다. 그리고 동기/비동기 방식을 동시에 지원한다.

동기 방식의 fetchThumbnail 함수를 사용하면 다음의 순서대로 함수가 실행된다. 이는 모두 하나의 스레드에서 실행되며, 이 스레드는 아래의 작업이 끝나기 전엔 다른 작업을 하지 못한다.

* fetchThumbnail 시작
* preparingThumbnail(of:) 실행
* fetchThumbnail 종료

하지만 비동기 방식의 fetchThumbnail 의 경우는 다음과 실행된다.

1. fetchThumbnail 시작
2. prepareThumbnail(of:completionHandler:) 실행
   1. 완료까지 기다림
   2. 완료를 기다리는 도중에 스레드로 다른 작업이 넘어오면 실행 가능
3. prepareThumbnail(of:completionHandler:) 종료. 스레드에 Notify.
4. fetchThumbnail 종료.

비동기 방식의 원리는 비동기 함수가 실행될 때 스레드를 잠시 Unblock 하여 다른 작업을 허용하는 것입니다. Unblock 은 비동기 함수가 끝날 때까지 지속됩니다.

지금까지의 Swift 코드를 생각해 보면서 아래의 상황을 가정해봅시다.

* UITableView 가 존재합니다. 셀에는 UIImageView 가 있습니다.
* UIImageView 는 서버에 있는 이미지를 사용합니다. 그러므로 ViewModel 에 thumbnail 이름을 전달하고 이미지를 받아오도록 요청할 것입니다.
* ViewModel 은 다음의 과정을 **순서대로** 진행하여 이미지를 반환할 것입니다.
  1. thumbnail 문자열로 URLRequest 객체를 생성합니다.
  2. dataTask(with:completion:) 메서드를 이용해서 데이터를 요청합니다.
  3. Data 객체를 받아오면 UIImage(data:) 초기화 메서드를 이용해 UIImage 로 초기화 합니다.
  4. prepareThumbnail(of:completionHandler:) 등 비동기로 실행되는 메소드를 이용해서 UIImage 를 전달합니다.
* 문제는 위의 4 과정 중 1,3 번은 빨라서 동기화 되어도 괜찮지만 2,4 번은 느려서 특별한 처리가 필요하다는 것입니다.

이 상황을 @escaping 메서드를 사용해서 구현해봅시다.

```swift
func fetchThumbnail(for id: String, completion: @escaping (UIImage?, Error?) -> Void) {
    let request = thumbnailURLRequest(for: id)
    let task = URLSession. shared.dataTask(with: request) { data, response, error in
    if let error = error {
        completion (nil, error)
    } else if (response as? HTTPURLResponse)?.statusCode != 200 {
        completion (nil, FetchError.badID)
    } else {
        guard let image = UIImage (data: data!) else {
            return
        }
        image.prepareThumbnail(of: CGSize(width: 40, height: 40)) { thumbnail in
            guard let thumbnail = thumbnail else {
                return
            }
            completion (thumbnail, nil)
        }
    }
    task.resume ()
}
```

여기서는 2 개의 문제가 있는데, 10번 줄과 14번 줄에 completion 메소드로 에러를 전달하지 않았다는 것입니다. 그리고 에러를 전달하는 코드를 넣는다고 해도 Swift Error Handling 에서 사용하는 throws 가 사용되지 않았기 때문에 효율적인 에러 핸들링 전략을 세울 수 없습니다.

코드 자체도 굉장히 길다는 점이 눈에 띄고 있습니다.

```swift
func fetchThumbnail(for id: String) async throws -> UIImage { // async/await when function or closure
    let request = thumbnailURLRequest(for: id)
    // system get control of function. caller may suspended also function does.
    let (data, response) = try await URLSession.shared.data(for: request) 
    guard (response as? HTTPURLResponse)?.statusCode == 200 else { 
        throw FetchError.badID 
    }
    let maybeImage = UIImage(data: data)
    guard let thumbnail = await maybeImage?.thumbnail else { 
        throw FetchError.badImage 
    }
    return thumbnail
}
```

async 는 await 가 불리는 영역을 표시할 때 사용합니다. await 는 비동기적으로 실행되며 오래 기다려야 하는 실행문에 사용해야 합니다.

위의 코드는 순차적으로 실행되지만, await 로 실행되는 함수가 실행되고 종료되는 동안 스레드는 Unblock 되어 있습니다. @escaping 함수를 쓰거나 dataTask 메소드를 쓰는 것처럼 다른 작업을 동시에 수행할 수 있는 것입니다. 그에 더해, 코드가 훨씬 간결해졌습니다.

여기서 궁금증이 생깁니다. Function 말고도 async 로 선언할 수 있는 것 있을까? 

```swift
extension UIImage { // async/await when property
    var thumbnail: UIImage? {
        get async { // throws 의 자리는 함수와 같다.
            let size = CGSize(width: 40, height: 40)
            return await self.byPreparingThumbnail(ofSize: size)
        }
    }
}
```

getter 에 async 키워드를 넣으면 클로저를 통해 await 함수를 호출할 수 있습니다. 즉, Property 를 사용해서도 async/await 사용이 가능합니다. 여기서 주의할 점은 **Read-Only Property 만 async 적용이 가능하다** 는 것이다.

async/await 는 Read-Only Property 뿐만 아니라 For/While/Repeat-Loop 에서도 사용 가능합니다.

```swift
for await id in staticImageIDsURL.lines {
    let thumbnail = await fetchThumbnail(for: id)
    collage.add(thumbnail)
}

let result = await collage.draw()
```

참고로 URL 에는 lines 프로퍼티가 있는데 URL 로부터 다운로드 받는 모든 line 들을 각각 async 하게 생성하는 sequence 로 만들어줍니다. 한 줄을 가져오는 작업이 async 하므로 await id 로 불러오는 것입니다.

어떻게 이런 일이 가능할까요? Function-Suspending 이 그 답입니다. 모든 작업은 Thread 의 관리 하에 이루어집니다. 하지만, Asynchronous 한 작업을 진행할 경우 Thread 는 Unblock 되어 다른 작업도 처리할 수 있어야 하기 때문에 외부의 간섭이 필요합니다. Function-Suspending 은 Thread 의 제어 권한을 System 으로 확대합니다. System 은 Thread Scheduling 을 통해 다른 작업도 Thread 가 수행하도록 도와줍니다.

await 가 호출되면 await 다음 줄까지 실행되는 시간 사이 Term 이 발생되며, 이 현상을 Suspending 이라고 명명하는 것입니다.

Async/Await 에 대해 다음과 같이 정리할 수 있습니다.

* `async` 는 함수가 suspend 될 수 있게 한다.
* `await` 는 async 함수가 실행을 suspend 할 것임을 명시한다.
* 다른 작업들이 위의 suspension 동안 실행될 수 있습니다.
* `await` 로 실행된 호출이 종료되면 다음 실행은 `await` 다음에 순차적으로 실행된다.

## Adopting Async Await

테스트에서 async await 는 활용도가 높습니다.

```swift
class MockViewModelSpec: XCTestCase {
    func testFetchThumbnails () throws {
        let expectation = XCTestExpectation(description: "mock thumbnails completion")
        self.mockViewModel.fetchThumbnail(for: mockID) { result, error in
            XCTAssertEqual(result?.size, CGSize (width: 40, height: 40))
            expectation.fulfill()
        }
        wait (for: [expectation], timeout: 5.0)
    }
}
```

많은 상황에서 사용되는 코드에 async/await 를 첨가해보도록 합니다. 아래 코드는 위의 코드와 동일한 기능을 합니다.

```swift
class MockViewModelSpec: XCTestCase {
    func testFetchThumbnails () async throws {
        let result = try await self.mockViewModel.fetchThumbnail(for: mockID)
        XCTAssertEqual(result.size, CGSize(width: 40, height: 40))
    }
}
```

아래의 SwiftUI 코드도 확인해봅시다. List 에 포함된 셀의 View 이며, 화면에 표시되면 Asynchronous 한 작업을 수행해야 합니다.

```swift
struct ThumbnailView: View {
    @ObservedObject var viewModel: ViewModel 
    var post: Post
    @State private var image: UIImage?
    
    var body: some View {
        Image (uiImage: self.image ?? placeholder)
            .onAppear {
                self.viewModel.fetchThumbnail(for: post.id) { result, _ in
                    self.image = result                
                }
            }
    }
```

이 Ugly 한 코드를 수정해봅니다. ohAppear 부분만 표시합니다. 

```swift
.onAppear {
    self.image = try? await self.viewModel.fetchThumbnail(for: post.id)
}
```

하지만 이 코드는 아래와 같은 컴파일 에러를 발생시킵니다.

> async 'fetchThumbnail(for:)' used in a context that does not support concurrency

이유는 간단합니다. 위의 await 실행문이 들어간 closure, function 을 확인해보면 모두 async 를 선언해주었습니다. Swift 는 async/await 는 같은 context 안에서 실행되어야 함을 언급하는 것입니다.

두 Context 를 아래와 같이 연결시킬 수 있습니다.

```swift
.onAppear {
    Task {
        self.image = try? await self.viewModel.fetchThumbnail(for: post.id)
    }
}
```

Task 는 클로저를 시스템에 전달하여 즉시 실행 Context 로 전달되도록 합니다. 이는 마치 DispatchQueue 클래스를 이용해 Global Queue 에 클로저를 전달하는 것과 같습니다.

iOS SDK 에는 이미 async/await 를 활용할 수 있는 수많은 API 들이 준비되어 있습니다. 예를 들어 `URLSession.shared.data(for:URLRequest)` 와 같은 것들입니다.

References:
* [WWDC21 - Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/)