---
layout: post
title: RxSwift연산자-skip
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 take와는 반대로 이벤트의 발생을 특정 수만큼 억누르는 skip 연산자에 대해서 알아보겠습니다.

![skip]({{'/img/skip.png'}}){: .center-block :}  

skip 연산자의 선언은 다음과 같습니다.

```swift
public func skip(_ count: Int) -> Observable<Element>
```  

skip은 count로 지정한 수만큼의 이벤트를 내보내지 않고 없애버립니다. 이 지정한 수 이상의 이벤트는 그대로 내보냅니다. 이런면에서 take에 정확히 반대되는 연산이라고 볼 수 있습니다. 또한 take와 마찬가지로 error나 completed가 발생하면 당연히 observable이 종료됩니다.  

take와 대비되는 만큼 skip의 변형 역시 take와 거의 비슷합니다. 완전히 같지는 않습니다.

```swift
public func skip(_ duration: RxTimeInterval, scheduler: SchedulerType) -> Observable<Element> // 특정 시간간격동안 발생한 이벤트를 무시합니다.  

public func skipUntil<Source: ObservableType>(_ other: Source) -> Observable<Element> // 지정한 Observable이 이벤트를 방출해야만 자신도 이벤트를 방출합니다.

public func skipWhile(_ predicate: @escaping (Element) throws -> Bool) -> Observable<Element> // 판별식을 만족하는 동안은 이벤트를 방출하지 않습니다.

``` 

```swift
Observable.from([1,2,3,4,5,6,7,8,9,10])
    .skip(7)
    .subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)
// 8
// 9
// 10
``` 

skip을 통해 take와는 반대의 방법으로 이벤트 수를 제한할 수 있습니다.