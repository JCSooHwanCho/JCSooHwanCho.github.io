---
layout: post
title: ModalPresentationStyle, TransitionStyle 파헤치기
comments: true
tags: [iOS 프로그래밍,Swift,Apple, 부스트코스]
---  

지난번 [Segue 파헤치기](../2019-07-28-Segue-파헤치기/)에서 못 다룬 부분을 마저 다뤄보고자 합니다. 바로 ModalPresentationStyle과 ModalTransitionStyle에 대한 부분입니다. 이 두가지 옵션은 화면이 어떤 방식으로 뜰 지, 어떻게 전환될지를 결정합니다.  

우선 이해하기 쉬운 ModalTransitionStyle부터 살펴 보겠습니다. ModalTransitionStyle은 뷰가 전환될 때의 효과를 제공해줍니다. 다음은 기본적으로 제공되는 전환효과들입니다. 

1. Cover Vertical  

    ![Cover Vertical]({{"/img/CoverVertical.gif"}})

    아래에서 세로로 다음 뷰컨트롤러가 올라오는 방식입니다. 아무 값도 설정하지 않았을 때의 기본 값으로 적용되기 때문에 가장 많이 쓰이는 방식입니다.  

2. Filp Horizontal  

    ![Filp Horizontal]({{"/img/FilpHorizontal.gif"}})

    가로방향으로 가운데를 축으로 삼아 마치 카드가 뒤집하는 듯한 효과를 냅니다.  

3. Cross Dissolve

    ![Cross Dissolve]({{"/img/CrossDissolve.gif"}})

    화면이 교차되는 효과입니다. 영상 편집을 해보셨다면 익숙할만한 효과입니다.  

4. Partial Curl  

    ![Partial Curl]({{"/img/PartialCurl.gif"}})  

    종이를 넘기는 듯한 효과입니다.  

다음으로는 PresentationStyle에 대해서 알아보도록 하겠습니다. PresentationStyle은 뷰 컨트롤러를 어떤 형태로 띄울 것인지를 결정해줍니다.  

> PresentationStyle은 지금 화면이 어떤 크기인가에 영향을 받습니다. 모든 화면에 대해서 프로파일을 제공할 수도 있겠지만, 그랬다가는 지나치게 복잡해지고 오토 레이아웃의 장점도 퇴색되기 때문에(모든 기기마다 레이아웃을 일일이 설정해야 한다면 오토 레이아웃은 필요가 없겠죠) Regular,Compact의 두가지로 구분을 합니다. Regular가 길고, Compact가 짧은 것이라고 생각하시면 됩니다.  
아이폰은 세로일 때 가로가 Compact, 세로가 Regular이고, 가로로 돌리면 반대로 가로가 Regular, 세로가 Compact가 됩니다. 아이패드는 가로 세로 모두 Regular 입니다.  

1. FullScreen  

    ![Full Screen]({{"/img/FullScreen.gif"}})  

    무조건 화면 전체를 차지하도록 띄웁니다. 새로 띄워진 뷰 아래 깔리는 뷰들은 전혀 보이지 않게 됩니다. Presentation의 기본값이고, 다른 PresentationStyle이 자신의 스타일을 적용할 수 없는 상황에서는 FullScreen으로 동작하게 됩니다.  

2. PageSheet  

     ![Page Sheet]({{"/img/PageSheet.gif"}})

    가로가 Regular일 때, 너비는 세로 방향일 때의 너비, 높이는 기기의 현재 높이로 지정됩니다. 즉, 세로모드일 때는 가로가 Regular라도 FullScreen과 동작이 동일하게 됩니다. 
    가로가 Compact일 경우에는 FullScreen으로 동작합니다.  

3. FormSheet  

    ![Form Sheet]({{"/img/FormSheet.gif"}})  

     가로가 Regular일 때, content 영역의 크기를 그대로 유지해주고, 아래에 반투명 뷰를 배치해줍니다. 기기가 가로모드이고 키보드가 나온다면, 뷰가 위로 올라가는 처리를 자동으로 해줍니다.  
     가로가 Compact일 경우에는 FullScreen으로 동작합니다.  

4. CurrentContext  

    ![Current Context]({{"/img/CurrentContext.gif"}})  

    글자 그대로 해석하면 현재 문맥에 맞춰서 띄우라는 뜻입니다. 정확히는 현재 뷰컨트롤러를 시작으로 해서 뷰컨트롤러의 [definesPresentationContext](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621456-definespresentationcontext)가 true로 설정되어 있는 뷰컨트롤러의 영역에 맞춰서 새로운 뷰가 띄워집니다. 일반적인 뷰컨트롤러는 기본값이 false지만, navigationController 등은 이 값이 true로 되어 있습니다. 해당 프로퍼티가 true인 뷰컨트롤러가 없는 경우, FullScreen으로 동작합니다.  
    
    이 CurrentContext는 화면에 2개 이상의 뷰 컨트롤러가 영역을 차지하고 있는 경우(SplitViewController 등) 하나만 바꾸고 싶다던가 할 때에 유용하게 사용할 수 있습니다.  
    ![SideBar example]({{"/img/CurrentContextExample.png"}})  

5. OverFullScreen

    ![Over Full Screen]({{"/img/OverFullScreen.gif"}})  

    FullScreen과 동일하지만 띄워지는 뷰가 투명할 경우 아래 쌓인 뷰가 비쳐보입니다. 실제로도 원래 FullScreen에서는 아래 깔리는 뷰가 뷰 계층에서 지워지지만 over를 적용하면 뷰 계층에서 사라지지 않습니다. 

6. OverCurrentContext  

    ![Over Current Context]({{"/img/OverCurrentContext.gif"}})  

    CurrentContext와 동일하지만 띄워지는 뷰가 투명할 경우 아래 쌓인 뷰가 비쳐보입니다. 실제로도 원래 CurrentContext에서는 아래 깔리는 뷰가 뷰 계층에서 지워지지만 over를 적용하면 뷰 계층에서 사라지지 않습니다. 

7. Popover  

    ![Popover]({{"/img/Popover.gif"}})  

    Popover는 다른 스타일과는 많이 다릅니다. 뷰의 특정 위치에서 팝업창처럼 뜨는 형태이기 때문에 두가지 옵션을 필수적으로 신경써야 합니다.

    1. 어느 정도의 크기로 띄워줄 것인가? - [preferredContentSize](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621476-preferredcontentsize) 프로퍼티로 조정해줍니다. 이 값은 Popover로 뜨는 뷰컨트롤러에 설정해줍니다. 
    2. 어떤 뷰를 중심으로 띄울 것인가? - Popover되는 뷰컨트롤러의 [popoverPresentationController](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621428-popoverpresentationcontroller) 프로퍼티를 통해 설정해줍니다. 두 가지 방법 중 반드시 하나만 적용되어야 합니다.
        1. [barButtonItem](https://developer.apple.com/documentation/uikit/uipopoverpresentationcontroller/1622314-barbuttonitem) 프로퍼티를 설정해줍니다.
        2. [sourceView](https://developer.apple.com/documentation/uikit/uipopoverpresentationcontroller/1622313-sourceview)와 [sourceRect](https://developer.apple.com/documentation/uikit/uipopoverpresentationcontroller/1622324-sourcerect) 프로퍼티를 설정해줍니다.
    
    Popover는 가로가 Regular인 경우에만 적용되며, 가로가 Compact인 경우에는 기본적으로 FullScreen으로 작동하게 됩니다.  

    Popover된 뷰 컨트롤러의 바깥 영역을 터치하게 되면 Popover된 뷰컨트롤러는 자동으로 dismiss됩니다. 하지만 가로가 Compact일 경우에 Popover가 작동할 경우에는 FullScreen이 되기 때문에 이 경우에도 dismiss가 제대로 될 수 있도록 장치를 마련해 둘 필요가 있습니다.  

    이외에도 TransitionStyle, PresentationStyle은 모두 커스텀이 가능합니다. 하지만 글이 너무 길어지기 때문에 관련 링크로 대체하겠습니다.  
    
    [Customizing the Transition Animations](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/CustomizingtheTransitionAnimations.html#//apple_ref/doc/uid/TP40007457-CH16-SW1)  
    [Creating Custom Presentations](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/DefiningCustomPresentations.html#//apple_ref/doc/uid/TP40007457-CH25-SW1)  

---  

단순하게 써왔던 것들도 자세히 파다보면 세심하게 설계되어 있음에 놀라곤 합니다. 다음에도 더 흥미로운 주제를 가지고 함께 알아보도록 하겠습니다:)