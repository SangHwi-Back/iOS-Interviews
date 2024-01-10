# iOS 의 메모리

> 공식문서 아카이브의 `About the Virtual Memory System` 아티클을 번역하였습니다.

`VM(Virtual Memory)`을 이해해야 하는 이유는 적절한 메모리 사용을 위한 코드가 메모리 사용량 뿐만 아니라 CPU 사용량도 효율적으로 만들어주기 때문입니다.

iOS/OS X의 fully-integrated virtual memory system 은 개발자나 사용자가 임의로 조작할 수는 없다. 언제나 켜져 있는 것이다. 32bit 프로세서에서는 경우 4GB 의 시스템을 갖습니다. OS X 의 경우 64비트 프로세서에서 18 exabytes 의 시스템을 갖습니다. 4GB 이상의 램 용량을 가진 컴퓨터의 경우 드물게 램 용량만큼 시스템도 램 용량만큼을 단일 프로세서에서 지원하곤 합니다.

OS X 의 경우 메모리가 꽉 차게 되면 데이터가 필요한 공간을 마련하기 위해 사용되지 않는 하드디스크 영역을 이용합니다. 하드디스크 중 사용되지 않는 데이터를 저장하는 곳을 `backing-store` 라고 합니다. main memory 의 backup storage 를 제공하기 때문입니다.

iOS 는 backing store 를 제공하지 않습니다. iPhone 앱에서는 code page 와 같은 디스크로부터 불려와진 읽기 전용 데이터가 메모리에 의해 해제되거나 불려와지게 됩니다. 쓰기 데이터는 반대로 iOS 에 의해 메모리에서 해제되는 것을 막습니다. 대신 메모리 공간이 특정 시점까지 떨어지게 되면 시스템(iOS)은 구동중인 앱들에게 자발적으로 메모리를 해제시켜서 새로운 데이터를 위한 공간을 만들 것을 지시합니다. 이러한 지시에도 불구하고 필요한 만큼의 메모리를 확보하는 데 실패하는 앱들은 강제 종료됩니다.

**NOTE**: UNIX-기반 운영체제들과는 달리, OS X 는 backing store 를 위해 디스크 파티션을 미리 확보해놓지 않습니다. 대신에, 각 기기마다 운용 가능한 boot partition 만을 사용합니다.

아래 섹션들은 용어를 설명하고 iOS/OS X의 VM 에 대한 간략한 정의를 소개한다. VM 시스템이 동작하는 더 자세한 정보는 [Kernel Programming Guide](https://developer.apple.com/library/archive/documentation/Darwin/Conceptual/KernelProgramming/About/About.html#//apple_ref/doc/uid/TP30000905) 를 참고하라.

## About Virtual Memory

VM 은 운영체제가 물리적인 램의 한계를 뛰어넘게 해줍니다.

VM 매니저는 각 프로세스를 위한 논리적인 주소 공간을 생성한 뒤 일정한 크기로 나누는데 이를 `pages` 라고 부릅니다. 프로세서와 `Memory-Management-Unit(MMU)`는 `page-table` 을 통해 프로그램의 논리적인 주소공간인 pages 와 램의 하드웨어 주소공간과 매핑하고 관리합니다.

프로그램의 코드가 메모리의 주소에 할당될 경우, MMU 는 page table 을 사용하여 논리적인 주소공간을 토대로 실제 하드웨어 메모리 주소공간으로 치환합니다. 이러한 치환은 자동으로 모든 앱들을 통틀어 발생합니다.

프로그램이 관리되는 한, 논리적인 주소공간에 의해 할당된 메모리 공간의 주소는 항상 사용 가능합니다. 이 상황에서 앱이 실제 메모리 공간에는 존재하지 않는 주소를 참조하면 이를 `page-fault` 라고 합니다.

page fault 가 발생하면 VM 시스템은 특별한 `page-fault-handler` 를 호출하여 이 현상을 해결하게 됩니다. page-fault-handler 는 page-fault 를 발생시킨 현재 코드를 중지시키고 실제 메모리의 빈 page 로 이동하여 필요한 만큼의 데이터를 디스크에서 호출합니다. 그리고 page-table 을 업데이트 합니다. 마지막으로 제어권을 프로그램의 코드에게 넘겨주어서 평소처럼 메모리 주소에 접근할 수 있도록 합니다.

이런 일련의 과정을 `paging` 이라고 합니다.

만약 실제 메모리에 여유 page 가 없다면, handler 는 반드시 현재 매핑된 page 들 중 맨 첫번째부터 release 하여 새로운 page 를 위한 공간을 만들어줘야 합니다. 시스템이 pages 를 release 하는 방식은 플랫폼에 따라 다릅니다.

OS X는 VM 시스템이 page 들을 backing store 에 할당합니다. backing store 는 디스크 기반의 저장소로 특정 프로세스의 메모리 pages 들을 보유하고 있습니다. 실제 메모리에서 backing store 로 데이터가 이동하는 것을 `paging-out(swapping out)` 이라고 하고, backing store 에서 실제 메모리로 이동하는 것을 `paging-in(swapping in)` 이라고 합니다.

iOS 에서는 backing store 가 없기 때문에 paging-out 은 존재하지 않지만, 읽기 전용 paging-in 을 통해 실제 메모리에 paged 됩니다.

OS X 나 이전 버전의 iOS 에서 page 의 크기는 4KB 였습니다. 이후 버전의 iOS(A7, A8 칩 기반의 시스템)는 4KB 페이지 기반의 16KB 페이지를 64비트 사용자환경에 제공합니다. 반면에 A9기반 시스템 이후부터는 16KB 페이지를 16KB 페이지에 기반하여 제공합니다. 이 page size 들은 page-fault 시 시스템에서 얼마나 많은 KB 를 읽어들이는지에 따라 다릅니다. `Disk-thrashing` 은 시스템이 프로그램에 쏟는 시간 대비 page-fault 와 paging 에 시간을 쏟는 경우 발생합니다.

paging 과 Disk-thrashing 은 시스템이 너무 많은 시간을 디스크를 읽고 쓰는데 보내도록 하기 때문에 성능에 부정적인 영향을 미칩니다. paging-in 은 실제 RAM 에서 직접 데이터를 읽는 것보다 훨씬 느립니다. 만약 시스템이 page 를 디스크에서부터 읽어들이기 보다는 디스크에 기록하고자 할 때, 성능은 더욱 낮아질 것입니다.

## Details of the Virtual Memory System

프로세스의 논리적인 주소공간은 매핑된 실제 메모리 구역으로 이루어져 있다고 할 수 있습니다. 매핑된 각 영역은 VM 의 page 갯수 정보를 포함하고 있습니다.

각 구역은 Inheritance(parent 구역을 매핑하는 구역), Write-Protection, 그 외의 wired 상태(예를 들어 paging-out 이 되지 않는 상태)를 관리하는 속성들로 구성되어 있습니다.

각 구역은 이미 정해진 수의 페이지가 포함되어 있으므로 page-aligned 라고 불리기도 합니다. 이는 영역의 시작 메모리 주소가 page 들의 시작 주소이고 끝 주소가 page 들의 끝 주소와 같다는 것입니다.

kernel 은 VM 객체를 논리적인 주소 공간에 위치시킵니다. kernel 은 VM 객체가 상주하거나 상주하지 않는 page 를 추적하고 관리합니다.

논리 주소공간은 backing store 나 메모리 매핑된 파일시스템에 매핑될 수 있습니다. VM 객체는 기본 pager 나 vnode pager 가 위치한 논리적인 주소공간에 대한 매핑 정보를 포함하고 있습니다. 

기본 pager 는 시스템 매니저로서 backing store 에 비상주하고 있는 VM page 들을 관리하고, 요청이 들어오면 backing store 로 상주시킵니다. vnode pager 는 메모리 매핑이 된 파일에 대한 접근 권한을 구현하고 있습니다. vnode pager 는 paging 매커니즘을 이용하여 파일에 접근하는 창구를 제공한다. 이 매커니즘은 파일이 메모리에 있는 것처럼 파일의 일부를 읽고 쓸 수 있게 도와줍니다.

구역을 기본 pager 혹은 vnode pager 로 매핑하는 것 말고도 VM 객체는 구역을 다른 VM 객체와 매핑해야 합니다. kernel 은 자기 참조 기술을 이용하여 copy-on-write 기능을 구현합니다. copy-on-write 구역은 다른 프로세스가 page를 쓰지 않는 한 page를 공유할 수 있도록 하는 것이다. 프로세스가 page를 사용하려고 할 때, copy of the page가 논리적인 주소 공간에 만들어진 뒤 사용하게 된다. 여기에 더해, 쓰기 프로세스는 프로세스가 관리하는 수많은 각각의 page 카피를 갖고 있다. 이 page 카피는 프로세스에 의해 언제든 사용될 수 있다. Copy-on-write 구역은 시스템이 많은 양의 데이터를 효율적으로 메모리 내에서 공유할 수 있게 해준다. 반면에 프로세스들에게 page들을 직접 혹은 간접적으로 필요할 때마다 사용할 수 있도록 한다. 이런 타입의 구역들은 시스템 프레임워크에 의해 불려와진다.

각 VM 객체는 다음의 필드를 포함한다.

| Field          | Description                                      |
|----------------|--------------------------------------------------|
| Resident pages | 구역에 포함되어 있고 실제 메모리에 상주한 page 들의 리스트              |
| Size           | 구역의 크기, byte                                     |
| Pager          | backing store 에 있는 page 들을 추적하고 다루는 책임을 가진 pager |
| Shadow         | copy-on-write 최적화를 위해 사용됨                        |
| Copy           | copy-on-write 최적화를 위해 사용됨                        |
| Attributes     | 구현 상세에 대한 다양한 상태를 나타내는 플래그                       |

VM 객체가 copy-on-write 작업에 연관되었을 경우, shadow/copy 필드가 다른 VM 객체를 가리키게 된다. 그렇지 않을 경우엔 이 필드는 단순히 NULL 이다.

## Wired memory

Wired 메모리(상주 메모리)는 paging out 되지 않는 kernel 코드와 자료구조를 저장하고 있다. 앱, 프레임워크, 사용자 레벨의 소프트웨어는 Wired 메모리를 할당할 수 없다. 그러나, 사용자 레벨은 언제든지 Wired 메모리 존재 여부에 영향을 끼칠 수 있다. 예를 들어, 스레드와 포트를 내부적으로 할당하는 애플리케이션은 Wired 메모리의 필요한 kernel 자원 중 관련된 자원에 메모리를 할당한다

아래 표는 앱 기반 엔티티에 의해 Wired 메모리가 할당되는 양을 나타낸 것이다. 해당 값은 바뀔 수 있다.

Resource	Wired Memory Used by kernel
Process	16 KB
Thread	연속 시 5 KB, 21 KB 시 차단.
Mach port	116 bytes
Mapping	32 bytes
Library	2 KB + ( 200 bytes * n tasks )
Memory region	160 bytes

만약 당신의 앱이 Wired 메모리를 사용한다면 kernel은 Wired 메모리의 다음 엔티티들을 요구할 것이다.

VM 객체
VM 버퍼 캐시
I/O 버퍼 캐시
드라이버

Wired 자료구조 또한 실제 page와 VM 매핑 정보를 저장한 매핑 테이블과 연관이 있다. 앞의 두 엔티티들은 실제 메모리의 가용량에 따라 크기가 결정된다. 결과적으로 메모리를 시스템에 적용할 때, Wired 메모리는 바뀌는 것이 없을지라도 증가하게 된다. 컴퓨터가 Finder에 부팅될 때(아무런 앱도 작동하지 않는 시점) Wired 메모리는 약 64 megabyte 시스템의 14 megabyte를 사용하고 128 megabyte 시스템의 17 megabyte를 사용한다.

Wired 메모리 page들은 비활성화 되었다고 하여 바로 해제되지 않는다. 대신 page out 이벤트를 유발하는 특정 지점까지 free-page count가 떨어져서 "garbage collected" 된다.

## Page Lists in the Kernel

kernel은 실제 메모리 3 개의 system-wide lists 를 관리하고 요청한다.

- active list = 최근 메모리에 매핑되었고 최근에 메모리에 접근한 page들.
- inactive list = 최근 실제 메모리에 상주하였지만 메모리에 접근은 하지 않은 page들. 이 page들은 유효한 데이터를 포함하지만 메모리에서 언제든지 제외될 수 있다.
- free list = 실제 메모리의 VM 객체와 연관이 없는 page들. 이 page들은 해당 page들을 필요로 하는 다른 프로세스에 의해 사용될 수 있다.

pager는 free list의 page들이 특정 시점이 되었을 때 Queue balancing을 시도하는데, page들을 inactive list로부터 뽑아내는 방식으로 진행한다. page가 최근 접근되었다면 다시 활성화되고 active list의 마지막에 위치한다. OS X에서는 inactive page가 backing store에 최근까지 포함되지 않았다면 inactive page의 컨텐츠는 디스크로 page out 되어야 free-list로 이동이 가능하다(iOS 에서는 사용되었지만 비활성화 된 page들은 반드시 메모리에 남아있어야 하고 page를 소유한 앱에 의해 정리되어야 한다). 비활성화 된 page가 최근까지 사용되지 않았고 계속 상주된 상태라면, 해당 page에 매핑된 모든 정보가 사라지고 free list로 이동한다. free list의 크기가 초과되면 pager는 중단된다.

kernel은 page가 사용되지 않을 경우 active list에서 inactive list로 이동시킨다. 반대로 inactive list에서 active list로 옮기는 것을 soft fault 라고 한다. page가 swapped out(paging out)되면 관련 실제 page들은 free list로 이동한다. 또한, 프로세스가 여러 메모리를 해제시키면 kernel은 관련 page들을 모두 free list로 이동시킨다.

## Paging Out Process

OS X 에서 free list의 page들이 특정 시점이 되면 kernel은 inactive list의 메모리 초과량만큼을 스와핑할 것을 실제 page들에게 요청한다. 이를 위해서 kernel은 모든 상주 page들을 순회하여 active에서 inactive list로 이동시키기 위해 다음 단계를 거친다.

1. active list의 page가 최근 사용되지 않았다면 inactive list로 이동한다.
2. inactive list의 page가 최근 사용되지 않았다면 kernel은 이 page의 VM 객체를 찾는다.
3. VM 객체가 매핑된 적이 없을 경우 kernel은 초기화 루틴을 호출하여 기본 pager 객체를 호출한다.
4. VM 객체의 기본 pager는 page를 backing store 외의 공간에 page를 저장한다.
5. pager가 작업을 성공하면 kernel은 실제 메모리 중 위의 page가 차지하고 있는 공간을 해제하고 page를 inactive list에서 free list로 이동시킨다.

iOS 에서는 kernel은 page를 backing store 외의 공간에 저장하지 못한다. 가용 메모리가 특정 시점에 도달하게 되면 kernel은 inactive하거나 수정된 적이 없는 page들을 방출한다. 그리고 실행중인 앱들에게 메모리를 해제하라고 지시한다. 자세한 사항은 Responding to Low-Memory Warnings in iOS를 봐라.

## Paging In Process

Virtual Memory 관리의 마지막 단계는 page를 backing store나 file이 포함하고 있는 page data에서 실제 메모리로 이동시키는 것이다. 이런 과정은 메모리 접근 오류 때문에 발생한다. 메모리 접근 오류는 코드가 실제 메모리와 매핑되지 않은 가상 주소에 있는 데이터에 접근하려고 하면 발생한다. 메모리 접근 오류의 종류는 2가지이다.

1. soft fault = page가 참조하는 주소가 실제 메모리의 상주 page 주소이지만 특정 프로세스에서 매핑이 잘못된 경우이다.
2. hard fault = page가 참조하는 주소가 실제 메모리의 상주 page 주소가 아니고 backing store에서 paging out(swapped out)되어 실제 메모리에 매핑되지 않은 (매핑이 가능한) 상태에서 발생한다. page fault라고도 한다.

이런 오류가 발생하면 kernel은 액세스 가능한 지역으로부터 매핑 정보와 VM 객체를 찾는다. kernel은 VM 상주 page의 VM 객체 목록을 점검한다. 원하는 page가 상주 page list에 존재할 경우, kernel은 soft fault로 간주한다. 반대로 list에 존재하지 않을 경우 hard fault를 발생시킨다.

soft fault일 때 kernel은 page를 포함한 실제 메모리를 프로세스의 가상 주소 공간에 매핑시킨다. kernel은 특정 page를 활성화 상태로 마킹한다. soft fault가 쓰기 작업을 포함하고 있다면, page는 수정된 것으로 간주되고 곧 해제될 예정이라면 backing store로 이동할 것이다.

hard fault일 때 VM 객체의 pager는 page를 backing store나 디스크 내 file에서 찾아낸다. 매핑 정보를 적절히 수정한 후 pager는 page를 실제 메모리에 위치시키고 page를 active list에 위치시킨다. soft fault과 마찬가지로 쓰기 작업을 포함하고 있다면 수정된 것으로 마킹한다.

Reference:

* [Apple_Documentation_AboutMemory](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/ManagingMemory/Articles/AboutMemory.html)
