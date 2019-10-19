---
layout: post
title: Rxswift기초 - Hot vs Cold Observable
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

Observable은 두가지 종류로 구분될 수 있습니다. Hot Observable과 Cold Observable이 그것입니다. 이 두가지는 같은 Observable이지만 그 특성은 다릅니다.

[공식 홈페이지](http://reactivex.io/documentation/observable.html)에서의 정의는 다음과 같습니다.

> When does an Observable begin emitting its sequence of items? It depends on the Observable. A “hot” Observable may begin emitting items as soon as it is created, and so any observer who later subscribes to that Observable may start observing the sequence somewhere in the middle. A “cold” Observable, on the other hand, waits until an observer subscribes to it before it begins to emit items, and so such an observer is guaranteed to see the whole sequence from the beginning.  
>
> 해석 : 언제 Observable은 이벤트 발생을 시작할까요? 이는 Observable에 따라 다릅니다. 'hot' Observable은 만들어진 즉시 이벤트를 발생시킬 수 있고, 나중에 오는 구독자는 시퀀스의 중간부터 이벤트를 관찰하게 됩니다. 'cold' Observable은 반대로 어 떤 Observer에 의해 구독될 때 까지 아이템을 방출하지 않고 기다립니다. 또한 시퀀스 전체를 Observer가 받아볼 수 있음이 보장됩니다.  

즉, Hot Observable과 Cold Observable을 구분하는 기준을 다음과 같이 살펴볼 수 있습니다.

* 이벤트 발생이 시작되는 시점 (구독시 vs 구독과 관계 없이)

* 구독했을 때 이벤트 시퀀스를 처음부터 관찰할 수 있는가? (있다 vs 없다)  

이러한 특성들 때문에 추가적으로 파생되는 차이점이 있습니다. 

* Hot Observable은 구독 여부에 상관 없이 이벤트를 발생시키기 때문에 일단 동작하기 시작하면 리소스를 사용하게 됩니다. 또한 이 동작을 시작하는 시점을 조절할 수 있는 메소드를 제공합니다. (RxSwift에서는 connect() 입니다.) Cold Observable은 구독과 동시에 무조건 동작하며, 그 이전에는 연산 자원을 소모하지 않습니다.

* Hot Observable은 여러 Observer가 하나의 Observable을 공유할 수 있습니다. 하지만 Cold Observable은 Observer마다 별도의 Observable 인스턴스를 가지게 됩니다. 만약 시퀀스를 만들어 내는 과정이 오래 걸리는 경우, Cold Observable은 구독이 일어날 때 마다 그 과정을 거쳐야 합니다.  

RxSwift는 Hot Observable을 'ConnectableObservable'로도 부릅니다. 또한 Cold Observable을 Hot Observable로 바꿔주는 연산자들도 제공합니다. 반대로 Hot Observable을 Cold Observable과 유사하게 취급할 수 있도록 ReplaySubject 같은 것도 제공합니다.  

Hot Observable의 대표적인 것들은 프로퍼티, 타이머, UIEvent 등이 있습니다. Cold Observable의 대표적인 것들은 Http 요청을 들 수 있습니다.  

