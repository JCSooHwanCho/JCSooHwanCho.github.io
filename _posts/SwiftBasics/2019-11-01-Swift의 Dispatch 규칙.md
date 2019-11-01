---
layout: post
title: Swift의 Dispatch 규칙
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

[지난 포스트](../2019-10-11-Dynamic-Dispatch와-성능-최적화)에서 Dynamic Dispatch를 피해서 성능 최적화를 하는 방법을 알아보았습니다. 하지만 swift가 Dynamic Dispatch만 사용하는 것은 아닙니다. Swift는 다양한 Dispatch 방식을 사용하는데, 이번 포스트에서는 어떻게 하면 이러한 Dispatch들을 사용하고 조절할 수 있는 지 알아보도록 하겠습니다.  

* **Static vs Dynamic**  
  Static Dispatch는 컴파일 타임에 실제 호출할 함수를 결정할 수 있기 때문에 함수 호출 과정이 간단하고, 컴파일러가 이것을 최적화할 수 여지가 많습니다. 즉, 속도가 빠릅니다. 하지만 참조 타입에 따라 호출될 함수가 결정이 되기 때문에 서브클래싱의 장점을 누리기는 어렵습니다. 

  반면 Dynamic Dispatch는 런타임에 호출될 함수를 결정합니다. 이를 위해서 Swift 에서는 클래스마다 vtable(Virtual Dispatch Table)이라는 것을 유지합니다. 이는 함수 포인터들의 배열로 표현되며, 하위 클래스가 메소드를 호출할 때 이 배열을 참조하여 실제 호출할 함수를 결정합니다. 이 모든 과정이 런타임에 결정되기 때문에 Static Dispatch에 비하면 추가적인 연산이 필요할 수밖에 없고, 컴파일러가 최적화 할 여지도 많지 않습니다.  

* **Value Type에서의 Dispatch**  
  Value Type에 해당하는 struct와 enum은 상속을 사용할 수 없다는 특징을 가지고 있습니다. 즉, static Dispatch의 단점인 서브클래싱이 불가능하다는 단점을 완벽히 피해갑니다. 따라서 Value Type에는 Static Dispatch가 적용됩니다.  

  ```swift  
  struct someStruct: someProtocol {
    func action() -> Int { 1 }
    }

    let s = someStruct()

    print(s.action()) // 1
  ```  

* **Reference Type에서의 Dispatch**  
   Reference Type에 해당하는 class는 반대로 항상 상속의 가능성에 노출되어 있습니다. 따라서 Dynamic Dispatch를 사용합니다. 그 대신 오버라이딩이 되지 않는다는 것을 컴파일러가 알 수 있다면, 컴파일러가 이를 Static Dispatch로 바꿔줄 수 있습니다.(관련 내용은 [이 포스트](../2019-10-11-Dynamic-Dispatch와-성능-최적화)를 참고해주세요.)  

   ```swift
   class SomeClass{
    func action() -> Int { 2 }
    }

    class DerivedClass: SomeClass {
        override func action() -> Int { 3 }
    }

    var c:SomeClass = SomeClass()
    print(c.action()) // 2

    c = DerivedClass()
    print(c.action()) // 3

   ```
* **Protocol에서의 Dispatch**  
   프로토콜은 구현체를 제공하지 않고, 선언부만 제공합니다. (물론 extension을 통하면 할 수 있지만, 이 부분은 뒤에서 다시 논하도록 하겠습니다.) 따라서 프로토콜을 채택한 타입은 이를 구현해야 하며, 프로토콜을 통해 호출하는 메소드는 프로토콜을 채택한 타입들이 실제로 구현한 메소드들입니다. 그런데 프로토콜 타입의 참조로만 이들을 사용해야 한다면, 해당 인스턴스의 타입에 맞는 메소드를 호출해야 합니다. 이를 위해 프로토콜은 고유의 vTable을 가지게 되며, 특별히 이를 Witness Table이라고 합니다. 즉, 프로토콜 역시 Dynamic Dispatch를 사용합니다.
   
   ```swift
    protocol SomeProtocol {
    func action() -> Int
    }

    class SomeClass: SomeProtocol {
    func action() -> Int { 2 }
    }

    class DerivedClass: SomeClass {
        override func action() -> Int { 3 }
    }

    var c:SomeProtocol = SomeClass()
    print(c.action())

    c = DerivedClass()
    print(c.action())
   ```  

* **Extension에서의 Dispatch**  
   Swift는 모든 타입에서 Extension으로 기능을 추가할 수 있습니다. 각 타입 별로 extension을 사용했을 때의 동작을 확인해보겠습니다.  

   1. 값 타입 : 역시 상속 가능성이 없기 때문에 Static Dispatch로 수행됩니다.  
        ```swift
        struct SomeStruct: SomeProtocol {
            func action() -> Int { 1 }
        }

        extension SomeStruct {
            func anotherAction() -> Int { 4 }
        }  

        let s = SomeStruct()

        print(s.action()) // 1
        print(s.anotherAction()) // 4      
        ```  

    2. 클래스 타입: extension으로 클래스에 추가 기능을 구현할 경우에는 오버라이드 여부에 따라 다음과 같은 규칙을 따릅니다. 
       * 하위 메소드를 오버라이드 하는 경우: 기본적으로 extension에서는 메소드 오버라이드를 금지합니다. 이는 클래스 본체에 선언된 메소드를 오버라이드 하는 것과 extension에서 선언된 메소드를 오버라이드 하는 것 모두 포함입니다. 이러한 기능을 사용하기 위해서는, Objective-C 런타임의 힘을 빌려야 합니다. (이에 대해서는 다른 포스트를 통해 자세히 알아보도록 하겠습니다.)  
       
        * 오버라이드 하거나, 오버라이드 되지 않는 경우: 자기 자신 뿐 아니라 하위 타입에서도 동일한 메소드를 참조함을 보장할 수 있습니다. 따라서 클래스 타입이여도 Static Dispatch가 가능해집니다.  
    
            ```swift
            class SomeClass {
            func action() -> Int { 2 }
            }

            class DerivedClass: SomeClass {
                override func action() -> Int { 3 }
            }

            extension SomeClass {
                func anotherAction() -> Int { 5 }
            }  

            var c:SomeClass = SomeClass()
            print(c.anotherAction())// 5

            c = DerivedClass()
            print(c.anotherAction())// 5
            ```

    3. 프로토콜 타입: 프로토콜 역시 extension을 통해 기능을 추가할 수 있습니다. 이 때, 프로토콜 본체에 선언된 멤버의 디폴트 구현체를 제공해주거나 프로토콜 본체에는 없는 기능을 추가적으로 제공해 줄 수 도 있습니다. 
       * 본체에 선언된 멤버의 디폴트 구현체를 제공하는 경우: 하위 클래스들이 메소드들을 구현하고 있음이 반드시 보장됩니다. 설령 구현하지 않았다 하더라도 디폴트 메소드를 이용하면 됩니다. 따라서 Witness Table을 이용한 Dynamic Dispatch가 이루어집니다.  

            ```swift
            protocol SomeProtocol {
                func action() -> Int
            }

            extension SomeProtocol {
                func action() -> Int { 4 }
            }

            class SomeClass: SomeProtocol {
                func action() -> Int { 2 }
            }

            class DerivedClass: SomeClass {
                override func action() -> Int { 3 }
            }  

            var c: SomeProtocol = SomeClass() 
            print(c.action()) // 2

            c = DerivedClass()
            print(c.action()) // 3
            ```   
       
       * 본체에 없는 기능을 추가한 경우: 본체에 선언하지 않고 extension으로 추가한 메소드들은 Witness Table을 이용할 수 없습니다. 즉, Dynamic Dispatch를 사용할 수 없고, Static Dispatch가 적용됩니다.  

            ```swift
            protocol SomeProtocol { }

            extension SomeProtocol {
                func action() -> Int { 4 }
            }

            class SomeClass: SomeProtocol {
                func action() -> Int { 2 }
            }

            class DerivedClass: SomeClass {
                override func action() -> Int { 3 }
            }  

            var c:SomeProtocol = SomeClass() 
            print(c.action()) // 4

            c = DerivedClass()
            print(c.action()) // 4
            ```  

지금까지 Swift가 지원하는 Dispatch 방법에 대해서 알아보았습니다. 하지만 Swift는 Objective-C와의 호환성을 위해 또 다른 하나의 방법을 제공합니다. 이에 대해서는 나중에 다시 알아보도록 하겠습니다.  

