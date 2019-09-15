---
layout: post
title: RxSwift연산자-startWith
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 Observable의 앞에 이벤트를 덧붙일 수 있는 startWith 연산에 대해서 알아보도록 하겠습니다.

![startWith]({{"/img/startWith.png"}}){: .center-block :}  

startWith의 선언은 다음과 같습니다.

```swift
     public func startWith(_ elements: Element ...) -> Observable<Element>  
```  

elements는 가변인자로, 원하는 만큼 인자를 넘길 수 있습니다. 이렇게 넘긴 인자는 구독시 원본 Observable의 구독이 일어나기 전에 먼저 구독자에게 전달됩니다. 


```swift
let timer = Observable<Int>.interval(RxTimeInterval.seconds(1), scheduler: MainScheduler.instance).map({"o1: \($0)"})

timer.startWith("ready","go!")
    .subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)
// ready
// go!
// o1: 0
// o1: 1
// o1: 2
// o1: 3
// ...
```  

startWith 연산자를 통해여 원하는 값을 앞에 자유롭게 붙일 수 있습니다.