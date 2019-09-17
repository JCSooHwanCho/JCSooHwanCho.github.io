---
layout: post
title: RxSwift연산자-zip
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 Observable의 이벤트를 수를 맞춰서 합쳐주는 zip 연산자를 알아보겠습니다.

![zip]({{"/img/zip.png"}}){: .center-block :}  

zip은 combineLatest와 이름빼고 거의 동일한 시그니처를 가집니다. 여러 Observable을 인자로 받을 수 있도록 선언되어 있고, 8개까지 정의되어 있다는 것 까지 동일합니다.

```swift
 public static func zip<O1: ObservableType, O2: ObservableType>
        (_ source1: O1, _ source2: O2, resultSelector: @escaping (O1.Element, O2.Element) throws -> Element)
        -> Observable<Element> {
        return Zip2(
            source1: source1.asObservable(), source2: source2.asObservable(),
            resultSelector: resultSelector
        )
    }
```  

combineLatest와의 차이는 이벤트를 내보내는 정책입니다. zip은 각 Observable이 내보내는 이벤트들을 내부적으로 가지고 있다가, 동일한 인덱스를 가진 이벤트들을 묶어 내보냅니다. 만약 인자로 주어진 Observable중 하나라도 종료된다면, 전체가 종료되어 버립니다.

```swift
let timer1 = Observable<Int>.interval(RxTimeInterval.seconds(1), scheduler: MainScheduler.instance).map({"o1: \($0)"})
let timer2 = Observable<Int>.interval(RxTimeInterval.seconds(2), scheduler: MainScheduler.instance).map({"o2: \($0)"})

Observable.zip(timer1,timer2)
    .subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)
// ("o1: 0", "o2: 0")
// ("o1: 1", "o2: 1")
// ("o1: 2", "o2: 2")
// ("o1: 3", "o2: 3")
// ...
``` 
zip을 이용해서 산발적으로 발생하는 이벤트를 동일한 순서로 동일한 때에 처리할 수 있습니다.