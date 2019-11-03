---
layout: post
title: Message Dispatch
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

[지난 포스트](../2019-11-01-Swift의-Dispatch-규칙/)에서 Dispatch 규칙에 대해서 알아보았습니다. 이번 포스트에서는 Swift가 지원하는 또 하나의 Dispatch 방법인 Message Dispatch와 이를 활용하는 방법에 대해서 알아 보겠습니다.  

* **Message Dispatch란?**  
  Message Dispatch는 Dynamic Dispatch의 일종입니다. 이전 포스트에서 이야기 했던 Dynamic Dispatch를 이 Message Dispatch와 구분하기 위해 Table Dispatch라고도 합니다.  

  Message Dispatch와 Table Dispatch는 메소드 리스트를 유지하고 검색하는 방법에서 차이를 보입니다. Table Dispatch의 경우는 자기 자신이 처리할 수 있는 모든 메소드에 대한 포인터를 유지하고 있습니다. 부모 타입으로부터 상속받은 메소드의 경우에는 같은 주소값을 유지하고, 오버라이드하게 되면 이를 덮어씁니다. 메소드를 호출 할 때 이 테이블을 참조하여 실제 호출될 메소드를 찾아냅니다.  

  ![TableDispatch]({{"/img/MethodDispatch/TableDispatch.png"}}){: .center-block :} 

  Message Dispatch는 자기 자신이 오버라이드 하거나 새로 정의한 메소드들만 테이블에 유지합니다. 대신 부모 타입으로의 포인터를 가지고 있어서, 부모 타입의 메소드들은 부모 타입에서 찾아서 실행합니다. 이러한 방식은 굉장히 유연해서, 아예 런타임에 메소드의 동작을 수정하는 것부터 새로운 메소드나 프로퍼티를 수정하는 등, 아예 클래스를 동적으로 만드는 것도 가능합니다. 다만, 이러한 기능을 제공하는 런타임 라이브러리가 필요한데 Swift는 이러한 기능을 자체적으로 제공하지 않습니다. 하지만 Swift는 Objective-C 런타임을 이용할 수 있고, Objective-C 런타임이 이러한 기능을 제공합니다. 즉, Message Dispatch를 이용하기 위해서는 Objective-C 런타임에 의존해야 합니다. 
  
  ![MessageDispatch]({{"/img/MethodDispatch/MessageDispatch.png"}}){: .center-block :} 

* **Message Dispatch 적용하기**  
   Swift의 클래스는 Objective-C의 클래스에서 Message Dispatch 능력을 뺀 것입니다. 따라서 원한다면 Objective-C 런타임과 연결해서 Message Dispatch 기능을 부활시킬 수 있습니다. Swift에서 Message Dispatch를 사용하기 위해서는 특정 멤버가 Objective-C의 런타임을 사용하겠다는 것을 명시적으로 알려줘야 합나다. 이를 위해 다음과 같은 과정을 거칩니다.  
  
  1. @objc 어노테이션을 선언 앞에 추가합니다. 이 어노테이션은 해당 요소가 Objective-C 런타임에 의해 접근 될 수 있게 합니다. 하지만 Objective-C 방식(Selector)등으로 접근하지 않는 경우는, 원래의 Dispatch가 적용됩니다.  
  
  2. dynamic 변경자(modifier)를 선언 앞에 추가합니다. 이 변경자의 경우는 해당 요소가 Dynamic dispatch를 사용하도록 유도합니다. 
   
     Swift 4 이전에는 dynamic을 쓰면  @objc 를 자동으로 추론해서 추가시켜 줬는데, [이 Proposal](https://github.com/apple/swift-evolution/blob/master/proposals/0160-objc-inference.md)이 Swift4 이후에 적용되어 dynamic만 써서는 @objc를 자동으로 추가해주지 않습니다.(단, @objc 어노테이션이 적용된 프로토콜의 메소드를 구현하거나 클래스 전체에 @objcMembers 어노테이션이 적용되어 있다면 자동 추론이 됩니다.) 하지만 Message Dispatch가 강제되는 상황이 아니면 컴파일 오류를 내지 않습니다. 하지만 이상태로 Objective-C의 기능(Selector, KVO 등)을 사용하면 런타임 에러가 납니다. 
     
     게다가 dynamic을 붙인다고 Message Dispatch가 되는 것도 아니니, 현재로써는 @objc와 같이 붙지 않으면 아무런 역할을 못합니다. 이는 추후에 swift가 Objective-C에 의존하지 않게 되면 다른 기능을 할당할 수도 있겠지만, 지금으로써는 기대하기가 어렵습니다.  

  위 두개를 합쳐서 **@objc dynamic** 형태가 되면, 해당 메소드나 프로퍼티는 이제 무조건 Objective-C 런타임을 거쳐 Message Dispatch형태로 참조가 이루어집니다.  

* **Message Dispatch를 사용해야 할 때**  
  Message Dispatch는 최대한 최적화되어 구현되어 있지만, 아무래도 런타임에 하는 일이 비교적 많다보니 많아지면 느릴 수 밖에 없습니다. 하지만 반드시 사용할 수 밖에 없는 경우가 있어서, 완전히 배제할 수도 없습니다. 

  1. extension에서 함수 오버라이드  
    기본적으로 Swift는 extension에서의 함수 오버라이드를 금지하고 있습니다. 이를 시도하면 다음과 같은 에러가 발생합니다.  

        ```swift
        class someClass {
        func action() -> Int { 1 }
        }

        class derived: someClass {}

        extension derived {
            override func action() -> Int { 2 } // Error! Overriding non-@objc declarations from extensions is not supported
        }
        ```  
    에러 메시지만 믿고 @objc만 붙이면 또 에러가 발생합니다.
    
        ```swift
        class someClass {
        @objc func action() -> Int { 1 }
        }

        class derived: someClass {}

        extension derived {
            override func action() -> Int { 2 } // Error! Cannot override a non-dynamic class declaration from an extension
        }
        ```  

        정답은 둘다 붙이는 것입니다.  

        ```swift
        class someClass {
        @objc dynamic func action() -> Int { 1 }
        }

        class derived: someClass {}

        extension derived {
            override func action() -> Int { 2 } // OK
        }
        ```  

        반대로 extension에서 처음 선언한 메소드를 오버라이드 가능하게 하기 위해서는 반드시 extension에서 @objc dynamic을 붙여줘야 합니다.  

  2. Selector, KVO 등의 기능을 사용할 때
    이러한 기능들은 Objective-C 런타임을 통해서만 제공되는 기능이기 때문에 어쩔 수 없습니다.  

  3. NSObject와 그 서브클래스들을 이용할 때
     NSObject 자체가 뭔가 특별한 건 아닙니다. 하지만 그 근본적인 특성상 NSObject 자신과 그 구현이 대부분 Objective-C 런타임에 의존하고 있습니다. 이는 코드를 한꺼번에 수정하지 않고도 Swift에서 기존 프레임워크를 이용할 수 있도록 하는 장점이 있지만, 최적화 문제와 Swift로 처음 진입한 개발자에게는 혼란과 진입 장벽이 되기도 합니다.  
  
* **정리**  
  
  |         종류          |        Dispatch        |
  | :-------------------: | :--------------------: |
  | 값 타입(struct, enum) |         Static         |
  |   참조 타입(class)    | Table, 가능하면 Static |
  |       Protocol        |         Table          |
  |       extension       |         Static         |
  |     @objc dynamic     |        Message         |


---  

이 부분은 완벽히 문서화 된 것이 없고, Swift 버전마다 조금씩 달라져 왔습니다. 이러한 것들은 계속된 토론과 버그 수정을 통해 합리적이고 예측 가능한 방향으로 변하고 있지만, 어떻게 바뀔지는 아무도 알 수 없습니다. 이 글은 Swift 5.1을 기반으로 작성되었고, 이후에 수정이 되면 다시 한번 알아보도록 하겠습니다.  

> 참고 자료  
> [Method Dispatch in swift](https://www.rightpoint.com/rplabs/switch-method-dispatch-table)  
> [SE-0160 Limiting @objc inference](https://github.com/apple/swift-evolution/blob/master/proposals/0160-objc-inference.md)  
> [Objective-C vs Swift messages dispatch](https://blog.untitledkingdom.com/objective-c-vs-swift-messages-dispatch-9d5b7fd58327)