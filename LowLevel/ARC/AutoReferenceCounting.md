## ARC

WWDC 21의 세션을 참고한다.

Swift 는 Struct/Enums 같은 값타입을 제공하는데 이러한 값타입을 사용하면 참조타입에서 오는 Reference Cycle 에 의한 Memory Leak 의 위험을 피할 수 있다. 

하지만 값 타입은 생명주기가 짧아서 오랫동안 상태를 유지해야 할 경우는 Class 를 사용하는 것이 좋다. 하지만 메모리의 효율적인 사용을 위해서 Class 로 Instance 를 생성할 경우 Swift 는 Instance 의 메모리를 Automatic Reference Counting 혹은 ARC 를 통해 객체의 메모리를 관리한다.

3 개의 개념을 나눠 설명해보도록 한다.

* 객체의 생명주기
* Reference Count 와 고려해야 할 사항
* 객체를 다루는 안전한 테크닉

### 객체의 생명주기와 ARC

1. ARC 는 객체의 생명주기가 끝나면 객체를 해제시킨다. 객체의 생명주기는 생성자를 호출하는 것으로 시작해서 객체의 사용이 종료될 경우 같이 종료된다. C++처럼 괄호의 마지막이 아니라 마지막 사용에 의해 생명주기가 결정된다.<br/>(하지만 대부분 괄호의 마지막에 생명주기가 끝나는데 이는 Swift Compiler 가 생명주기를 Strict 하게 다루지 않기 때문이며, 프로젝트 빌드 설정인 "Optimize Object Lifetimes" 을 통해 Strict 한 관리가 가능하다.) 
2. ARC 는 객체의 생명주기를 참조 카운트(ref_count 혹은 RC)로 판단한다. Swift Runtime 은 객체의 ref_count 가 0이 되면 객체를 해제시킨다.
3. ARC 의 동작방식은 Swift Compiler 가 컴파일 과정에서 코드 중간마다 retain/release 함수를 삽입함으로써 메모리 관리를 하는 것이다.<br/>Swift Compiler 에 의해 ARC 는 객체의 생성자가 실행될 때 retain 함수를 삽입하고 마지막 사용이 확인되면 release 함수를 삽입한다.

### Reference Count Deep-Dive

ref_count 는 세 개의 종류가 있다.

| Name                    | Description                                                                                                                                                                                                                                                                                                    |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Strong Reference Count  | 객체가 강하게 참조된 갯수를 나타낸다. 강하다는 뜻은 이 카운트가 0으로 사라지면 객체도 사라지기 때문이다.<br/>Strong RC 는 extra count 로도 증가될 수 있다. 실제 필드의 카운트가 0이면 논리적으로는 1로 간주해야 한다는 것이다.                                                                                                                                                                  |
| Unowned Reference Count | 객체가 미소유로 참조된 갯수를 나타낸다.<br/>미소유 참조는 특이하게 강한 참조가 생성자로 인해 1 증가할 때 같이 추가된다. 이 추가 카운트는 deinit 완료 시 사라진다.<br/>미소유 ref_count 가 중요한 이유는 이 카운트가 0이 될 경우 객체가 freed 되기 때문이다.<br/>Strong RC는 0이 되면 deinit, Unowned RC는 0이 되면 freed 된다.<br/>deinit 은 인스턴스가 소멸하는 것이고 freed 는 힙 메모리에서 객체가 해제되는 것이다. 순서로는 deinit 다음이 freed 이다. |
| Weak Reference Count    | 객체가 약하게 참조된 갯수를 나타낸다.<br/>미소유 참조처럼 약한 참조는 미소유 참조가 1 증가할 때 같이 추가된다. 이 추가 카운트는 freed 될 때 같이 사라진다.<br/>미소유 참조가 0이 되면 freed 되는 것처럼 약한 참조는 0이 될 경우 객체의 side table entry 가 해제된다.(Side table 은 아래 상세히 설명)                                                                                                             |

### Weak/Unowned Reference 로 인한 문제

Weak/Unowned Reference 는 ref_count 를 증가시키지 않기 때문에 Reference Cycle 현상을 개선하는데 유용하다.

Reference Cycle 이란 객체의 ref_count 가 줄어들 가능성이 존재하지 않는 상황임에도 1 이상의 ref_count 가 남아있어서 객체가 해제되지 않아 Memory Leak 을 발생시키는 것을 말한다.

Weak/Unowned Reference 된 객체의 생명주기가 끝난 상태에서 객체가 다시 사용될 경우 weak 변수는 nil 을 반환하고, unowned 는 크래쉬가 발생한다. 그래서 실제 객체를 사용할 때 weak 는 Optional Binding, unowned 는 Unowned Optional 로 바꾸고 Optional Binding 을 혼합해서 사용할 수 있다.

하지만 이런 방식으로 weak/unowned 말고도 Memory Leak 을 방지하는 방법은 몇가지 더 있다. 사실 위에서 Binding 을 활용하는 방법은 Silence Bug 를 찾지 못한 채로 프로젝트를 방치할 수 있습니다.

* withExtendedLifetime() 함수를 weak 변수에 대해 사용한다.
* 코드 내의 Strong/Weak/Unowned 참조를 다시 디자인한다.

withExtendedLifetime()는 특정 객체가 파라미터로 반영되거나 실행되는 클로저 내에서 사용될 경우 클로저가 끝나기 전까지는 생명주기가 끝나지 않는다. defer 처럼 사용할 수도 있는 것이다. 하지만 이 방법은 Weak 참조일 때만 가능하고 유지보수 비용도 늘어나며 코드 가독성에 문제가 생길 수도 있다.

참조할 객체에 대해 다시 디자인 하는 방법은 상황에 따라 여러가지가 될 수 있다. 예를 들어 Weak/Unowned 로 참조해야 하는 객체만을 따로 참조하는 객체를 따로 분리하는 것이다. Reference Cycle 을 관리할 때 새로 생성된 객체만을 관리하면 되는 것이다.

### Deinitializer 에 의한 문제

만약 Deinitializer 가 객체 내부를 변환시키는 구조는 지양하도록 하자. Deinitializer 에서 객체의 사용이 중지된 상태에서 객체가 다시 참조될 수도 있기 때문에 어떤 부작용이 발생할지 알 수 없다.

위에서 제시한 withExtendedLifetime(), Redesign 을 추천한다.

### Optimize Object Lifetimes

Xcode 프로젝트의 빌드 옵션 중 하나이다. 객체의 생명주기를 좀 더 Strict 하게 다룬다.

이 옵션이 활성화되지 않은 경우 대부분의 객체는 실제 자신이 생성된 Scope 내의 괄호가 끝나는 시점에 해제된다. 하지만 ARC 에서 객체는 사용이 끝나는 시점에 메모리에서 해제되는 것이 가장 효율적인 메모리 관리법이다.

해당 옵션을 활성화하여 객체의 사용이 끝나면 바로 해제시킬 수 있도록 하게 되면 효율적인 메모리 관리가 가능하지만, 메모리 해제로 인한 Silence warning 이 발생할 수도 있다.

---

### Side Table

Swift 4부터 추가되었다. 객체에 대한 Side Table 이 발생하는 경우는 다음과 같다. 

* 객체에 추가 공간이 필요한 경우
* 범위 overflow 가 발생하는 경우
* 약한 참조가 발생하는 경우

첫번째와 두번째는 흔치 않은 경우이므로 세번째에 대해서만 알아보도록 하자.

약한 참조는 특이한 점이 하나 있는데, **강한/미소유 참조는 객체를 직접 참조하지만 약한 참조는 객체의 사이드 테이블을 참조한다는 것이다.** 힙 메모리의 객체과 Side-Table 을 코드를 통해서 더욱 자세히 알아보도록 하자.

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

HeapObject 가 객체인데 isa 저장소, InlineRefCounts 를 가진다. (상세한 설명은 추가 업데이트 예정)

InlineRefCounts 안에는 strong/unowned RC 는 존재하지만, weak RC 는 없다. 대신, HeapObjectSideTableEntry 포인터를 갖는다. 사이드 테이블의 구현에는 strong/unowned/weak RC 를 갖는다.

**객체의 생명주기와 Reference Count**

이제 객체가 생성되고 소멸되기까지의 시나리오를 그려보도록 한다.

객체는 약하게 참조되지 않아 사이드 테이블 없이 존재할 경우 strong RC 1/unowned RC 1/weak RC 1 참조를 갖는다. 강한 참조에 의해 미소유 참조 카운트가 하나 늘어나고, 미소유 참조에 의해 약한 참조 카운트가 증가하기 때문이다.

* strong/unowned 참조로 이 객체를 참조해도 문제가 없다.
* weak 참조는 객체를 참조할 수 없다. 앞에서 설명했다시피 약한 참조는 Side table 을 참조하기 때문인데, 아직 Side table 이 존재하지 않기 때문이다.

UITableView 의 delegate 프로퍼티에 UITableViewDelegate 를 구현한 객체를 생성자로 반영하였는데 버그가 생겨서 디버깅을 해보면, 분명히 생성자로 반영한 delegate 프로퍼티가 nil 로 되어 있는 경우가 종종 있다. 이는 weak 참조만 존재하는 상태에서 delegate 프로퍼티에 객체를 반영하는 함수가 종료되면 retain, release 될 수 있기 때문이다. 

다시 얘기로 돌아와서 약한 참조가 없는 객체에 약한 참조를 한다고 가정해보자. 이러면 Side table 이 생기는데 LIVE with side table 이라고도 한다. 이 때는 약한 참조들도 해당 객체를 참조할 수 있다.

이제 강한 참조가 0이 되면 deinit 이 호출된다.

### DEINITING with/without side table

Side table 이 없는 상태에서 객체의 deinit 이 실행될 경우(실행중) 각 참조상태에 대해 알아보자.

* 강한 참조는 아무 변화가 없다.
* 약한 참조는 객체를 불러올 수 없으며 nil 이 저장된다.
* 미소유 참조는 swift_abortRetainUnowned()를 호출하며 객체 불러오길 거부하고 나머지는 변화가 없다.<br/>deinit 이 종료되면 swift_deallocObject 가 실행되고 deinit 에 미소유 참조나 약한 참조가 있는지 canBeFreedNow() 를 통해 확인한다.<br/>미소유/약한 참조가 발견되면 미소유 참조 카운트가 하나 줄어들고 객체는 DEINITED 상태가 된다. 하지만 강한 참조 말고 다른 참조가 없으면 DEAD 상태가 된다.

DEINITED 상태에서는 강한 참조 카운트가 0이지만 미소유 참조는 존재한다. 이 상태에서 미소유 참조 카운트가 0이 되면 DEAD 상태로 변경된다. FREED 상태는 없다

Side table 이 있는 상태라면 deinit 실행 중 약한 참조가 존재하고 앞으로도 존재한다는 것이기 떄문에 canBeFreedNow() 에서는 항상 false 가 반환된다. DEINITING 상태가 된다.

FREED 상태는 객체가 해제되었음에도 객체에 대한 Side table 에 약한 참조 카운트가 남아있는 경우이다(약한 참조에 대한 값은 nil). 여기서 약한 참조까지 0이 되면 Side table entry 까지 메모리에서 해제되면서 DEAD 상태가 된다.

Reference:
* [https://developer.apple.com/videos/play/wwdc2021/10216/](https://developer.apple.com/videos/play/wwdc2021/10216/)
* [https://jeonyeohun.tistory.com/373](https://jeonyeohun.tistory.com/373)
* [https://sihyungyou.github.io/iOS-side-table/](https://sihyungyou.github.io/iOS-side-table/)
