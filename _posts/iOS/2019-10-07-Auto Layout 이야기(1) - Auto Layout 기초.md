---
layout: post
title: Auto Layout 이야기(1) - Auto Layout 기초
comments: true
tags: [iOS 프로그래밍,Swift,Apple]
category: [iOS]
---  

 iOS 개발에서 Auto Layout은 큰 비중을 차지합니다. SwiftUI가 나오면서 이후에는 이러한 상황이 바뀌게 되겠지만 SwiftUI가 활성화되기 까지는 아직 시간이 남았기 때문에, 여전히 배워야 할 이유는 남아있습니다. 이번 포스트를 시작으로 Auto Layout의 동작 원리를 하나하나 살펴보면서 Auto Layout을 자세히 살펴보도록 하겠습니다.

> 이 포스트는 다음 가이드을 참조하여 작성되었습니다.  
>  [Auto Layout Guide](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/index.html#//apple_ref/doc/uid/TP40010853-CH7-SW1)  

* **Auto Layout의 필요성**  
   뷰가 항상 고정된 크기와 위치를 유지한다면, Auto Layout은 필요가 없을 것입니다. 하지만 뷰는 외부적인 이유나 내부적인 이유로 그 크기와 위치가 변해야만 하는 상황에 처하게 됩니다. 

  1. 외부적 원인
    * window 사이즈를 바꾼다(macOS)
    * iPad에서 Split View를 사용한다.
    * 디바이스가 회전하면서 가로/세로 모드가 바뀐다.(iOS)
    * 전화가 오거나 오디오 녹음으로 인해 추가적인 상태바가 뜬다.
    * 서로 다른 사이즈 클래스, 혹은 서로 다른 스크린 사이즈를 하나의 뷰로 지원하고 싶다.  
<br>
  2. 내부적 요인
    * 컨텐츠 자체가 변한다.
    * 여러 지역에서 각 지역에 맞도록 앱을 만들어야 한다.(Internationalization)
    * [동적 타입(Dynamic Type)](https://developer.apple.com/documentation/uikit/uifont/scaling_fonts_automatically)을 지원해야 한다.

    이러한 변화 요인들은 런타임에 동적으로 발생하고, 실행전에 파악할 수 있는 요소들이라도(사이즈 클래스, 스크린 사이즈 등) 그 경우의 수가 다양하기 때문에 이러한 경우를 일일이 수동으로 대응하는 것은 굉장히 비효율적입니다.

    Auto Layout 이전에는 직접 프로그래머가 frame의 위치를 계산하고, autoResizingMask를 이용해서 상위뷰의 변화에 자동으로 대응할 수 있도록 했습니다. 하지만 이러한 방법은 뷰를 디자인하기도 어렵고, 디버깅과 유지보수에도 어려움이 있었습니다. autoResizingMask가 이러한 어려움을 조금 경감시켜주긴 했지만, 뷰가 조금만 복잡해져도 신경써야할 것이 많아지고, 내부적 요인에는 대응할 수 없다는 치명적인 문제 또한 가지고 있었습니다.

    Auto Layout은 이러한 frame 기반 레이아웃 패러다임을 뷰 간의 관계(RelationShip) 기반으로 변화시켰습니다. Auto Layout은 뷰 간의 관계를 정의한 Constraint(제한)의 모음으로 정의되며, Auto Layout은 외부적, 내부적 변화에 따라 뷰의 사이즈와 위치를 Constraint를 만족하도록 자동으로 계산해줍니다.   

* **Auto Layout의 원리**  
    Auto Layout은 Constraint의 모음이며, 이 Constraint들은 1차 방정식(혹은 부등식)들입니다. 즉 Auto Layout은 연립방정식(혹은 부등식)의 형태이며, 이 연립식들이 유일한 해를 가지도록 정의하는 것이 프로그래머의 목표입니다. 

    ![AutoLayout Formula]({{"/img/AutoLayout/AutoLayout_Formula.png"}}){: .center-block :}  
    * item 1: Constraint의 대상이 되는 두 뷰 중 하나입니다. 뷰 혹은 레이아웃 가이드 객체만이 가능하며, 반드시 지정되어야만 합니다.
    
    * atrribute1: item1에서 실제로 Constraint에 영향을 받는 속성입니다.
    
    * relationShip: 좌변과 우변의 관계입니다. 등호(=), 등호가 붙은 부등호(>=,<=)의 3가지 경우가 가능합니다.
    
    * multiplier: attribute2에 곱해지는 부동 소수점 타입의 계수입니다.  
    
    * item2: Constraint의 대상이 되는 두 뷰 중 나머지 하나입니다. First Item과 다르게, 지정되지 않는 것이 가능합니다.
    
    * attribute2: item2에서 실제로 Constraint에 영향을 받는 속성입니다.
    
    * constant: attribute2에 더해지는 부동 소수점 타입의 상수

  대부분의 Constraint는 위와 같은 형태로 정의됩니다. item1과 item2는 동일할 수도 있는데, 이 경우는 하나의 item의 서로 다른 두 속성간의 관계를 정의하는 것이 됩니다. 또한 특정 attribute를 상수 값으로 고정하는 것도 가능한데, 이 경우 item2는 공백으로 남고 attribute는 [notAnAttribute](https://developer.apple.com/documentation/uikit/nslayoutconstraint/attribute/notanattribute)가 되며 multiplier는 0.0으로 설정됩니다.

  Auto Layout을 적용할 수 있는 속성은 기본적으로 다음과 같습니다.

  * 사방 테두리(leading, trailing, top, bottom)
  
  * 너비와 높이(width, height)
  
  * 가로, 세로 중앙(vertical center, horizontal center)
  
  * (Text 계열의 경우) 1개 이상의 baseline
  
  * LayoutMargin의 속성들(View와 동일)

  > Auto Layout을 적용할 수 있는 속성은 macOS와 iOS에서 큰 차이를 보이기 때문에, 각 플랫폼에 맞는 문서를 참조해야 합니다.

  Constraint의 조합은 유일한 해를 가져야 하는데, 이를 좀 더 쉽게 말하면 모든 뷰의 위치와 크기가 Constraint를 통해 정의될 수 있어야 한다는 것입니다. 이 조건을 만족하는 Constraint 구성은 여러가지가 있을 수 있습니다.

  Auto Layout은 기본적으로 모든 Constraint를 적용해서 계산을 하는데, 이 때 유일한 해를 찾을 수 없을 경우, 에러를 발생시키고 Constraint중 하나를 빼도록 요구합니다. 이럴때 사용할 수 있는 것이 Constraint Priority입니다.

  Constraint Priority는 1에서 1000의 값을 가집니다. 1000의 우선순위를 가진 Constraint는 required로 분류되어 무조건적으로 사용되고, 나머지 Priority는 모두 Optional로 분류됩니다. Constraint의 해를 계산할 때, 높은 우선순위부터 낮은 우선순위까지 차례대로 계산해가면서, 올바르지 않은 Constraint는 스킵하고 계산을 이어나갑니다. 하지만 이렇게 스킵된 Constraint가 전혀 영향을 미치지 않는 것은 아닙니다. 마지막까지 계산된 결과에 모호성이 있을경우, 시스템은 스킵된 Constraint를 최대한 만족하는 답을 선택합니다.

---  

Auto Layout의 기초에 대해서 살펴보았습니다. 다음번 포스트에서는 이어서 Auto Layout에 대해서 알아야 할 것들을 살펴보도록 하겠습니다.