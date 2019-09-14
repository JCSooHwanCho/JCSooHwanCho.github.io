---
layout: post
title: RxSwift연산자-combineLatest
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 여러개의 Observable들의 최근 값들을 받아서 처리할 수 있는 combineLatest 연산자에 대해서 알아보겠습니다.

![combineLatest]({{"/img/CombineLatest.png"}}){: .center-block :}  

combineLatest는 합치는 Observable에 수에 따라 별도로 정의되어 있습니다. 그 중 최소인 2개의 Observable을 받는 연산자의 선언은 다음과 같습니다.

```swift
 public static func combineLatest<O1: ObservableType, O2: ObservableType>
        (_ source1: O1, _ source2: O2, resultSelector: @escaping (O1.Element, O2.Element) throws -> Element)
            -> Observable<Element>
```  

가변 인자로 정의되지 않은 이유를 여기서 알 수 있는데, combineLatest로 합치는 Observable들의 타입이 서로 다를 수 있기 때문입니다. Rxswift는 단일 연산자로 총 8개까지 합성할 수 있도록 제공하고 있습니다.

combineLatest의 인자로 들어간 두 Observable이 내보낸 이벤트는 가장 최근값에 한해서 내부적으로 저장이 됩니다. 두 Observable이 모두 1번 이상 이벤트를 내보내야만 CombineLatest를 통해 이벤트를 받아볼 수 있는데, 이 때 두 Observable의 최근 값을 함께 내보냅니다. 이렇게 나온 값은 바로 구독자에게 전달되지 않고, resultSelector를 거쳐서 가공된 후에야 비로소 구독자들에게 전달됩니다.

> 만약 두 Observable이 속도가 안 맞아서 한 Observable이 이벤트를 방출하기 전에 다른 Observable이 여러개의 이벤트를 방출해버린다면, 최근 값 1개만 남기고는 받아볼 수 없습니다.

같은 타입을 수 제한 없이 합치기 위해 Observable의 Collection을 인자로 받는 combineLatest도 있으니, 필요하다면 이것도 고려해볼만 합니다. 이 경우는 가장 최근 값들이 Array에 담겨 나오며, 이 값들의 순서는 Collection애서의 Observable의 순서와 일치하게 됩니다.

```swift
    public static func combineLatest<Collection: Swift.Collection>(_ collection: Collection, resultSelector: @escaping ([Collection.Element.Element]) throws -> Element)
     -> Observable<Element> where Collection.Element: ObservableType
```  

```swift
let timer1 = Observable<Int>.interval(RxTimeInterval.seconds(1), scheduler: MainScheduler.instance).map({"o1: \($0)"})
let timer2 = Observable<Int>.interval(RxTimeInterval.seconds(2), scheduler: MainScheduler.instance).map({"o2: \($0)"})

Observable.combineLatest(timer1,timer2){ v1, v2 in
        v1+" "+v2
    }
    .subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)
// o1: 1 o2: 0
// o1: 2 o2: 0
// o1: 3 o2: 0
// o1: 3 o2: 1
// o1: 4 o2: 1
// o1: 5 o2: 1
// o1: 5 o2: 2
// o1: 6 o2: 2
// ...
``` 

여러 요소들의 최근 상태를 동시에 체크해야만 할 때, CombineLatest가 유용하게 쓰일 수 있습니다.