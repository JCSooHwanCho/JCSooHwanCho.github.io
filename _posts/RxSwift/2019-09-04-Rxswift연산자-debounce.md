---
layout: post
title: RxSwift연산자-debounce
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 연속된 이벤트를 하나로 처리할 수 있는 debounce 연산자에 대해서 알아보겠습니다.

![debounce]({{'/img/debounce.png'}}){: .center-block :}  

debounce 연산자의 선언은 다음과 같습니다.

```swift
  public func debounce(_ dueTime: RxTimeInterval, scheduler: SchedulerType) -> Observable<Element>
```  

dueTime은 RxTimeInterval타입으로, 이는 [DispatchTimeInterval](https://developer.apple.com/documentation/dispatch/dispatchtimeinterval)의 타입앨리어스입니다. 이벤트가 발생했을 때, 바로 이벤트를 발생시키지 않고 대기하는 시간을 의미합니다.

scheduler는 dueTime만큼 기다리기 위한 타이머를 동작시킬 스케쥴러를 나타냅니다. 타이머가 만료되면, 해당 스케쥴러에서 이벤트가 나오게 됩니다. 

debounce연산자는 이벤트가 발생하면, 이를 바로 발생시키는 것이 아니라 인자로 주어진 스케쥴러에 인자로 주어진 시간이 지나면 이벤트가 발생하도록 스케쥴링 합니다. 타임아웃이 되기전에 이벤트가 발생하는지에 따라 다음과 같이 나뉩니다.

1. 타임아웃 되기 전에 이벤트가 발생할 경우 : 이전 이벤트는 취소되고, 새로운 이벤트가 스케쥴러에 스케쥴링 됩니다.

2. 타임아웃 되기 전에 이벤트가 발생하지 않을 경우 : 이벤트가 발생합니다.

3. error가 발생할 경우 : 타이머 설정된 이벤트가 방출되지 않고 종료됩니다.

4. completed가 발생할 경우 : 타이머 설정된 이벤트가 즉시 방출되고 종료됩니다.  

```swift
let timer = Observable<Int>.interval(RxTimeInterval.seconds(4), scheduler: MainScheduler.instance) // 4초마다 1씩 증가하는 이벤트를 MainScheduler에서 발생시킵니다.

timer.debounce(RxTimeInterval.seconds(3), scheduler: MainScheduler.instance) 
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
// ...
```  

debounce는 연속적인 UI 입력을 받았을 때, 실제 요청은 한번만 처리하게 한다던지 하는 곳에서 유용하게 사용할 수 있습니다.