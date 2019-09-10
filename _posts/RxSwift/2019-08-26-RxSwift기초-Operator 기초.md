---
layout: post
title: RxSwift기초-Operator기초
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

RxSwift를 이루는 근본적인 요소를 뽑으라면 Observable, Observer, Operator, Scheduler의 4가지를 꼽을 수 있을 것 같습니다. 이번 포스트에서는 그 중 Operator의 원리에 대해서 알아보겠습니다. 

> 이 포스트는 Operator의 사용법에 대해서 다루고 있지 않습니다. 

---  

Rxswift에서 Operator(연산자)는 Observable을 다루는 메소드들을 통칭하는 용어입니다. 입력의 형태는 다양하여 일반화 할 수 없지만, **공통적으로 Observable을 반환한다는 공통점이 있습니다.** Operator를 카테고리로 분류해보자면 다음과 같습니다. Operator들은 이들 중 1개 이상의 카테고리에 속해있습니다.

> 해당 분류는 [reactiveX 공식 사이트](http://reactivex.io)를 참조하였습니다.

1. Creating Operator(생성 연산자) : 새로운 Observable을 만들어 반환합니다.
2. Filtering Operator(여과 연산자) : 기존 Observable을 조건에 맞는 이벤트만 방출하는 Observable로 가공합니다.
3. Transforming Operator(변환 연산자) : Observable가 내보내는 이벤트의 타입을 바꿔줍니다.
4. Combining Operator(합성 연산자) : 두개 이상의 Observable 을 하나의 Observable로 만들어줍니다.
5. Error Handling Operator(에러 처리 연산자) : error 이벤트가 나왔을 때, 에러를 처리할 수 있도록 흐름을 잠시 바꿔줍니다.
6. Observable Utility Operator(유틸리티 연산자) : Observable에 적용할 수 있는 유용한 기능들을 제공합니다.
7. Conditional and Boolean Operator(조건 연산자) : Observable의 이벤트 혹은 Observable을 판단하는 방법을 제공합니다.
8. Mathmetical and Aggregate Operator(수학, 집합 연산자) : Observable의 전체 이벤트를 대상으로 하는 연산자들입니다.
9. Connectable Observable Operator : 구독시 일어나는 행동들을 좀 더 세세하게 지정할 수 있게하는 Observable을 다루는 연산입니다.
10. Convert Observable : Observable을 다른 자료구조로 바꾸는 연산자입니다.

> Backpressure operator가 카테고리에 있지만, RxSwift에는 관련 연산자가 없기 때문에 제외하였습니다.  

이렇게 많은 연산자들을 마음껏 연결하여 사용할 수 있다는 것이 RxSwift의 강력한 점 중 하나입니다. 그렇다면 이런 것이 어떻게 가능할까요? 연산자를 호출하면 받게되는 Observable은 다음과 같은 구조로 이루어져 있습니다.

![OperatorHierachy]({{"/img/OperatorHierachy.png"}})  

간단하게 말하면, 각 연산자에 해당하는 일을 실제로 수행하는 구체 클래스를 Observable 이라는 공통된 인터페이스로 사용하는 형태입니다. 이러한 Observable을 차례대로 연결하여 실제 사용자가 구독하게 되는 하나의 Observable을 만들게 됩니다.

![OperatorBlueprint]({{"/img/ObservableBlueprint.png"}})

Subscribe메소드는 Observable 계층을 따라 올라가면서 상위 Observable들을 차례대로 구독하게 되고, Event는 Observable 계층을 따라 처리되면서 subscribe 메소드에서 넘겼던 클로져까지 전달되게 됩니다.(Operator에 따라서는 전달이 안될 수도 있습니다.) 

---  

Operator의 원리에 대해 알아보았습니다. 다음에는 상세한 Operator에 대해서 예시와 함께 알아보겠습니다.