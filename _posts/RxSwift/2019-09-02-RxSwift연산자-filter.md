---
layout: post
title: RxSwift연산자-filter
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 Observable이 발생시키는 이벤트를 필터링할 수 있는 filter연산자에 대해서 알아보겠습니다. 

![filter]({{'/img/filter.png'}}){: .center-block :} 

filter 연산자의 선언은 다음과 같습니다.  

```swift
public func filter(_ predicate: @escaping (Element) throws -> Bool)
        -> Observable<Element> {
        return Filter(source: self.asObservable(), predicate: predicate)
    }
```  

static 한정자가 붙지 않았기 때문에, 이 메소드는 이미 존재하는 Observable을 대상으로 호출하는 인스턴스 메소드입니다. 인자로는 (Element) -> Bool 타입을 받는데, 원본 Observable의 이벤트를 받아서 next 이벤트인 경우, 이벤트의 값을 검사하는 판단식을 의미합니다. 여기서 판단식이 true를 반환할 경우에는 이벤트가 무사히 나오게 되지만, false를 반환할 경우, 해당 이벤트는 더 이상 전달되지 못하고 사라져 버립니다.  

```swift
Observable.from([1,2,3,4,5,6,7,8,9,10])
    .filter({ $0%2 == 0 }) // 짝수인지 여부를 검사한다.
    .subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)

//2
//4
//6
//8
//10
//finished
```  

filter 연산을 통해, 필요한 이벤트만 골라 받을 수 있게됩니다.