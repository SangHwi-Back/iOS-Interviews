## ARC

WWDC 21의 세션을 참고한다.

Swift 는 Struct/Enums 같은 값타입을 제공하는데 이러한 값타입을 사용하면 참조타입에서 오는 Reference Cycle 에 의한 Memory Leak 의 위험을 피할 수 있다. 

하지만 값 타입은 생명주기가 짧아서 오랫동안 상태를 유지해야 할 경우는 Class 를 사용하는 것이 좋다. 하지만 메모리의 효율적인 사용을 위해서 Class 로 Instance 를 생성할 경우, Swift 는 Instance 의 메모리를 ARC (Automatic Reference Counting) 라는 메모리 관리 기법을 사용한다.

3 개의 개념을 나눠 설명해보도록 한다.

* 객체의 생명주기
* Reference Count 와 고려해야 할 사항
* 객체를 다루는 안전한 테크닉

### 객체의 생명주기와 ARC

1. ARC 는 객체의 생명주기가 시작되면 객체를 메모리에 retain 하고 생명주기가 종료되면 release 하기 위해 존재하는 Swift Compiler 의 메모리 관리 기법이다.
2. 객체의 생명주기는 생성자를 호출하는 것으로 시작한다.
3. 객체의 생명주기는 객체가 더 이상 사용되지 않을 때 끝난다.
4. 하지만 대부분 괄호의 마지막에 생명주기가 끝난다. 이는 Swift Compiler 가 생명주기를 Strict 하게 다루지 않기 때문이며, 프로젝트 빌드 설정인 "Optimize Object Lifetimes" 을 통해 Strict 한 관리가 가능하다.
5. 여기서 ARC 는 객체의 생명주기의 끝을 판단해야 한다. 여기서의 판단 기준은 참조 카운트(ref_count 혹은 RC)이다. Swift Runtime 은 객체의 참조 카운트가 0이 되면 객체를 해제시킨다.
6. ARC 의 동작방식은 다음과 같다.
   - Swift Compiler 가 컴파일 과정에서 코드 중간마다 retain/release 함수를 삽입
   - 객체의 생성자가 실행되면 retain 함수가 삽입되고, 생명주기가 끝날 때 release 함수를 삽입하여 실행되도록 한다

```swift
class Traveler {
    var name: String
    var destination: String?
}

func test() {
    let traveler1 = Traveler(name: "Lily") // Retain count added 1.
    // retain for 'traveler1', Retain count added 1.
    let traveler2 = traveler1
    // release for 'traveler1', Retain count decreased 1.
    traveler2.destination = "Big Sur"
    // release for 'traveler1', Retain count decreased 1.
    print("Done traveling")
}
```

### Reference Count Deep-Dive

ref_count 는 세 개의 종류가 있다.

| Name                    | Description                                                                                                                                                                                                                                                                                                    |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Strong Reference Count  | 객체가 강하게 참조된 갯수를 나타낸다. 강하다는 뜻은 이 카운트가 0으로 사라지면 객체도 사라지기 때문이다.<br/>Strong RC 는 extra count 로도 증가될 수 있다. 실제 필드의 카운트가 0이면 논리적으로는 1로 간주해야 한다는 것이다.                                                                                                                                                                  |
| Unowned Reference Count | 객체가 미소유로 참조된 갯수를 나타낸다.<br/>미소유 참조는 특이하게 강한 참조가 생성자로 인해 1 증가할 때 같이 추가된다. 이 추가 카운트는 deinit 완료 시 사라진다.<br/>미소유 ref_count 가 중요한 이유는 이 카운트가 0이 될 경우 객체가 freed 되기 때문이다.<br/>Strong RC는 0이 되면 deinit, Unowned RC는 0이 되면 freed 된다.<br/>deinit 은 인스턴스가 소멸하는 것이고 freed 는 힙 메모리에서 객체가 해제되는 것이다. 순서로는 deinit 다음이 freed 이다. |
| Weak Reference Count    | 객체가 약하게 참조된 갯수를 나타낸다.<br/>미소유 참조처럼 약한 참조는 미소유 참조가 1 증가할 때 같이 추가된다. 이 추가 카운트는 freed 될 때 같이 사라진다.<br/>미소유 참조가 0이 되면 freed 되는 것처럼 약한 참조는 0이 될 경우 객체의 side table entry 가 해제된다.(Side table 은 아래 상세히 설명)                                                                                                             |

### Weak/Unowned Reference 로 인한 문제

Weak/Unowned Reference 는 ref_count 를 증가시키지 않기 때문에 Reference Cycle 현상을 개선하는데 유용하다.

Reference Cycle 이란 객체끼리 서로 참조하는 상황에서 ref_count 가 줄어들 가능성이 존재하지 않는 상황임에도 1 이상의 ref_count 가 남아있어서 객체가 해제되지 않아 Memory Leak 을 발생시키는 것을 말한다.

Weak/Unowned Reference 된 객체의 생명주기가 끝난 상태에서 객체가 다시 사용될 경우 weak 변수는 nil 을 반환하고, unowned 는 크래쉬가 발생한다. 그래서 실제 객체를 사용할 때 weak 는 Optional Binding, unowned 는 Unowned Optional 로 바꾸고 Optional Binding 을 혼합해서 사용할 수 있다.

하지만 이런 방식으로 weak/unowned 말고도 Memory Leak 을 방지하는 방법은 몇가지 더 있다. (Optional Binding 은 앱의 Silence Bug 를 찾지 못하게 만들 수도 있다)

* withExtendedLifetime()
* Code Redesign

withExtendedLifetime()는 파라미터로 특정 객체를 전달하면, 클로저가 끝나기 전까지는 파라미터 객체의 생명주기가 끝나지 않도록 한다. defer 처럼 사용할 수도 있는 것이다. 하지만 이 방법보다는 아래의 Redesign 을 추천하는 경우가 많은데, 유지보수 비용 상승과 코드 가독성 문제 때문이다.

참조할 객체에 대해 다시 디자인 하는 방법은 상황에 따라 여러가지가 될 수 있다. 예를 들어 Weak/Unowned 로 참조해야 하는 프로퍼티만을 따로 관리하는 객체를 만드는 것이다. Reference Cycle 을 일으킨 객체들은 새로 만들어진 객체를 참조하기 때문에 ref_count 관리가 수월해진다. Reference Cycle 을 관리할 때 새로 생성된 객체만을 관리하면 되는 것이다.

코드에 Weak/Unowned 를 넣을 때는 항상 명확한 이유가 필요하다. 이러한 참조들은 추가적인 관리비용이 발생한다는 사실을 잊어서는 안된다.

### Deinitializer 에 의한 문제

Deinitializer 가 객체 내부를 변환시키는 구조는 지양하도록 하자. Deinitializer 에서 객체의 사용이 중지된 상태에서 객체가 다시 참조될 수도 있기 때문에 어떤 부작용이 발생할지 알 수 없다.

위에서 제시한 withExtendedLifetime(), Redesign 을 추천한다.

### Optimize Object Lifetimes

Xcode 프로젝트의 빌드 옵션 중 하나이다. 객체의 생명주기를 좀 더 Strict 하게 다룬다.

이 옵션이 활성화되지 않은 경우 대부분의 객체는 실제 자신이 생성된 Scope 내의 괄호가 끝나는 시점에 해제된다. 하지만 ARC 에서 객체는 사용이 끝나는 시점에 메모리에서 해제되는 것이 가장 효율적인 메모리 관리법이다.

해당 옵션을 활성화하여 객체의 사용이 끝나면 바로 해제시킬 수 있도록 하게 되면 효율적인 메모리 관리가 가능하지만, 메모리 해제로 인한 Silence warning 이 발생할 수도 있다.

---

### Side Table

Swift 4부터 추가되었다. 객체에 대한 약한 참조를 저장하기 위해 만들어진 저장소이다.

사실 참조 카운트에는 Strong/Weak/Unowned 카운트가 따로 나뉘어져 있다. Swift 4 이전엔 객체에 대한 참조들이 모두 한 저장소에 저장되었다. 여기서 Strong 참조가 0이 되면 객체가 메모리에서 사라지는 것이 일반적이었다(deallocated).

하지만 Strong 참조 0 은 객체를 deinitialized 상태로 만든다. 만약 Strong 참조 0, Weak 참조 1 인 상태일 경우 만으로는 메모리에서 객체가 해제되는 것을 보장하지 않는다. 때문에 객체가 좀비 상태로 남아버리는, 즉 Memory Leak 이 발생하는 문제가 있었다.

참고로 Weak 참조에 의해 좀비 객체가 발생한다면 메모리 해제 알고리즘을 동작시켜서 객체를 메모리에서 해제시켜 주어야 한다. 문제는 이 알고리즘은 '런타임에' Weak 참조가 다시 로드 되어야만 작동된다는 것이었다. 단순히 Strong 참조만을 고려하여 객체가 메모리에서 해제된다고 여길 경우 Silent Bug 및 Memory Leak 을 방치하게 될 수도 있기에 상당히 까다로운 내용이었다.

그래서 Weak 참조는 Side-Table 이라는 별도의 메모리를 참조하기 시작한다(Strong/Unowned 참조는 객체 자체를 참조한다). Side-Table 은 객체의 추가적인 정보를 저장하기 위한 분리된 메모리이다.

이제 Weak 참조는 별개의 문제인 상태로, 강한 참조가 0이 되면 객체는 어김없이 deallocated 된다. 좀 더 효율적인(가차없는) 메모리 관리가 진행되는 것이다.

아래는 Side Table 이 발생하는 조건을 나열한 것이다.

* 객체에 추가 공간이 필요한 경우
* Strong / Unowned 참조가 overflow 가 되는 경우
* Weak 참조가 발생하는 경우

첫번째와 두번째는 흔치 않은 경우이므로 세번째인 Weak 참조가 발생하는 경우만을 고려하는 것이 일반적이다.

Weak 참조가 Strong/Unowned 참조처럼 객체를 직접 참조하는 것이 아닌 객체의 Side Table 을 참조한다는 사실을 기억하자.

HeapObject 를 생성하는 소스코드를 확인하면 SideTable 의 생성에 관해 좀 더 자세히 알 수 있다. RefCount.h 소스코드의 주석에서 객체가 저장되는 형태(Storage Layout) 라고 언급한 주석을 참고하였다.

[RefCount.h](https://github.com/apple/swift/blob/d1c87f3c936c41418ee93320e42d523b3f51b6df/stdlib/public/SwiftShims/RefCount.h)

```cpp
HeapObject {
    isa
    InlineRefCounts {
        atomic<InlineRefCountBits> {
            strong RC + unowned RC + flags
            OR
            HeapObjectSideTableEntry*
        }
    }
}

HeapObjectSideTableEntry {
    SideTableRefCounts {
        object pointer
        atomic<SideTableRefCountBits> {
            strong RC + unowned RC + weak RC + flags
        }
    }
}
```

HeapObject 를 생성하는 과정에서 isa 저장소, InlineRefCounts 가 생성된다.

InlineRefCounts 를 생성하는 과정에서 HeapObjectSideTableEntry 객체를 추가로 생성하는 코드가 보인다. 코드를 자세히 보면 InlineRefCounts 안에는 Strong/Unowned 참조가 존재하지만, Weak 참조는 없다. 대신, HeapObjectSideTableEntry 에는 Strong/Unowned/Weak 참조 모두를 갖는다.

Side Table 의 구현에는 Strong/Unowned/Weak 참조 정보를 모두 갖는다는 것에 주의하자. Side Table 은 객체에 추가 공간이 필요하거나 Strong/Unowned 참조가 overflow 되는 경우에도 필요하다.

### 객체의 생명주기와 Reference Count

이제 객체가 생성되고 소멸되기까지의 시나리오를 그려보도록 한다.

객체에 Weak 참조가 존재하지 않아 Side Table 도 없을 경우 Strong 참조 1 / Unowned 참조 1 / Weak 참조 1 을 갖는다. Strong 참조에 의해 Unowned 참조 카운트가 하나 늘어나고, Unowned 참조에 의해 Weak 참조 카운트가 증가하기 때문이다.

Weak 참조가 없는 객체에 Weak 참조를 한다고 가정해보자. 이러면 Side Table 이 생성되고 Side Table 에 Weak 참조 카운트를 증가시킨다. 이렇게 존재하는 Weak 참조를 LIVE with side table 이라고도 한다.

아래의 예시를 통해 동작방식을 자세히 이해해보도록 하자.

* UITableView 의 delegate 프로퍼티에 UITableViewDelegate 를 구현한 객체를 생성자로 반영하였는데 버그가 생겨서 디버깅을 해보면, 분명히 생성자로 반영한 delegate 프로퍼티가 nil 로 되어 있는 경우가 종종 있다.
* UITableView 의 delegate 프로퍼티는 Weak 참조를 하고 있고, 이 프로퍼티에 delegate 객체를 참조시키면 프로퍼티는 객체의 Side Table 을 참조한다.
* 이 상황에서 Strong 참조가 0이 되는 순간 이는 weak 참조만 존재하는 상태에서 delegate 프로퍼티에 객체를 반영하는 함수가 종료되면 retain, release 될 수 있기 때문이다. 
* 이제 강한 참조가 0이 되면 deinit 이 호출된다.

### DEINITING with/without side table

Side table 이 없는 상태에서 객체의 deinit 이 실행될 경우(실행중) 각 참조상태에 대해 알아보자.

* Strong 참조는 아무 변화가 없다.
* Weak 참조는 객체를 불러올 수 없으며 nil 이 저장된다.
* Unowned 참조는 swift_abortRetainUnowned()를 호출하며 객체 불러오길 거부하고 나머지는 변화가 없다.

- deinit 이 종료되면 swift_deallocObject 가 실행된다. 
- deinit 에 Unowned 참조나 Weak 참조가 있는지 canBeFreedNow() 를 통해 확인한다.
- Unowned/Weak 참조가 발견되면 Unowned 참조 카운트가 하나 줄어들고 객체는 DEINITED 상태가 된다.
- 하지만 Strong 참조 말고 다른 참조가 없으면 DEAD 상태가 된다.

DEINITED 상태에서는 강한 참조 카운트가 0이지만 미소유 참조는 존재한다. 이 상태에서 미소유 참조 카운트가 0이 되면 DEAD 상태로 변경된다. FREED 상태는 없다

Side table 이 있는 상태라면 deinit 실행 중 약한 참조가 존재하고 앞으로도 존재한다는 것이기 떄문에 canBeFreedNow() 에서는 항상 false 가 반환된다. DEINITING 상태가 된다.

FREED 상태는 객체가 해제되었음에도 객체에 대한 Side table 에 약한 참조 카운트가 남아있는 경우이다(약한 참조에 대한 값은 nil). 여기서 약한 참조까지 0이 되면 Side table entry 까지 메모리에서 해제되면서 DEAD 상태가 된다.

## Summary

ARC 는 Swift 에서 메모리 관리를 위해 도입한 메모리 관리 전략이다. 일반적으로 ARC 는 개발자가 메모리 관리에 대한 고려 없이도 프로그래밍을 할 수 있도록 도와준다.

동작방식은 Swift Compiler 가 컴파일 되는 코드에 retain/release 함수를 삽입해주는 것이다. 객체가 실제 사용되기 위해서는 메모리로 load 되어야 하는데, 이를 담당하는 retain, 그리고 load 된 메모리를 해제시켜주는 retain 함수를 내부 알고리즘에 따라 삽입해주는 것이다.

ARC 를 통해 객체의 메모리를 관리할 때는 참조 카운트(Reference Count)를 이용한다. retain 은 참조 카운트를 1 증가, release 는 1 감소시키는데, 참조 카운트가 0이 되면 객체를 해제시키는 방식으로 메모리를 관리한다. Swift Compiler 는 이를 토대로 retain/release 를 적절히 코드 내에 삽입한다.

ARC 가 자동으로 메모리 관리를 해줌에도 이를 알아야 하는 이유는 작업자의 실수 등으로 인한 Reference Cycle 및 Memory Leak 을 발견하고 수정하기 위해서이다.

Reference:
* [https://developer.apple.com/videos/play/wwdc2021/10216/](https://developer.apple.com/videos/play/wwdc2021/10216/)
* [https://jeonyeohun.tistory.com/373](https://jeonyeohun.tistory.com/373)
* [https://sihyungyou.github.io/iOS-side-table/](https://sihyungyou.github.io/iOS-side-table/)
