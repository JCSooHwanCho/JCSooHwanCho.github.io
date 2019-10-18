---
layout: post
title: Rxswift연산자-concat
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 두 Observable을 연결하는 concat 연산에 대해서 알아보도록 하겠습니다.  

![concat]({{"/img/concat.png"}}){: .center-block :}  

concat의 선언은 다음과 같습니다.

```swift

// 한 Observable에 다른 Observable을 연결합니다.
public func concat<Source: ObservableConvertibleType>(_ second: Source) -> Observable<Element> 
    where Source.Element == Element 

// Sequence 형태로 주어진 Observable들을 순서대로 연결합니다.
public static func concat<Sequence: Swift.Sequence>(_ sequence: Sequence) -> Observable<Element>
    where Sequence.Element == Observable<Element> 

// Collection 형태로 주어진 Observable들을 순서대로 연결합니다.
public static func concat<Collection: Swift.Collection>(_ collection: Collection) -> Observable<Element>
    where Collection.Element == Observable<Element> 

// 가변 인자로 주어진 Observable들을 순서대로 연결합니다.
 public static func concat(_ sources: Observable<Element> ...) -> Observable<Element>
```  

다양한 형태로 제공되긴 하지만 기능은 간결합니다. 앞에서부터 차례대로 Observable들을 이어붙여서 하나의 Observable로 합쳐주는 것입니다. 하나의 Observable이 될 수 있어야 하므로 모든 Observable들의 타입이 동일하거나 동일하게 캐스팅 되어야 합니다.  

내부적으로는 우선 첫번째 Observable을 구독한 뒤, 이벤트를 그대로 다시 방출합니다. 그러다가 completed 이벤트가 발생하면 다음 observable을 구독하고 똑같이 이벤트들을 그대로 방출하는 것을, 인자로 주어진 모든 Observable을 구독할 때 까지 수행합니다.

만약 중간에 error이벤트가 발생할 경우, 이후의 Observable들은 구독될 기회를 얻지 못하고 즉시 Observable이 종료되고 dispose 됩니다.  

이전 Observable이 끝나야만 다음 Observable 을 구독한다는 특성상 끝나지 않는 Observable을 대상으로 하거나, 구독 여부와 상관 없이 이벤트를 방출하는 특성을 가진 Hot Observable등에는 사용을 권장하지 않습니다. 