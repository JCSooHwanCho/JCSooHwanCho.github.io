---
layout: post
title: RxSwift연산자-flatmap
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 Observable들을 합쳐서 map을 적용할 수 있게 해주는 flatmap에 대해서 알아보도록 하겠습니다.

![flatmap]({{"/img/flatMap.png"}})  

flatmap의 선언은 다음과 같습니다.

```swift
// merge.swift에 정의되어 있습니다.
public func flatMap<Source: ObservableConvertibleType>(_ selector: @escaping (Element) throws -> Source) -> Observable<Source.Element>
```  

selector 인자는 Element를 인자로 받아서 Observable로 변환 가능한 타입(그냥 Observable로 봐도 무방합니다)을 반환합니다. 여기서 반환된 Observable을 내부적으로 모두 합쳐서 하나의 Observable을 통해 이벤트를 내보내게 되는데, 이 Observable이 바로 반환값으로 주어지는 Observable 입니다.

map과의 차이점은 인자로 주어지는 클로저의 반환 타입뿐입니다. 따라서 map에서 수행했던 변환과정은 flatmap에서도 적용이 가능합니다. 다만 클로저의 반환값이 Observable이라는 것을 주의합시다. 

위 두 문단을 합치면 다음과 같은 결론을 얻을 수 있습니다.

1. map을 적용한 Observable 들을
2. 모두 합쳐 하나의 Observable로 만들어준다

flatmpa은 몇가지 변형을 제공해주는데, 기본적인 원리는 같지만 합성된 Observable들 중에서 내부적으로 특정 조건에 맞는 Observable을 선택하여 내보냅니다.

```swift
// 현재 이벤트를 발생시키고 있는 Observable이 dispose되지 않으면, 다음에 나온 Observable들은 구독되지 못하고 무시되버립니다.
public func flatMapFirst<Source: ObservableConvertibleType>(_ selector: @escaping (Element) throws -> Source) -> Observable<Source.Element>
// 다음 Observable이 나오면, 이전의 Observable은 dispose 됩니다.
public func flatMapLatest<Source: ObservableConvertibleType>(_ selector: @escaping (Element) throws -> Source) -> Observable<Source.Element>
```  

> flatMapWithIndex() 라는 연산자도 있었지만 deprecated되었습니다. 이는 각 Observbale의 이벤트를 인덱스 값을 추가로 받아 처리할 수 있게 해주는 역할이였는데, 지금은 대신해서 enumerated().flatmap() 연산을 사용하도록 권장하고 있습니다.  

```swift
let timer1 = Observable<Int>.interval(RxTimeInterval.seconds(1), scheduler: MainScheduler.instance).map({"o1: \($0)"})
let timer2 = Observable<Int>.interval(RxTimeInterval.seconds(2), scheduler: MainScheduler.instance).map({"o2: \($0)"})

Observable.of(timer1,timer2)
    .flatMap({$0})
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
```  

flatmap은 특히 비동기 처리 후, 그 결과를 Observable로 변환해서 Rx 스타일로 처리하도록 하는데에 굉장히 유용하게 쓰입니다.