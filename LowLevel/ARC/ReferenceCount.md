# ReferenceCount

## 개요

객체가 초기화되어 메모리에 적재된 이후 메모리에서 해제되는 시점을 알기 위해 참조 카운트가 0이 되면 메모리에서 해제되도록 하는 방식을 뜻한다.

Swift 에서도 객체에 대한 메모리 관리를 효율적으로 하기 위하여 이 방식으로 객체의 메모리를 관리한다. iOS 에서는 애플리케이션에서 초기화한 객체를 Memory 구조 중 Heap Memory 에 저장한다.

메모리를 관리하는 방식을 간단히 설명하자면, 메모리를 점유한 객체가 참조되는 횟수를 Reference Count 로 저장하고 있다가 참조가 해제되면 저장한 Count 를 줄이고 0이 되면 메모리를 해제하도록 한다.

## Reference Count 의 종류

[AutoReferenceCounting 의 reference-count-deep-dive 참고](https://github.com/SangHwi-Back/iOS-Interviews/blob/main/LowLevel/ARC/AutoReferenceCounting.md#reference-count-deep-dive)

## Reference Count 가 저장되는 장소

Reference Count 는 기본적으로 객체가 인스턴스화 되면 생성되는 HeapObject 에 저장된다.

HeapObject 는 크게 2 가지 영역을 갖는데 메모리 내에 인스턴스화 된 객체에 접근하는 isa, Reference Count 를 저장하는 InlineRefCounts 로 구성된다.

즉, Reference Count 는 HeapObject 의 InlineRefCounts 에 저장된다.

* 객체를 강하게 혹은 미소유 참조할 경우(Strong/Unowned Reference) 발생하는 참조 카운트는 객체 자체에 저장된다.
* 객체를 약하게 참조할 경우(Weak Reference) 발생하는 참조 카운트는 객체 자체가 아닌, Side-Table 에 저장된다.

## Side Table 간단 설명

약한 참조가 발생할 경우 변수가 객체를 참조하는 참조 카운트를 저장하는 장소이다.

자세한 것은 [AutoReferenceCounting.md 의 Side Table](https://github.com/SangHwi-Back/iOS-Interviews/blob/main/LowLevel/ARC/AutoReferenceCounting.md#side-table) 참고.

## Swift Object Lifecycle

객체에 Strong 참조를 하여 Side-Table 없이 존재할 경우 Strong/Unowned/Weak 참조를 각각 하나씩 갖는다.

* Unowned 참조는 Strong 참조에 의해 extra 로 하나 늘어난다.
* Weak 참조는 Unowned 참조해 의해 extra 로 하나 늘어난다.
* 이 상태에서는 강한/미소유 참조로 이 객체를 참조해도 문제가 없다.

이 경우 객체는 더 이상 사용이 되지 않는다고 판단될 경우 Strong 참조가 하나 줄어들면서 메모리에서 해제된다.

Weak 참조로 객체를 참조할 경우에는 객체를 직접 참조하지 않고 sideTableEntry 를 참조한다. 그리고 참조가 발생한 실행 Blcok 등이 끝났을 때 강한 참조가 없는 객체는 메모리에서 해제된다. 

예시로 다음같은 상황을 생각해보자.

1. UITableView 를 UIViewController 내에서 생성했다.
2. delegate/datasource 프로퍼티에 UITableView 를 소유한 UIViewController 가 아닌 외부 클래스를 사용하려 한다.
3. UIViewController 에 delegate/datasource 객체를 프로퍼티로 생성하는 것이 아니라 생성자를 이용해서 바로 UITableView 에 반영해준다.
4. delegate/datasource 가 예상대로 동작하지 않는다. 디버깅을 해보니 반영한 줄 알았던 UITableView 의 delegate/datasource 프로퍼티가 nil 이다.

## 결론

Swift 는 객체의 Reference Count 를 통해 앱의 메모리를 관리한다. Reference Count 는 객체가 참조될 때마다 하나씩 증가하고 참조가 더 이상 쓸모없다고 판단되면 하나씩 감소한다. Reference Count 가 0에 도달하면 메모리에서 해제된다.

한 객체의 Reference Count 는 객체에 저장된다. Strong/Unowned 참조는 객체에 직접 저장되고 Weak 참조는 객체의 Side-Table 에 저장된다.

즉, Strong/Unowned 참조는 객체를 직접 참조하지만 Weak 참조는 객체의 Side-Table 을 참조하는 것이다.

## 애플 깃허브에서 찾아낸 Reference Count 주석 (2022/12/20 확인 후 작성)

```objectivec
/*
  An object conceptually has three refcounts. These refcounts 
  are stored either "inline" in the field following the isa
  or in a "side table entry" pointed to by the field following the isa.
  
  The strong RC counts strong references to the object. When the strong RC 
  reaches zero the object is deinited, unowned reference reads become errors, 
  and weak reference reads become nil. The strong RC is stored as an extra
  count: when the physical field is 0 the logical value is 1.
  The unowned RC counts unowned references to the object. The unowned RC 
  also has an extra +1 on behalf of the strong references; this +1 is 
  decremented after deinit completes. When the unowned RC reaches zero 
  the object's allocation is freed.
  The weak RC counts weak references to the object. The weak RC also has an 
  extra +1 on behalf of the unowned references; this +1 is decremented 
  after the object's allocation is freed. When the weak RC reaches zero 
  the object's side table entry is freed.
  Objects initially start with no side table. They can gain a side table when:
  * a weak reference is formed 
  and pending future implementation:
  * strong RC or unowned RC overflows (inline RCs will be small on 32-bit)
  * associated object storage is needed on an object
  * etc
  Gaining a side table entry is a one-way operation; an object with a side 
  table entry never loses it. This prevents some thread races.
  Strong and unowned variables point at the object.
  Weak variables point at the object's side table.
  Storage layout:
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
  InlineRefCounts and SideTableRefCounts share some implementation
  via RefCounts<T>.
  InlineRefCountBits and SideTableRefCountBits share some implementation
  via RefCountBitsT<bool>.
  In general: The InlineRefCounts implementation tries to perform the 
  operation inline. If the object has a side table it calls the 
  HeapObjectSideTableEntry implementation which in turn calls the 
  SideTableRefCounts implementation. 
  Downside: this code is a bit twisted.
  Upside: this code has less duplication than it might otherwise
  Object lifecycle state machine:
  LIVE without side table
  The object is alive.
  Object's refcounts are initialized as 1 strong, 1 unowned, 1 weak.
  No side table. No weak RC storage.
  Strong variable operations work normally. 
  Unowned variable operations work normally.
  Weak variable load can't happen.
  Weak variable store adds the side table, becoming LIVE with side table.
  When the strong RC reaches zero deinit() is called and the object 
    becomes DEINITING.
  LIVE with side table
  Weak variable operations work normally.
  Everything else is the same as LIVE.
  DEINITING without side table
  deinit() is in progress on the object.
  Strong variable operations have no effect.
  Unowned variable load halts in swift_abortRetainUnowned().
  Unowned variable store works normally.
  Weak variable load can't happen.
  Weak variable store stores nil.
  When deinit() completes, the generated code calls swift_deallocObject. 
    swift_deallocObject calls canBeFreedNow() checking for the fast path 
    of no weak or unowned references. 
    If canBeFreedNow() the object is freed and it becomes DEAD. 
    Otherwise, it decrements the unowned RC and the object becomes DEINITED.
  DEINITING with side table
  Weak variable load returns nil. 
  Weak variable store stores nil.
  canBeFreedNow() is always false, so it never transitions directly to DEAD.
  Everything else is the same as DEINITING.
  DEINITED without side table
  deinit() has completed but there are unowned references outstanding.
  Strong variable operations can't happen.
  Unowned variable store can't happen.
  Unowned variable load halts in swift_abortRetainUnowned().
  Weak variable operations can't happen.
  When the unowned RC reaches zero, the object is freed and it becomes DEAD.
  DEINITED with side table
  Weak variable load returns nil.
  Weak variable store can't happen.
  When the unowned RC reaches zero, the object is freed, the weak RC is 
    decremented, and the object becomes FREED.
  Everything else is the same as DEINITED.
  FREED without side table
  This state never happens.
  FREED with side table
  The object is freed but there are weak refs to the side table outstanding.
  Strong variable operations can't happen.
  Unowned variable operations can't happen.
  Weak variable load returns nil.
  Weak variable store can't happen.
  When the weak RC reaches zero, the side table entry is freed and 
    the object becomes DEAD.
  DEAD
  The object and its side table are gone.
*/
```

[출처는 여기](https://github.com/apple/swift/blob/d1c87f3c936c41418ee93320e42d523b3f51b6df/stdlib/public/SwiftShims/RefCount.h)
