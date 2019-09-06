---
layout: post
title: RxSwift연산자-take
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 이벤트의 갯수를 제한하는 take 연산자에 대해서 알아보겠습니다.

![take]({{'/img/take.png'}}){: .center-block :}  

take 연산자의 선언은 다음과 같습니다.

```swift
 public func take(_ count: Int) -> Observable<Element>
```  

take는 count로 지정한 만큼의 이벤트만 방출합니다. 지정한 수의 이벤트가 방출되면 곧바로 completed이벤트가 발생합니다. 물론 원본 Observable이 error나 complete가 나온다면 그 즉시 종료됩니다. 

take는 여러가지 변형을 제공하는데, 차례대로 나열해보면 다음과 같습니다.  

```swift
 public func take(_ duration: RxTimeInterval, scheduler: SchedulerType) -> Observable<Element> // 특정 시간 간격동안 나온 이벤트만 내보냅니다. 타임아웃될 경우 complete 이벤트가 나옵니다.
 
 public func takeUntil<Source: ObservableType>(_ other: Source) -> Observable<Element> // 인자로 넘긴 Observable이 이벤트를 방출하기 전까지만 이벤트를 내보냅니다.

 public func takeUntil(_ behavior: TakeUntilBehavior, predicate: @escaping (Element) throws -> Bool) -> Observable<Element> // 조건에 맞지 않는 이벤트가 발생할 때 까지만 이벤트를 내보냅니다. behavior는 마지막 이벤트(즉, predicate가 false를 반환하게 한 이벤트)를 내보낼지 아닐지를 결정하는 열거형 값입니다. [.inclusive, .exclusive]

public func takeWhile(_ predicate: @escaping (Element) throws -> Bool) -> Observable<Element> // takeUntil(.exclusive ,predicate) 와 동일한 연산입니다. 다만 원조는 takeWhile입니다.

public func takeLast(_ count: Int) -> Observable<Element> // 지정된 수만큼의 이벤트를 내부적으로 유지하고 있다가, complete될 때 이 이벤트를 순차적으로 내보내고 나서 complete됩니다. error가 발생할 경우는 이 이벤트들을 받아볼 수 없습니다.
```  

```swift
Observable.from([1,2,3,4,5,6,7,8,9,10])
    .take(3)
    .subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)
// 1
// 2
// 3
// finished
``` 
take 연산자를 통해 원하는 만큼만 이벤트를 제한할 수 있습니다.