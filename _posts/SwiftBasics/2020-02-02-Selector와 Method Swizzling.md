---
layout: post
title: Selector와 Method Swizzling
comments: true
tags: [Swift,Apple,Objective-C]
category: [SwiftBasics]
---  

이번 포스트에서는 셀렉터(Selector)와 이를 이용한 테크닉인 메소드 스위즐링(Method Swizzling)에 대해서 알아보도록 하겠습니다.

* **Selector란?**  
  Selector는 Objective-C에서 사용하던 개념으로, Objective-C의 객체의 메소드를 문자열을 통해 지정하고 호출하는 방법입니다. KVC의 메소드 버전이라고 생각하시면 됩니다. 실제로는 해당 메소드가 존재하는 메모리의 포인터 값이며, 이를 이용해서 Objective-C 런타임은 실제 호출할 함수를 동적으로 결정하게 됩니다. 동적으로 결정하게 된다는 것은 컴파일 타임에는 어떤 메소드를 호출할지 알 수 없고, 실행 시점에서야 실제 호출할 메소드를 결정한다는 의미입니다. 이 때 [이전 포스트](https://jcsoohwancho.github.io/2019-11-02-Message-Dispatch/)에서 다룬 message dispatch가 사용됩니다. 

  Swift2 이전에는 셀렉터를 만들 때 문자열 리터럴을 그대로 이용했지만 Swift3 이후로는 #selector() 문법을 이용해서 셀렉터 객체를 만들 수 있도록 하였습니다. 본질은 달라지지 않았지만, 이렇게 하면 컴파일러가 셀렉터의 유효성을 알 수 있어서 빠르게 에러를 잡아내고, 자동완성 등의 편리한 기능등을 제공해 줄 수 있게 되었다는 것은 큰 장점입니다.

* **Method Swizzling**  
  호출할 메소드를 실행시점에서야 결정하기 때문에, 만약 메소드를 결정하는데 사용하는 정보를 조작할 수 있다면 기존 메소드 대신에 우리가 원하는 메소드를 호출할 수 있을 것입니다. Objective-C 런타임은 이를 가능하게 하는 인터페이스를 제공해주며, Swift에서도 마찬가지로 이를 사용할 수 있습니다. 이렇게 메소드의 구현을 런타임에 동적으로 변경하는 행위를 Method Swizzling이라고 합니다. 다음은 한 클래스에서 두 인스턴스 메소드의 구현체를 서로 바꾸는 과정입니다.

  1. Swizzling을 할 클래스를 준비합니다. 해당 클래스는 NSObject를 상속해야 하며, Swizzling할 메소드는 메소드에 @objc 혹은 클래스 전체에 @objcMembers 어노테이션을 붙여서 Objective-C 런타임에 노출시켜야만 합니다. 또한 message dispatch를 사용해야 하므로 dynamic을 붙여주어야 합니다. 혹은 extension에서 정의하면 dynamic을 붙이지 않아도 message dispatch가 적용됩니다. 예제로 사용할 클래스는 다음과 같습니다.  

      ```swift
      class Printer: NSObject {

          @objc dynamic func print(_ str: String) {
              Swift.print(str)
          }

          @objc dynamic func swizzledPrint(_ str: String) {
              print("Swizzled: \(str)")
          }
      }
      ```  
    실행할 예제 코드는 다음과 같습니다.  

      ```swift
      let printer = Printer()

      printer.print("Hello, World!") // Hello, World!
      printer.swizzledPrint("Hello, World!") // Swizzled: Hello, World!
      ```  

  2. Swizzling할 두 메소드의 Selector를 준비합니다. 

      ```swift
      let originalSelector = #selector(print(_:))
      let swizzledSelector = #selector(swizzledPrint(_:))
      ``` 
  
  3. [class_getClassMethod(_:_:)](https://developer.apple.com/documentation/objectivec/1418887-class_getclassmethod), [class_getInstanceMethod(_:_:)](https://developer.apple.com/documentation/objectivec/1418530-class_getinstancemethod)를 통해 클래스 메소드 혹은 인스턴스 메소드의 주소 값을 구합니다. 여기서는 인스턴스 메소드를 Swizzling할 것이기 때문에 다음과 같이 써줍니다.  
   
      ```swift
      let originalMethod = class_getInstanceMethod(Printer.self, originalSelector)
      let swizzledMethod = class_getInstanceMethod(Printer.self, swizzledSelector)
      ```  
  
  4. 이번에는 메소드의 구현체를 구합니다. 이때는 [class_getMethodImplementation(_:_:)](https://developer.apple.com/documentation/objectivec/1418811-class_getmethodimplementation)를 이용합니다.  

      ```swift
      let originalImplementation = class_getMethodImplementation(Printer.self, originalSelector)
      let swizzledImplementation = class_getMethodImplementation(Printer.self, swizzledSelector)
      ```  

  5. 3,4에서 구한 메소드와 구현을 [method_setImplementation(_:_:)](https://developer.apple.com/documentation/objectivec/1418707-method_setimplementation)을 통해 연결해줍니다. 이때, 3,4에서 구한 결과 값이 모두 Optional이기 때문에, 이 옵셔널을 벗겨내고 사용해야 합니다.  

      ```swift
      if let originalMethod = originalMethod,
      let swizzledMethod = swizzledMethod,
      let originalImplementation = originalImplementation,
      let swizzledImplementation = swizzledImplementation {
          method_setImplementation(originalMethod, swizzledImplementation)
          method_setImplementation(swizzledMethod, originalImplementation)
      }
      ```  

    이제 swizzle을 하는 코드를 실행한 뒤, 앞에서 실행했던 코드를 다시 실행한 결과는 다음과 같습니다.  

    ```swift
    Printer.swizzle() // swizzling을 해줍니다. 메소드 호출이 이루어지기 전에 반드시 이루어져야 합니다.

    let printer = Printer()

    printer.print("Hello, World!") // Swizzled: Hello, World!
    printer.swizzledPrint("Hello, World!") // Hello, World!
    ```  
    보다시피 메소드의 구현체가 서로 바뀌어 있습니다. 여기서 4번과 5번 과정을 묶어서 수행해주는 [method_exchangeImplementations](https://developer.apple.com/documentation/objectivec/1418769-method_exchangeimplementations) 함수를 이용하면 코드를 더욱 단순화 시킬 수 있습니다. 또한 구현체를 잘못 할당하여 다른쪽 메소드에 대한 참조를 잃어버리는 경우도 막을 수 있습니다. 

    ```swift  
    method_setImplementation(originalMethod, swizzledImplementation)
    method_setImplementation(swizzledMethod, swizzledImplementation) // originalImplementation이 들어갈 자리! 
    // 이 상태로 originalImplementation 참조가 사라질 경우, 해당 메소드 구현은 다시 사용할 수 없게 됩니다.
    ```  

    swizzle된 함수에서, swizzle하기 이전에 메소드를 호출해야 하는 경우가 있습니다. 기능 자체를 바꾸는 게 아니라, 앞뒤로 추가적인 동작을 수행하도록 하고  싶은 경우가 그런 경우인데, 이 경우에는 함수 구현체 내에서의 호출도 실행될 때에야 결정된다는 특성을 이용해서 다음과 같이 작성합니다. 이는 자칫 무한 재귀에 빠질 것 처럼 보이는 코드이지만, 실제로는 정상적으로 호출됩니다.

    ```swift
    extension Printer {
      @objc func print(_ str: String) {
        Swift.print(str)
      }

      @objc func swizzledPrint(_ str: String) {
        Swift.print("Swizzled: \(str)")
        self.swizzledPrint("exchanged: \(str)") // 원래의 print메소드가 호출됨, swizzle이 안된 경우에는 무한 재귀
      }
    }
    ```  

* **Swizzling의 유용성과 위험성**  
  Swizzling은 기존 메소드 구현을 바꿀 수 없는 상황에서 기능 자체를 바꿔버리거나 앞뒤로 추가적인 작업을 수행하고 싶을 때 유용하게 사용할 수 있습니다. 다만 런타임에 프로그램 동작을 바꾸는 작업이기 때문에 잘못 바꿨을 경우 런타임 에러를 야기할 수 있습니다. 따라서 반드시 필요한 경우에만 제한적으로 사용하도록 주의를 기울일 필요가 있습니다.