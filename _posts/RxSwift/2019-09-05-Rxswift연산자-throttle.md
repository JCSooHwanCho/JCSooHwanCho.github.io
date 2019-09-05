---
layout: post
title: RxSwift연산자-throttle
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트는 이벤트가 발생하는 최소 간격을 제한할 수 있는 throttle 연산자에 대해서 알아보겠습니다.

![throttle]({{'/img/throttle.png'}}){: .center-block :}  

throttle 연산자의 선언부터 확인해보겠습니다.

```swift
    public func throttle(_ dueTime: RxTimeInterval, latest: Bool = true, scheduler: SchedulerType)-> Observable<Element>
```  

throttle연산은 쉽게 이해하기 어려운 부분이 있습니다. 차근차근 살펴보도록 하겠습니다.

1. 첫 이벤트가 발생할 경우, 이 이벤트는 무조건 구독자들에게 전달됩니다. 그리고 dueTime으로 지정한 시간만큼 이벤트 발생을 막습니다.

2. 타이머가 만료되기 전 이벤트가 발생할 경우에는 해당 이벤트는 발생되지 않습니다.

3. 타이머가 만료되었을 때는, 2번째 latest 인자에 따라 결과가 달라집니다.
   1. latest == true : 타이머가 돌아가던 중에 발생했던 이벤트들 중 가장 마지막 이벤트(즉, 최신 이벤트)가 전달됩니다. 이후 다시 타이머가 돌아갑니다. 만약 타이머가 돌아가던 도중 이벤트가 발생하지 않았다면 아무런 이벤트가 발생하지 않고, 다음 이벤트가 발생할 때에 그 이벤트를 내보내고 타이머가 돌아갑니다.
   
   2. latest == false : 타이머만 만료되고 아무일도 일어나지 않습니다. 이후에 이벤트가 발생하면, 해당 이벤트를 방출하고 타이머가 돌아갑니다.

이렇게 throttle을 활용하면 이벤트가 발생하는 간격은 최소 dueTime으로 지정한 시간만큼 벌어지게 됩니다. 

```swift
let timer = Observable<Int>.interval(RxTimeInterval.seconds(2), scheduler: MainScheduler.instance)

timer.throttle(RxTimeInterval.seconds(5), latest: true, scheduler: MainScheduler.instance)
    .subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)
// 0
// 2
// 5
// 7
// 10
// ...
```  
헷갈리기 쉽기 때문에 throttle을 사용할 때는 시간 간격을 잘 계산해서, 원하는 결과가 제대로 나올 수 있도록 해야합니다.