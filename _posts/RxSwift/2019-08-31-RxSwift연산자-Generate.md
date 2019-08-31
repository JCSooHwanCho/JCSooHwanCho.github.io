---
layout: post
title: RxSwift연산자-Generate
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 for 반복문에 대응하는 generate 연산자에 대해서 알아보겠습니다.

generate의 함수 선언은 다음과 같습니다.

```swift
public static func generate(initialState: Element, condition: @escaping (Element) throws -> Bool, scheduler: ImmediateSchedulerType = CurrentThreadScheduler.instance, iterate: @escaping (Element) throws -> Element) -> Observable<Element>
``` 

선언이 굉장히 길지만 쪼개서 보면 다음과 같은 인자를 받습니다.

1. initialState : 초기 상태
2. condition : 현재 상태를 검사하여, true를 반환하면 Observable 을 진행시키고, false를 반환하면 Observable을 종료시킵니다.
3. scheduler : 어떤 scheduler에서 실행시킬 지 선택합니다. 기본 값은 현재 스레드입니다.
4. iterate : 이벤트를 발생시킨 후 매번 실행되는 명령입니다.

여기서 scheduler를 제외한다면, 우리는 이미 위와 같은 순서를 사용하는 구문을 알고 있습니다. 바로 C 계열 언어의 for문입니다.


```c
int i;
for(i=0;i<10;i++) {
    // 작업을 수행합니다.
}
``` 

이처럼 generate함수는 상태값을 가지는 루프를 수행하며 이벤트를 내보내는 Observable을 만드는 데 사용합니다. 

```swift
Observable.generate(initialState: 0,
                    condition: { $0 < 10},
                    iterate: { $0+1 })
    .subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)
// 0
// 1
// 2
// 3
// 4
// 5
// 6
// 7
// 8
// 9
// finished
```  

기존의 인덱스 기반 for문 등을 RxSwift로 포팅해야할 때 유용하게 사용할 수 있을 것입니다.