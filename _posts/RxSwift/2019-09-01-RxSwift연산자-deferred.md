---
layout: post
title: RxSwift연산자-deferred
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 외부 조건에 따라 다른 Observable을 제공해 주는 [deferred](http://reactivex.io/documentation/operators/defer.html)연산자에 대해 알아보겠습니다.

![deferred]({{'/img/deferred.png'}}){: .center-block :} 

deferred 연산자의 선언부는 다음과 같습니다.

```swift
public static func deferred(_ observableFactory: @escaping () throws -> Observable<Element>)
```  

deferred 연산자는 Observable을 만들어내는 팩토리 클로저를 인자로 받습니다. 그리고 실제 구독이 일어나는 시점에서야 실제 Observable을 만들어냅니다. 즉, 실제 Observable이 만들어지는 시점이 미루어진다고 해서 'deferred' 입니다.

```swift
var observableSwitch:Bool = false

let observable = Observable<Int>.deferred {
    observableSwitch.toggle()
    
    if observableSwitch {
        return Observable.from([1,2,3])
    } else {
        return Observable.from([4,5,6])
    }
}

observable.subscribe { event in
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
observable.subscribe { event in
    switch event {
    case let .next(value):
        print(value)
    default:
        print("finished")
    }
    
    }.disposed(by: bag)
// 4
// 5
// 6
// finished
``` 

이런식으로 deferred 연산자를 통해 상태에 따라 다른 데이터를 처리해야할 필요가 있을 경우를 대처할 수 있습니다.