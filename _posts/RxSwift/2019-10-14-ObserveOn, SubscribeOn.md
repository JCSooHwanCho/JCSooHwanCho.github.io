---
layout: post
title: ObserveOn, SubscribeOn
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 ObserveOn과 SubscribeOn에 대해서 알아보겠습니다. 이 두 연산자는 자주 쓰이지만, 자주 헷갈리는 연산이기도 합니다.  

![observeOn]({{"/img/observeOn.png"}}){: .center-block :}  

![subscribeOn]({{"/img/subscribeOn.png"}}){: .center-block :}  

observeOn과 subscribeOn의 선언은 다음과 같습니다.  

```swift
     public func observeOn(_ scheduler: ImmediateSchedulerType) -> Observable<Element>

     public func subscribeOn(_ scheduler: ImmediateSchedulerType) -> Observable<Element>
```  

보다시피 선언은 이름 빼고는 완전히 동일합니다. 이 두개의 차이를 이해하기 위해 과거 연산자 기초 포스트에서 썼던 이미지를 다시 보도록 하겠습니다. 

![OperatorBlueprint]({{"/img/ObservableBlueprint.png"}}){: .center-block :}  

observeOn은 Event Flow의 스케쥴러를 바꿔주고, subscribeOn은 Subscribe Flow의 스케쥴러를 변경해줍니다. 이러한 것을 두고, subscribeOn은 위쪽으로 영향을 주고, observeOn은 아래쪽으로 영향을 준다고도 이야기 합니다.

이해를 위해 의사코드를 작성하도록 하겠습니다.
```swift
Observable.of(...) // scheduler 2
    .filter { ... } // scheduler 2
    .observeOn(scheduler1) // scheduler 2
    .map { ... } // scheduler 1
    .subscribeOn(scheduler2) // scheduler 1
     ... //각종 연산들, scheduler 1
    .subscribe { // scheduler 1
        ...
    }.disposed(by:bag)
```  

subscribe가 될 때 해당 Observable은 기본적으로 subscribe를 수행했던 스케줄러를 따라갑니다. subscribeOn은 해당 연산자 이후 구독이 일어나는(즉, 위쪽 방향으로의) Observable이 수행되는 default 스케줄러를 결정합니다. 다만, subscribeOn을 여러 번 쓸 경우, 가장 처음에 쓴 subscribeOn만 실질적인 영향을 미칩니다. 이는 실제 Observable이 동작하는 스케줄러는 이벤트가 발생한 스케줄러이기 때문입니다.

이후 이벤트가 발생하면, 각 Observable은 이벤트가 발생한 스케줄러 상에서 실행이 됩니다. 하지만 observeOn을 만나면, observeOn으로 지정한 스케줄러로 이벤트가 발생하는 스케줄러가 바뀝니다. 

이러한 특성을 이용하면 특정 작업을 수행할 스케줄러를 프로그래머의 의도에 맞게 자유자재로 조절해 줄 수 있습니다. 보통 subscribeOn은 한번만 써주고, ObserveOn으로 동작하는 스케쥴러를 바꾸는 방식을 사용합니다.