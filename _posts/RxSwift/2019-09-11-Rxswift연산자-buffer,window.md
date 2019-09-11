---
layout: post
title: RxSwift연산자-buffer,window
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 이벤트를 묶음으로 전달하는 buffer와 window 연산자에 대해서 알아보겠습니다. 

![buffer]({{"/img/Buffer.png"}}){: .center-block :}  
![groupBy]({{"/img/window.png"}}){: .center-block :}  

buffer와 window의 선언은 다음과 같습니다.

```swift
  public func buffer(timeSpan: RxTimeInterval, count: Int, scheduler: SchedulerType) -> Observable<[Element]>

   public func window(timeSpan: RxTimeInterval, count: Int, scheduler: SchedulerType) -> Observable<Observable<Element>>
```  
보다시피 두 연산자의 인자는 완전히 동일합니다. 다만 반환값이 다를 뿐이죠.

buffer나 window모두 다음 두가지 조건 중 1가지가 만족되기 전까지는 이벤트를 방출하지 않습니다.

1. 첫 구독 시점 혹은 가장 최근 이벤트 발생 후로 timespan으로 지정한 시간이 지났다(타임 아웃)
2. 첫 구독 시점 혹은 가장 최근 이벤트 발생 후로 원본 Observable에서 count만큼의 이벤트가 발생했다.  

이 두가지 조건중 하나가 만족되기 전까지는, 원본 Observable의 이벤트는 내부 버퍼에 저장됩니다. 그러다 이벤트 발생 시점이 되면, 해당 버퍼의 이벤트들을 모두 내보내고, 버퍼를 비우고 타이머를 다시 시작합니다. 

buffer와 window의 차이는 단지 이 버퍼가 Array의 형태로 방출되는가 또 다른 Observable의 형태로 방출되는 가의 차이입니다.

```swift
let timer1 = Observable<Int>.interval(RxTimeInterval.seconds(1), scheduler: MainScheduler.instance).map({"o1: \($0)"})

timer1.buffer(timeSpan: RxTimeInterval.seconds(3), count: 2, scheduler: MainScheduler.instance)
    .subscribe { event in
    switch event {
    case let .next(value):
        print(value)
    default:
        print("finished")
    }
    
    }.disposed(by: bag)
// ["o1: 0", "o1: 1"]
// ["o1: 2", "o1: 3"]
// ["o1: 4", "o1: 5"]
// ....

timer.window(timeSpan: RxTimeInterval.seconds(3), count: 2, scheduler: MainScheduler.instance)
    .subscribe { event in
        switch event {
        case let .next(observable):
            observable.subscribe { e in
                switch e {
                case let .next(value):
                    print(value)
                default:
                    print("inner finished")
                }
            }
        default:
            print("finished")
        }
        
    }.disposed(by: bag)

timer.window(timeSpan: RxTimeInterval.seconds(3), count: 2, scheduler: MainScheduler.instance)
    .subscribe { event in
        switch event {
        case let .next(observable):
            observable.subscribe { e in
                switch e {
                case let .next(value):
                    print(value)
                default:
                    print("inner finished")
                }
            }
        default:
            print("finished")
        }
        
    }.disposed(by: bag)
// o1: 0
// o1: 1
// inner finished
// o1: 2
// o1: 3
// inner finished
// o1: 4
// o1: 5
// inner finished
// ...
```  

buffer나 window를 통해서 이벤트를 특정 횟수나 시간의 단위로 묶어서 한꺼번에 처리할 수 있습니다.