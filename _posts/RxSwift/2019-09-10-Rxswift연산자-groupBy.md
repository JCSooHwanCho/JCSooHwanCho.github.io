---
layout: post
title: RxSwift연산자-groupBy
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 이벤트를 분류하여 Observable로 만들어 내보내는 GroupBy 연산자에 대해 알아보겠습니다.

![groupBy]({{"/img/GroupBy.png"}}) 

GroupBy의 선언은 다음과 같습니다.

```swift
    public func groupBy<Key: Hashable>(keySelector: @escaping (Element) throws -> Key) -> Observable<GroupedObservable<Key, Element>>  
```  

GroupedObservable이라는 것이 생소할 수 있지만, 단순히 Observable에 키(key)를 추가로 가진 것입니다. 이 키를 통해 Observable을 구분할 수 있습니다.  

GroupBy는 keySelector라는 것을 인자로 받는데, 이는 이벤트의 값을 보고 해당 이벤트를 특정 키와 연결해주는 역할을 합니다. 그리고 이 키를 가진 Observable을 통해서 이벤트가 방출되게 됩니다. 

이렇게 그룹으로 묶인 Observable들은 마지막에 하나의 Observable을 통해서 방출되게 되고, 이를 구독하게 되면 Observable을 값으로 받아볼 수 있게 됩니다.

```swift
Observable.from([1,2,3,4,5,6,7,8,9,10])
    .groupBy(keySelector: { i -> String in
        if i%2 == 0 {
            return "odd"
        } else {
            return "even"
        }
    } ).flatMap { o -> Observable<String> in // Observable들을 모두 하나로 합쳐줍니다.
        if o.key == "odd" {
            return o.map { v in
                "odd \(v)"
            }
        } else {
                return o.map { v in
                    "even \(v)"
                }
            }
        }
    .subscribe { event in
        switch event {
        case let .next(value):
           print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)
// odd 1
// even 2
// odd 3
// even 4
// odd 5
// even 6
// odd 7
// even 8
// odd 9
// even 10
// finished
```  

GroupBy연산을 통해 이벤트들을 분류해서 별도의 Observable로 만들어 내보내는 작업을 손쉽게 할 수 있습니다.