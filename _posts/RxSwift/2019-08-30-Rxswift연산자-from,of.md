---
layout: post
title: RxSwift연산자-from,of
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 시퀀스를 가지고 Observable을 만드는 from, of 연산자에 대해 알아보겠습니다.

from과 of는 모두 여러 값들을 차례대로 내보내는 Observable을 만듭니다. 이 두가지 연산자는 굉장히 유사하고, 실제로 공식 홈페이지의 설명은 [from](http://reactivex.io/documentation/operators/from.html) 하나뿐입니다.

![From]({{'/img/from.png'}}){: .center-block :} 

이는 기본적으로 from과 of가 동일한 연산자이기 때문입니다. 실제로도 RxSwift 저장소를 살펴보면 [Sequence.swift](https://github.com/ReactiveX/RxSwift/blob/master/RxSwift/Observables/Sequence.swift) 안에 두 연산자가 모두 정의되어 있고, 함수 몸체도 거의 차이가 없습니다.

```swift
public static func of(_ elements: Element ..., scheduler: ImmediateSchedulerType = CurrentThreadScheduler.instance) -> Observable<Element> {
    return ObservableSequence(elements: elements, scheduler: scheduler)
}

public static func from(_ array: [Element], scheduler: ImmediateSchedulerType = CurrentThreadScheduler.instance) -> Observable<Element> {
    return ObservableSequence(elements: array, scheduler: scheduler)
}
```  

본체에 대한 자세한 설명은 생략하지만, 두 연산자가 거의 같다는 것은 확실히 알 수 있습니다. 두 연산자의 차이는 첫 인자가 가변인자인가 배열인가의 차이입니다.  

```swift
// 이 두 경우 모두 값을 직접 받기 때문에, 타입 추론이 가능해집니다.
// from 연산자 사용
Observable.from([1,2,3]) // Observable<Int>
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

// of 연산자 사용
Observable.of([1,2,3]) // Observable<[Int]>
    .subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
    }.disposed(by: bag)

// [1,2,3]
// finished
``` 

상황에 맞게 두 연산자를 구분하며 사용하면 됩니다.