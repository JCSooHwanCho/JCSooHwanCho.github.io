---
layout: post
title: NotificationCenter 살펴보기
comments: true
tags: [iOS 프로그래밍,Swift,Apple, 부스트코스]
category: [iOS]
---  

이번 포스트에서는 iOS의 Notification에 대해서 알아보도록 하겠습니다. Notification은 앱 내에서 비동기적으로 이벤트를 처리하는 데 있어서 상당히 중요한 역할을 합니다.  

> 이 글은 공식 문서와 다음 글들을 참고하여 작성되었음을 밝힙니다.  
>  [Notification Programming Topics](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Notifications/Introduction/introNotifications.html#//apple_ref/doc/uid/10000043i)  
> [How to send notifications asynchronously using NotificationQueue](https://www.hackingwithswift.com/example-code/system/how-to-send-notifications-asynchronously-using-notificationqueue)
 
* [Notification](https://developer.apple.com/documentation/foundation/notification)은 앱 상에서 발생하는 이벤트들을 캡슐화한 것입니다. 또, [NotificationCenter](https://developer.apple.com/documentation/foundation/notificationcenter)라는 것이 있어서, Notification을 받고자 하는 객체는 Center에 자신을 등록해놓고 Center는 Notification이 발생하면 등록한 객체에 이를 전달하는 역할을 합니다. 또한 NotificationQueue를 이용하여 Notification을 일정시간 지연시키거나, 비슷한 Notification을 묶어서 한번만 전송하는 것도 가능합니다.  

    Notification은 기존의 객체간 통신 방법이였던 매시지 패싱(message passing)의 단점을 보완하기 위해 나왔습니다. 기존 메시지 패싱방식은 메시지를 보내는 객체가 메시지를 받는 객체와 보내야 하는 메시지를 반드시 알아야만 했습니다. 이 과정에서 객체간의 강한 결합성(Coupling)이 생기게 되는데, 이를 해결하기 위해 브로드캐스트 모델이 나오게 되고, 이것이 Notification과 NotificationCenter입니다. 이것은 [옵저버 패턴](https://ko.wikipedia.org/wiki/옵서버_패턴)의 일종으로 볼 수 있습니다.  

    ![Notification Structure]({{'/img/NotificationStructure.png'}}){: .center-block :}  

    Notification은 Notification의 이름(필수입니다), Notification을 발생시킨 객체(nil로 설정할 수 있습니다, 즉 발신 객체를 명시하지 않을 수 있습니다), 추가로 담아서 보낼 데이터를 담은 Dictionary 구조체(역시 nil일 수 있습니다)로 이루어져 있습니다. 해당 객체를 Notification의 post 메소드를 통해 NotificationCenter에 보내면 Center에 등록되어 있는 리스트를 확인해서 해당 이벤트를 구독한 객체들에게 Notification을 전달하게 됩니다. 보통은 Notification 객체를 직접 만들기보다는 post메소드에 필요 요소들을 인자로 넘기게 됩니다.

> Notification.Name은 NSString의 Wrapper이기 때문에, NSString또는 swift의 String을 이용해 원하는 이름을 줄 수 있습니다. 또한 여러 시스템 프리셋을 제공해서, 필요한 시스템 이벤트를 쉽게 구독하여 받아볼 수 있습니다.  

* NotificationCenter는 Notification을 전달하는 매커니즘이 담긴 객체로, 모든 애플리케이션은 싱글턴 형태로 기본 NotificationCenter를 제공합니다. 사용자는 이 싱글턴 객체를 참조해, addObserver 메소드를 통해서 [클로저 자체를 Center에 등록](https://developer.apple.com/documentation/foundation/notificationcenter/1411723-addobserver)하거나, [객체를 옵저버로 등록](https://developer.apple.com/documentation/foundation/notificationcenter/1415360-addobserver)할 수 있습니다. 또한 post 메소드를 호출해서 Notification을 전달하도록 요청할 수도 있습니다. 또 더이상 옵저버가 필요 없어진 상황에서는 removeObserver 메소드를 호출할 수도 있습니다. 

> iOS 9.0, macOS 10.11 이상 버젼에서는 removeObserver를 명시적으로 호출하지 않아도 자동으로 시스템이 해줍니다. 하지만 이하 버전에서는 객체가 해제될 때 반드시 옵저버를 해제해줘야 하며, 하지 않을 경우 Notification을 처리하는 과정에서 앱이 크래시나게 됩니다.
 
> 등록된 옵저버의 수가 많아질수록, NotificationCenter가 Notification을 보내는 시간이 길어집니다. 이를 고민할 정도로 느려졌다면, Notification을 적절히 카테고리화하여 여러개의 Center를 만드는 것도 고려해볼만 합니다.

> Notification은 단일 앱 안에서만 동작하도록 설계되어 있습니다. 여러 앱 간에 메시지를 보내기 위해서는 DistributedNotificationCenter를 사용해야 합니다. 하지만 해당 클래스가 iOS에는 없기 때문에 앱 개발자들에게는 고려 대상이 아닙니다.  

* post 메소드는 동기메소드(synchronized method)이기 때문에, Notification 발송이 모두 끝나기 전까지는 사용자가 컨트롤을 할 수 없게 됩니다. 이를 해결하기 위해 NotificationQueue라는 것을 사용할 수 있는데, NotificationCenter의 Buffer역할을 하는 객체입니다. 이 버퍼를 통해 Notification을 비동기적으로 처리할 수 있게 되고, 거기에 더해 Notification 이름, Notification을 보낸 객체 혹은 둘 다를 기준으로 Notification을 합져서 한꺼번에 처리하는 기능(coalescing)을 제공합니다.

    NotificationQueue역시 NotificationCenter와 마찬가지로 싱글턴으로 기본 인스턴스를 제공하는데, Center와는 다르게 스레드 별로 별도의 Queue를 가지고있어서, 같은 프로퍼티를 통해 참조하되 반환 값은 스레드마다 다릅니다. 이 Queue에 [enqueue](https://developer.apple.com/documentation/foundation/notificationqueue/1413873-enqueue) 메소드를 통해 Notification을 집어넣으면, 설정한 조건에 맞게 비동기적으로 Notification이 처리되게 됩니다.

    * postingStyle 매개변수는 열거형으로, 해당 Notification이 언제 Center로 post 되는지를 나타냅니다.  
      * asap : 현재 스레드의 runloop가 끝나는 대로 post합니다.  

      * whenIdle : 현재 스레드가 idle 상태가 되면 post됩니다.  

      * now : coalescing을 거친 후 즉시 post됩니다. 동기적으로 동작하게 하려면 해당 옵션을 줍니다. 
    
    > 동기적으로 동작하게 하는 목적이라면 그냥 post메소드를 직접 호출해도 되지만, Notification을 합쳐서 처리하게 하려면 enqueue를 사용해야 합니다.

    * coalesceMask 매개변수는 Queue안에 있는 Notification을 확인하고, 기준에 맞는 Notification이 있을 경우 이를 Queue에서 빼내 실제로 Notification을 한번만 호출하게 해줍니다.  
      * none : 해당 notification에 대해서는 coalescing이 일어나지 않습니다.
     
      * onName : 이름이 같은 notification에 대해서 coalescing을 합니다.
     
      * onSender : 같은 객체가 보낸 notification에 대해서 coalescing을 합니다.
    
    > coalesceMask의 타입인 NotificationQueue.NotificationCoalescing 은 OptionSet을 채택한 타입으로 [.onName, .onSender] 와 같은 형태로 옵션을 동시에 줄 수 있습니다.

    * forModes는 현재 thread의 runLoop가 Notification을 post할 수 있는 상태를 제한합니다. nil을 줄 경우 default로 동작합니다.

> runLoop의 대한 내용은 이 포스트의 범위를 벗어나므로 자세한 설명은 생략합니다. 다음번에 기회가 되면 정리해보도록 하겠습니다.
    
----  

Notification을 잘 사용하면, 여러 비동기적인 상황을 처리해야되는 애플리케이션의 코드를 많이 단순화시킬 수 있습니다. 또 Notification의 기반이 되는 디자인 패턴인 옵저버 패턴은 많이들 사용하는 프레임워크인 Rx의 기반이 되는 패턴이기도 합니다. 이러한 이유들을 가져댜 붙이지 않더라도, 좋은 애플리케이션을 만들기 위해서는 꼭 숙지해야 할 부분 중 하나입니다.