---
layout: post
title: RxSwift연산자-merge
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 같은 타입의 Observable들을 하나의 Observable로 합쳐주는 merge 연산자를 다루도록 하겠습니다.
![merge]({{"/img/merge.png"}}){: .center-block :}  

merge는 다양한 형태로 제공이 되는데, 하나하나 살펴보도록 하겠습니다.

```swift
    public func merge() -> Observable<Element.Element> 
    public func merge(maxConcurrent: Int) -> Observable<Element.Element>
```  
이 merge는 Element.Element 타입의 Observable을 내보내는데, 앞 Element는 ObservableConvertibleType입니다. 즉, merge연산은 Observable을 내보내는 Observable에서 호출하며 이렇게 방출된 Observable들을 모두 하나의 Observable 로 합쳐주게 됩니다. 이 때 합치는 수의 제한을 둘 수가 있는데, 이렇게 되면 수가 다차서 합쳐지지 못한 Observable들은 내부 큐에 보관되고, 이전에 합쳐졌던 Observable들 중 하나가 complete 될 때, 자동으로 큐에서 Observable을 하나 빼내서 이를 구독합니다. (이처럼 실제 구독 시점이 예상과는 다를 수 있기 때문에 Hot Observable의 경우에는 이벤트가 손실될 수도 있습니다.)

merge는 다음과 같은 연산도 제공합니다.

```swift
public static func merge<Collection: Swift.Collection>(_ sources: Collection) -> Observable<Element> where Collection.Element == Observable<Element>

public static func merge(_ sources: [Observable<Element>]) -> Observable<Element>

public static func merge(_ sources: Observable<Element>...) -> Observable<Element>
```  

이 연산은 Sequence 형태로 Observable을 받는다는 점을 빼면 위 merge와 기능은 같습니다. 

```swift
let timer1 = Observable<Int>.interval(RxTimeInterval.seconds(1), scheduler: MainScheduler.instance).map({"o1: \($0)"})
let timer2 = Observable<Int>.interval(RxTimeInterval.seconds(2), scheduler: MainScheduler.instance).map({"o2: \($0)"})


Observable.merge(timer1,timer2)
    .subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)
// o1: 0
// o1: 1
// o2: 0
// o1: 2
// o1: 3
// o2: 1
// o1: 4
// o1: 5
// o2: 2
```
merge를 통해 여러 흐름의 Observable을 하나로 취급하여 사용할 수 있게 됩니다.