---
layout: post
title: RxSwift연산자-skip
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 이벤트를 변형해줄 수 있는 map 연산자에 대해서 알아보겠습니다.

![map]({{"/img/Map.png"}}){: .center-block :}  

map연산자의 선언은 다음과 같습니다.

```swift
 public func map<Result>(_ transform: @escaping (Element) throws -> Result) -> Observable<Result> 
``` 

transform은 이벤트가 가진 값을 인자로 받아서, Result 타입으로 변환하는 클로저입니다. Element는 Result와 같은 타입일 수도, 다른 타입일 수도 있습니다. 


```swift
Observable.from([1,2,3,4,5,6,7,8,9,10])
    .map({"num: \($0)"})
    .subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)

// num: 1
// num: 2
// num: 3
// ...
// num: 10
// finished
```  

map은 이벤트의 타입을 우리가 사용하기 편한 타입으로 가공하는데에 유용하게 사용됩니다.