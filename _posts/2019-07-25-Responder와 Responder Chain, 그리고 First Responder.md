---
layout: post
title: Responder와 Responder Chain, 그리고 First Responder
comments: true
tags: [iOS 프로그래밍,Swift,Apple, 부스트코스]
---  

textField,textView를 다루다가보니 키보드가 화면을 가리는 경우가 생겼습니다. 엔터키를 눌러도, 키보드 바깥을 눌러도 아무런 반응이 없어서, 관련 키워드를 찾아보다가 First Responder라는 개념을 알게 되었습니다. 해당 개념을 잘 사용해서 문제는 해결했지만, First Responder를 찾다 보니 Responder라는 개념도 알아둘 필요가 있다는 생각이 들었습니다. 그래서 관련 글을 찾아 정리해보았습니다.

> 해당 글은 UIKit을 중심으로 서술이 되어 있지만, 같은 개념이 AppKit에도 있습니다. 다만 프로퍼티 명이나 메소드 명이 조금씩 다르기 때문에 그 부분을 주의하셔야 합니다.

> 이 글은 다음 Article들과 블로그를 참조하여 작성되었습니다  
[Using Responders and the Responder Chain to Handle Events](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events)  
[Responder Object](https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/Responder.html)  
[Responder Chani and Touch Event](https://baked-corn.tistory.com/129)


* *개요*  

    앱은 UIResponder 타입의 객체(이하 Responder) 를 이용해 여러 이벤트를 받고 처리합니다. 이 UIResponder의 하위 타입으로는  UIView,UIViewController,UIApplication 등이 있는데, 이는 거의 대부분의 UI 객체들이 이벤트를 받고 처리할 수 있는 능력을 가질 수 있다는 것을 뜻합니다. 

    ![Responder hierachy]({{"/img/ResponderHierachy.jpg"}})  

    Responder가 이벤트를 받으면 이를 반드시 처리하거나, 다른 Responder가 처리할 수 있도록 넘겨야 할 의무가 생깁니다. 앱이 이벤트를 받으면, UIKit은 적절한 Responder를 지정해서 이벤트를 넘겨서 처리를 하게 하는데, 이 처음으로 이벤트를 받는 Responder를 First Responder라고 합니다.

    만약 First Responder가 이벤트를 처리하지 못한다면 처리할 수 있는 Responder가 나올 때 까지 연쇄적으로 다음 Responder에게 넘어가게 됩니다. 이것을 *Respoder Chain* 이라고 하고, 이 Chain은 뷰 계층의 상태에 따라 자동으로 구성되게 됩니다.

    ![Responder Chain]({{"/img/ResponderChain.png"}})  

    특이할 점은, 만약 현재 Responder가 Root View인 경우에는 이벤트가 다음 상위 객체인 UIWindow로 넘어가는 게 아니라 해당 뷰를 소유하는 ViewController로 넘어가게 된다는 점입니다. 이렇게 이벤트는 넘어가고 넘어가 UIApplication 객체, 만약 appDelegate가 UIResponder 타입의 객체고 이전에 이벤트르 받은 적이 없다면 appDelegate까지 전달되게 됩니다.  

* *First Responder 결정하기*  

    UIKit은 이벤트 타입에 기반하여 해당 이벤트의 First Responder가 될 뷰를 결정하게 되는데 그 내용은 다음 표와 같습니다.  

    | 이벤트 타입 | First Responder |
    |:-:|:-:|
    |터치 이벤트 | 터치가 발생한 뷰 |
    |프레스 이벤트 | 포커스를 가진 뷰 |
    | 흔들기 이벤트 | 사용자나 UIKit이 지정한 뷰 |
    | 원격 이벤트 | 사용자나 UIKit이 지정한 뷰 |
    | 편집 메뉴 이벤트 | 사용자나 UIKit이 지정한 뷰 |  

    <br>
    
    > 주의할 점은 가속도 센서, 자이로 센서, 자기장 센서 등에서 발생하는 이벤트는 Responder Chain을 따르지 않고 지정된 객체로 바로 전달된다는 점입니다. 해당 이벤트를 처리하기 위해 서는 [Core Motion Framework](https://developer.apple.com/documentation/coremotion)를 사용해야 합니다.  

    컨트롤 객체들은 액션 메시지를 통해서 자신들의 타켓 객체와 통신하게 되는데, 사용자가 컨트롤 객체와 상호작용하면, 컨트롤 객체는 자신의 타겟 객체에게 액션 메시지를 보내는 방식으로 이루어지게 됩니다. 

    비록 액션 메시지는 이벤트는 아니자만, Responder Chain을 이용하게 됩니다. 만약 특정 컨트롤에서의 타겟이 nil이라면, UIKit은 해당 컨트롤을 가진 뷰에서부터 시작해서 Responder Chain을 따라가며 적절한 액션 메소드를 찾게 됩니다.  

* *어떤 Responder가 이벤트를 가지고 있는지 확인하기*  
    
    UIKit은 뷰를 기반으로 hit-testing을 진행하여 이벤트가 발생한 위치를 확인합니다. UIView의 [hitTest(_: with:)](https://developer.apple.com/documentation/uikit/uiview/1622469-hittest) 함수는 뷰 계층을 탐색하면서 해당 터치 이벤트를 가지고 있는 가장 깊은 계층의 뷰를 찾아 내어 반환하는데, 이렇게 찾아낸 뷰는 해당 터치 이벤트에 대한 First Responder가 됩니다. 
    
    > 이 때 주의할 것은, 해당 뷰의 경계를 벗어나는 이벤트에 대해서는 뷰 자신 뿐 아니라 모든 하위 뷰에 대해서도 해당 이벤트를 무시한다는 점입니다. 뷰의 [clipToBounds](https://developer.apple.com/documentation/uikit/uiview/1622415-clipstobounds) 프로퍼티가 false인 경우 하위 뷰가 상위 뷰의 경계를 벗어날 수 있지만, 이 경우 상위 뷰의 범위를 벗어나서 일어나는 하위 뷰의 이벤트는 받을 수 없게 됩니다. 

    터치 이벤트가 발생하면 UIKit은 UITouch 객체를 만들어서 뷰와 연결합니다. 터치 위치가 바뀌거나 여러 변수들이 바뀌어도 동일한 UITouch 객체를 사용하여 정보만 갱신해줍니다. 절대 바뀌지 않는 프로퍼티는 view로, 터치 위치가 뷰를 벗어나더라도 이 값은 변하지 않습니다. 터치가 끝나게 되면 UIKit은 사용했던 UITouch 객체를 해제합니다.
    
    > 다만 실제로는 무조건 해제가 되는 건 아닙니다.  

    ```swift
        var id:Set<ObjectIdentifier> = Set()
        override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
            
            let t = event!.allTouches!
            for to in t{
                id.insert(ObjectIdentifier(to))
            }
            print("touch begin : \(id.count)")
        }
        
        override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
            let t = event!.allTouches!
            for to in t{
                id.insert(ObjectIdentifier(to))
            }
            print("touch end : \(id.count)")
        }  
    ```  
    <br>  

    |![result]({{"/img/UITouchExample.png"}})|  
    |:-:|  
    |실행 결과. 한 개 손가락으로만 반복적으로 터치했습니다.|  

    <br>  
    > 만약 매번 객체가 새로 만들어진다면 숫자가 지속적으로 증가해야만 하는데, 어느정도 유지되는 구간이 존재합니다. 다만 어느정도 텀을 두고 해도 유지되는 경우가 있고 늘어나는 구간이 있어서 규칙성을 찾기는 쉽지 않았습니다.

* *Responder Chain 변경하기*  

    next 프로퍼티를 오버라이드 해서 다음번 Responder 객체를 변경함으로서 Responder Chain을 변경할 수 있습니다.  

    * 예시  
        - UIView : 자신이 최상위 뷰일 경우에 다음 Responder를 자신의 상위 뷰가 아닌 뷰컨트롤러로 변경합니다.  
        - UIViewController
            1. 최상위 뷰컨트롤러라면, 다음 Responder는 window 객체가 됩니다. 
            2. 다른 뷰컨트롤러에 의해 띄워진 뷰컨트롤러일 경우는 해당 뷰 컨트롤러를 띄운 뷰컨트롤러가 다음 Responder가 됩니다.  
        - UIWindow : 최상위 뷰이기 때문에, 다음 Responder는 UIApplication 객체가 됩니다.  
        - UIApplication : appDelegate 객체가 다음 Responder가 되는데, appDelegate 객체가 UIResponder 타입의 객체고 뷰,뷰컨트롤러 혹은 UIApplication 자신이면 안된다.
        - appDelegate : 더 이상 내려갈 데가 없기 때문에 nil로 설정된다. 이 때까지 처리되지 않은 이벤트는 사라집니다.

---  

* *First Responder에 대해서* 
    터치 이벤트의 경우는 터치가 일어난 뷰가 곧 First Responder가 되고 이는 어떤 뷰여도 가능합니다. 하지만 다른 이벤트들은 포커스를 받고 있거나(UIPress), UIKit에 의해 First Responder로 지정이 되어있는 상태여야 합니다. 오늘은 그 중에서 후자인  UIKit이 어떻게 First Responder를 지정하는지 확인해보고자 합니다.  
    First Responder가 되면 거의 모든 이벤트들을 우선적으로 받게 됩니다. 거기다가 타겟이 없는 상태의 액션 메시지도 받을 수 있는 상태가 됩니다.
    어떤 뷰가 First Responder가 되기 위해서는 해당 뷰가 First Responder가 될 의지가 있음을 알릴 필요가 있는데 이는 다음 프로퍼티를 통해 이루어집니다.

    ```swift
    var canBecomeFirstResponder: Bool//기본값은 false
    ```

    커스텀 뷰를 만들어 해당 프로퍼티를 true를 반환하도록 변경하면 이제 이 커스텀 뷰는 First Responder가 될 수 있습니다.  
    First Responder가 되려면 다음과 같은 과정을 거칩니다.

    1. First Responder를 만들려는 View에 대해 becomeFirstResponder() 메소드를 호출합니다.  
    
        ```swift
        func becomeFirstResponder() -> Bool //호출한 뷰가 first responder가 되면 true,되지 않으면 false
        ```  

    2. UIKit은 현재 First Responder 상태인 뷰에게 First Responder 권한을 내려놓을 것을 요청합니다. 요청을 받은 뷰는 First Responder 권한을 내려놓을 수 있는 지 여부를 체크합니다. 

        ```swift
        var canResignFirstResponder: Bool { get } //First Responder 권한을 내려놓을 수 있는지 여부, 기본값은 true

        func resignFirstResponder() -> Bool //First Responder 권한을 내려놓는다. 성공하면 true, 실패하면 false를 반환한다. 기본값은 true
        ```  

    3. 2번에서 true를 반환한다면, First Responder가 되려는 View의  canBecomeFirstResponder 프로퍼티를 호출해서 true를 반환한다면 기존 First Responder는 권한을 내려놓고, 새로운 View가 First Responder가 됩니다. 아니라면 아무 일도 일어나지 않습니다. 이 때 권한을 내려놓을 때 이전 First Responder에 의해 띄워졌던 시스템 키보드(혹은 inputView)는 내려가게 됩니다. 

    4. First Responder가 되면, 이어지는 이벤트들이 해당 뷰로 전달되게 되고 해당 뷰가 inputView를 가지고 있다면 UIKit은 inputView를, 없다면 시스템 키보드를 띄워주게 됩니다.

    > canBecomeFirstResponder,becomeFirstResponder()는 현재 활성화된 뷰 계층에 있지 않은 뷰에 대해서 호출하는 것에 대해서의 결과가 정의되어 있지 않기 때문에 호출하면 안됩니다.   
    호출 전에 뷰가 유효한지 판단하려면, 해당 view의 window 프로퍼티를 체크해서 유효한 window 객체를 가졌는지 체크하는 방법을 사용할 수 있습니다.

* *UITextField,UITextView,UISearchBar 에서의 First Responder*  

   UITextField,UITextView,UISearchBar는 특별합니다. 대부분의 뷰와 컨트롤들은 canBecomeFirstResponder가 기본값인 false로 설정되어 있는데, 이 세 가지 뷰는 canBecomeFirstResponder 메소드가 true가 되도록 오버라이드 되어있습니다.  

   추가적으로 해당 뷰는 '편집 모드(editing mode)'라는 것을 가지고 있습니다. 시스템 키보드가 등장할 때, 편집 모드에 들어감과 동시에 델리게이트에 알림(notification)을 전달합니다. 이후 First Responder 권한이 사라질 때 시스템 키보드가 내려가면서 델리게이트에 알림을 전달하게 됩니다. 이 과정에서 델리게이트가 끼여있는 이유는 시스템 키보드가 나타나면 화면 일부를 가리면서 사용성에 문제가 생길 수 있기 때문입니다. 주로 원래의 뷰를 위로 올려주거나 내려주고, 스크롤을 옮겨주거나 하는 역할을 맡습니다. 
  시스템 키보드는 First Responder가 될 때 등장하고, First Responder가 아니게 될 때 사라지므로 First Responder와 편집 모드는 거의 동일하다고 볼 수 있습니다.  

  시스템 키보드도 하나의 뷰로써, 터치 이벤트를 받아서 현재의 First Responder에게 포워딩하면 First Responder는 이 이벤트를 받아 현재 뷰의 텍스트를 변화시키게 됩니다.