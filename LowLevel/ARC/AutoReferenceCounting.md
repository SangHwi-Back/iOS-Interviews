## ARC

WWDC 21의 세션을 참고한다. Swift는 Struct/Enums 같은 강력한 값타입을 제공하는데 값타입을 사용하면 참조타입에서 오는 Memory Leak의 위험을 피할 수 있다. 만약 당신이 Class를 사용하고자 한다면 Swift는 메모리를 Automatic Reference Counting 혹은 ARC로 관리한다.

우선 객체의 생명주기와 ARC를 엮어서 설명한 뒤 언어 특성을 통해 자세한 객체 생명주기를 예상하는 방법을 설명할 것이다. 마지막으로 안전한 테크닉들을 이용해서 문제 상황을 해결해보자.

### 객체의 생명주기와 ARC

1. 객체의 생명주기는 생성자로 시작해서 마지막 사용 시 종료된다. C++처럼 괄호의 마지막이 아니라 마지막 사용(하지만 대부분 괄호의 마지막에 생명주기가 끝난다)에 의해 생명주기가 결정된다. ARC는 객체의 생명주기가 끝나면 객체를 해제시킨다.
2. ARC는 객체의 생명주기를 ref_count로 판단한다. Swift Runtime은 객체의 Reference count가 0이 되면 객체를 해제시키는 것이다.
3. ARC는 Swift Compiler가 retain/release 함수를 삽입함으로써 메모리 관리를 하는 것이다. Swift Compiler에 의해 ARC는 객체의 생성자가 실행될 때 retain 함수를 삽입(윗줄에 삽입한다 생각하면 편함)하고 마지막 사용이 확인되면 release 함수를 삽입(아랫줄에 삽입한다 생각하면 편함)한다.

### Weak/Unowned Reference로 인한 문제

Weak/Unowned Reference는 ref_count를 증가시키지 않기 때문에 reference cycle을 해결한다.

reference cycle이란 객체의 ref_count가 줄어들 가능성이 존재하지 않는 상태에서도 1 이상이라서 객체가 해제되지 않아서 Memory Leak을 발생시키는 것을 말한다.

Weak/Unowned Reference 된 객체의 생명주기가 끝난 상태에서 사용될 경우 weak는 nil을 반환하고, unowned는 크래쉬가 발생한다. 이런 것을 방지하기 위해서는 weak일 경우 Optional Binding을 이용하거나, unowned의 경우 Unowned Optional와 Optional Binding을 같이 사용하면 되지만 이건 Silent Bug를 유발한다. 두 가지 해결책이 있는데 withExtendedLifetime() 함수(Weak 일 때만 사용)를 사용하거나 Strong/Weak/Unowned 참조를 다시 디자인 해보는 것이다.

withExtendedLifetime()에 객체를 파라미터로 넣거나 클로저에서 객체가 사용될 경우 해제될 객체도 클로저가 끝나기 전까지는 생명주기가 끝나지 않는다. defer에 함수 실행문을 넣는 것도 괜찮지만 이 방법은 Weak 참조일 때만 가능하고 약간 신경이 쓰인다.

아니면 Weak/Unowned로 참조해야 할 참조를 수행하는 다른 객체를 만들어서(나눠서, 분리하여) Reference Cycle 자체를 없애는 방법도 있다.

### Deinitializer에 의한 문제

만약 Deinitializer가 객체 내부를 변환시키는 구조라면 큰일이다. 사용이 중지되면 객체가 해제될 수도 있기 때문에 deinit이 호출되고 객체 내부의 프로퍼티 값들이 변경되면 사용이 중지된 이후에 어떤 부작용이 발생할지 알 수 없다.

위와 마찬가지로 withExtendedLifetime() 함수(Weak 일 때만 사용)를 사용하거나 Strong/Weak/Unowned 참조를 다시 디자인 해본다.

### Optimize Object Lifetimes

이 빌드 옵션은 객체의 생명주기를 좀 더 타이트하게 다룬다. 만약 객체의 사용이 끝나면 바로 해제시켜 버리기 때문에 Silence warning을 발견할 수도 있다.

### Reference Count Deep-Dive

객체는 개념적으로 세 개의 ref_count를 갖는다.

Strong Reference Count는 객체가 강하게 참조된 갯수를 나타낸다. 강하다는 뜻은 이 카운트가 0이 되면 객체가 deinit되기 때문이다. Strong RC는 extra count로 저장된다. 실제 필드의 카운트가 0이면 논리적으로는 1로 간주해야 한다는 것이다.

Unowned Reference Count는 객체가 미소유로 참조된 갯수를 나타낸다. 미소유 참조는 특이하게 강한 참조에 대해 1의 추가적인 카운트로서 추가될 수 있다. 이 추가 카운트는 deinit 완료 시 사라진다. 미소유 ref_count가 중요한 이유는 이 카운트가 0이 될 경우 객체가 freed 되기 때문이다. Strong RC는 0이 되면 deinit, Unowned RC는 0이 되면 freed 된다. deinit은 인스턴스가 소멸하는 것이고 freed는 힙 메모리에서 객체가 해제되는 것이다. deinit 다음이 freed로 순서가 있다.

Weak Reference Countsms 객체가 약하게 참조된 갯수를 나타낸다. 미소유 참조 때 처럼 약한 참조는 미소유 참조에 대해 1의 추가적인 카운트로서 추가될 수 있다. 이 추가 카운트는 freed될 때 같이 사라진다. 미소유 참조가 0이 되면 freed 되는 것처럼 약한 참조는 0이 될 경우 객체의 side table entry가 해제된다.

### Side Table

Swift 4부터 추가되었다. 객체에게 추가적인 공간이 필요하거나 범위 overflow가 일어나거나 약한 참조가 발생할 경우 객체에 대한 사이드 테이블이 생성된다. 특이한 점이 하나 있는데, 강한/미소유 참조는 객체를 직접 참조하지만 약한 참조는 객체의 사이드 테이블을 참조한다는 것이다.

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


객체의 구현과 사이드테이블의 구현이다. HeapObject가 객체인데 isa 저장소(뭔지 모름) InlineRefCounts 라는 걸 가진다. InlineRefCounts 안에는 강한/미소유 참조, 플래그 값을 갖지만 약한 참조는 없다. 대신, HeapObjectSideTableEntry 포인터를 갖는다. 사이드 테이블의 구현에는 강한/미소유/약한 참조를 갖고 플래그 값도 갖는다.

객체의 생명주기와 Reference Count

객체는 사이드 테이블 없이 존재할 경우 강한1/미소유1/약한1 참조를 갖는다. 강한 참조에 의해 미소유 참조가 하나 늘어나고, 미소유 참조에 의해 약한 참소가 증가하기 때문이다. 이 상태에서는 강한/미소유 참조로 이 객체를 참조해도 문제가 없다.

하지만 약한 참조는 객체를 참조할 수 없다. 앞에서 설명했다시피 약한 참조는 Side table을 참조하기 때문인데, 아직 Side table이 존재하지 않기 때문이다. 일례로 UITableView를 UIViewController 내에서 생성한 뒤 delegate/datasource 프로퍼티에 생성자로 객체를 생성하였는데 나중에 디버깅 시 nil로 되어 있는 것을 경험했다면 이것이 그 이유다.

여기서 추가로 약한 참조가 발생한다고 가정해보자. 이러면 Side table이 생기는데 LIVE with side table 이라고도 한다. 이 때는 약한 참조들도 해당 객체를 참조할 수 있다.

이제 강한 참조가 0이 되면 deinit이 호출된다.

### DEINITING with/without side table

Side table이 없는 상태에서 객체의 deinit()이 실행될 경우(실행중) 각 참조상태에 대해 알아보자. 강한 참조는 아무 변화가 없다. 약한 참조는 객체를 불러올 수 없으며 nil이 저장된다. 미소유 참조는 swift_abortRetainUnowned()를 호출하며 객체 불러오길 거부하고 나머지는 변화가 없다. deinit()이 종료되면 swift_deallocObject가 실행되고 deinit에 미소유 참조나 약한 참조가 있었는지 canBeFreedNow()를 통해 확인한다. 미소유/약한 참조가 발견되면 미소유 참조 카운트가 하나 줄어들고 객체는 DEINITED 상태가 된다. 하지만 강한 참조 말고 다른 참조가 없으면 DEAD 상태가 된다.

DEINITED 상태에서는 강한 참조 카운트가 0이지만 미소유 참조는 존재한다. 이 상태에서 미소유 참조 카운트가 0이 되면 DEAD 상태로 변경된다. FREED 상태는 없다

Side table이 있는 상태라면 deinit() 실행 중 약한 참조가 존재하고 앞으로도 존재한다는 것이기 떄문에 canBeFreedNow()에서는 항상 false가 반환된다. DEINITING 상태가 된다. FREED 상태는 객체가 해제되었음에도 객체에 대한 Side table에 약한 참조 카운트가 남아있는 경우이다(약한 참조에 대한 값은 nil). 여기서 약한 참조까지 0이 되면 Side table entry까지 메모리에서 해제되면서 DEAD 상태가 된다.

Reference:
* [https://developer.apple.com/videos/play/wwdc2021/10216/](https://developer.apple.com/videos/play/wwdc2021/10216/)
* [https://jeonyeohun.tistory.com/373](https://jeonyeohun.tistory.com/373)
* [https://sihyungyou.github.io/iOS-side-table/](https://sihyungyou.github.io/iOS-side-table/)
