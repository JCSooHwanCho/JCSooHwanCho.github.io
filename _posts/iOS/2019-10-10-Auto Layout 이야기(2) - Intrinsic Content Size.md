---
layout: post
title: 2019-10-10-Auto Layout 이야기(2) - Intrinsic Content Size
comments: true
tags: [iOS 프로그래밍,Swift,Apple]
category: [iOS]
---  

지난 포스트에 이어서 Auto Layout에 대해서 좀 더 살펴보도록 하겠습니다. 

* **Intrinsic Size**  
    Auto Layout은 기본적으로 뷰의 위치와 크기를 모두 정의하도록 Constraint를 구성해야 합니다. 하지만 어떤 뷰는 Content에 따라 고유한 크기를 가지곤 합니다. 이 크기를 'Intrinsic Content Size(컨텐츠의 본질적인 크기)'라고 합니다. 예를 들어 버튼은 (버튼의 타이틀 + 약간의 마진)의 크기를 Intrinsic Content Size로 가집니다.  

    모든 뷰가 이 Intrinsic content size를 가지는 것은 아닙니다. 각 뷰의 Intrinsic content size는 다음과 같습니다.

    1. UIView - Intrinsic content size를 가지지 않습니다.
    
    2. Slider - width만 Intrinsic content size로 가집니다.
    
    3. Label, Button, Switch, TextField - height와 width 모두 가집니다.
    
    4. TextView, ImageView - content에 따라 변화합니다.  

    Intrinsic content size는 뷰의 현재 컨텐츠에 기반을 둡니다. 예를 들어 Label이나 Button은 띄울 텍스트의 분량과 폰트에 따라 Intrinsic Size가 결정됩니다. ImageView나 처음에는 Intrinsic Size를 가지고 있지 않다가, 나중에 이미지가 채워질 때에야 비로소 이미지의 크기로 Intrinsic Size 로 정해집니다.  

    TextView는 좀 더 복잡합니다. Scrolling이 설정되어 있는 경우, Intrinsic Size가 설정되지 않습니다. Scrolling이 설정되어 있지 않은 경우, 자동 줄바꿈(line wrapping)을 고려하지 않은 텍스트의 크기로 결정됩니다. 또한 TextView 자체에 대한 Constraint가 있는 경우, 해당 Constraint를 만족시키는 선에서 Contents를 띄울 수 있도록 Intrinsic size를 조정합니다. 

* **Content Hugging&Compression Resistance**  
    Auto Layout은 Intrinsic size를 각 차원(width, height)마다 한쌍의 부등식으로 정의합니다. 각 부등식은 방향에 따라 각각 Content Hugging, Compression Resistance로 부릅니다.

    ```
    // Compression Resistance
    View.height >= 0.0 * NotAnAttribute + IntrinsicHeight
    View.width >= 0.0 * NotAnAttribute + IntrinsicWidth
    
    // Content Hugging
    View.height <= 0.0 * NotAnAttribute + IntrinsicHeight
    View.width <= 0.0 * NotAnAttribute + IntrinsicWidth
    ```  

    ![IntrinsicContentSize]({{"/img/AutoLayout/IntrinsicContentSize.png"}}){: .center-block :}  

    Content Hugging은 뷰를 안쪽으로 축소시켜서 뷰의 크기를 컨텐츠에 맞추는 역할을 하고, Compression Resistance는 뷰를 바깥쪽으로 확장시켜서 컨텐츠가 잘리지 않도록 해주는 역할을 합니다. 이 부등식 역시 Constaint이기 때문에 우선순위 값을 가지는데, Content Hugging은 250, Copression Resistance는 750이 기본 값입니다. 이는 뷰가 확장되어 불필요한 공간을 차지하는 것이 뷰가 지나치게 줄어들어 컨텐츠가 잘리는 것보다는 낫다는 뜻입니다.  

    Intrinsic size를 사용하면 레이아웃을 컨텐츠 크게에 맞게 동적으로 조절해 줄 수 있으며, Auto Layout을 맞추기 위해 필요한 Constraint 수 역시 줄여줍니다. 하지만 이를 위해서는 Content Hugging & Compression Resistance(CHCR) 값을 신경써 주어야 합니다.

* **CHCR 관리 가이드라인**  
   * 여러 뷰를 확장해서 빈칸을 채워야 하는경우, 이 뷰들이 모두 같은 Content Hugging(CH)을 가지면 안됩니다. 예를 들어 Label과 TextField를 함께 두는 경우, TextField가 확장되서 빈칸을 채우는 것이 자연스럽습니다. 이 경우 TextField의 CH를 Label의 Ch보다 낮춰주면 됩니다. 
   
   * 뷰가 의도치 않게 확장되어서 레이아웃이 깨지는 경우가 왕왕 있습니다. 이를 막기 위해서 CH를 높여주면 됩니다.  
   
   * Baseline에 관한 Constraint는 뷰가 Intrinsic height를 유지하고 있는 경우에만 올바로 동작합니다. 만약에 뷰가 세로로 축소되거나 확장된다면, 해당 Constraint는 올바로 동작하지 않습니다.  
   
   * 일부 뷰는 항상 Intrinsic size를 유지해야 하는 경우가 있습니다. 이때는 CHCR 값을 모두 높여주어서 크기가 변하지 않도록 해줍니다.
   
   * CHCR 값을 required(1000)으로 만들지 않도록 합니다. 뷰가 잘못된 크기가 되는 것이 Auto Layout 전체에 충돌을 일으키는 것보다는 낫습니다. 정말로 큰 값이 필요하다면 999를 할당하도록 합니다.  

---  

이것으로 auto Layout의 핵심적인 내용을 모두 살펴보았습니다. 아는 만큼 더 좋은 코드를 고민하고 만들어 낼 수 있게 될 것을 기대해봅니다.