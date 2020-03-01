---
layout: post
title: Foundation의 Swift 타입 브릿징
comments: true
tags: [iOS 프로그래밍,Swift,Apple]
category: [iOS]
---  

[지난 포스트](../2020-02-23-Foundation이-제공하는-특별한-Collection들/)에서 이러한 이야기를 했었습니다.  

> NSObject의 자식타입이 아닌 객체를 NSObject의 서브클래스로 래핑해주는 과정  

이번 포스트에서는 이에 대해서 좀 더 자세히 알아보도록 하겠습니다.

* **개요**  
  Foundation에서 사용하는 Collection들의 경우 여러가지 타입을 담을 수 있도록 Any 타입으로 인자를 받는 경우가 있습니다. 그런데 Foundation은 Objective-C에서도 사용되는데, Objective-C에서는 모든 객체가 NSObject를 상속 받은 클래스여야 합니다. Swift의 타입에는 이러한 제한이 없는데, 실제로 해보면 별다른 무리 없이 잘 들어가는 것을 확인할 수 있습니다. Objective-C 에서와 Swift가 다른 Foundation을 사용하고 있는 걸까요? 비결이 무엇일까요?

* **살펴보기**  
  NSArray의 코드를 보면서 이 비밀을 살펴보도록 합시다. 그 중에서도 가장 쉽게 이해할 수 있는 코드인 NSMutableArray의 Insert 함수를 가져오겠습니다.  

  ```swift
  open func insert(_ anObject: Any, at index: Int) {
      // ...생략...

      // _storage: [AnyObject]
      _storage.insert(__SwiftValue.store(anObject), at: index) // _SwiftValue는 무엇인가?
  }
  ```  

  __SwiftValue라는 의문의 타입이 등장합니다. 이 타입의 store메소드에 객체를 넘겨서 이를 _storage 프로퍼티에 담습니다. 이 _storage는 AnyObject 타입의 배열입니다. 즉, __SwiftValue.store(anObject)의 결과가 AnyObject라는 것입니다. 즉 Any 타입은 __SwiftValue 타입을 통해 NSArray가 담을 수 있는 객체(즉, NSObject의 서브클래스)로 변환되는 것이라 추측을 해볼 수 있습니다.

* **__SwiftValue**  
  __SwiftValue 타입의 선언을 간추려서 보면 다음과 같습니다. 프로퍼티와 주요 메소드들만 볼 것이고, 여기서 타입 메소드들은 후에 살펴보기 위해 잠시 제외하겠습니다.

  ```swift  
  internal final class __SwiftValue : NSObject, NSCopying {
      public private(set) var value: Any

      init(_ value: Any) {
          self.value = value
      }
      
      // NSObject의 hash 프로퍼티
      override var hash: Int { 
          if let hashable = value as? AnyHashable { // value가 Hashable을 채택한 타입이면
              return hashable.hashValue // value의 hash값을 내보낸다
          }
          return ObjectIdentifier(self).hashValue // 자신의 Identifier에 기반한 Hash값을 내보낸다.
      }
      
      // NSObject의 isEqual 메소드
      override func isEqual(_ value: Any?) -> Bool {
          switch value { 
          case let other as __SwiftValue: // __SwiftValue로 포장된 값이면
              guard let left = other.value as? AnyHashable,
                  let right = self.value as? AnyHashable else { return self === other } // 한쪽이라도 Hashable이 아니면 Identity를 비교한다.
              
              return left == right // 양쪽다 Hashable하면 양쪽 값의 Hash값을 비교한다.
          case let other as AnyHashable: // 다른 값이 __Swiftvalue로 포장되지 않은 값이면서 Hashable한 경우
              guard let hashable = self.value as? AnyHashable else { return false } // 자신이 Hashable하지 않은 경우 비교가 안된다.
              return other == hashable //  자신이 hashable한 경우, 이를 비교할 수 있다.
          default: // 그 외의 경우는 비교가 안된다.
              return false
          }
      }
      
      // NSCopying의 copy 메소드
      public func copy(with zone: NSZone?) -> Any {
          return __SwiftValue(value)
      }
  }
  ```  

  __SwiftValue는 NSObject의 서브클래스이며, Any 타입의 값 하나를 가지고 있습니다. 비교와 해시를 위해 오버라이딩을 추가로 한 것 이외에는 특별한 게 없습니다. 즉, 이는 NSObject가 아닌 객체를 NSObject 객체로 사용하기 위한 Wrapper입니다. 코드상 주석에서는 에서는 Box라고 표현 됩니다.__SwiftValue는 몇가지 타입 메소드를 가지는데, 그 중 핵심인 메소드 두가지를 살펴보겠습니다. 

    1. store: 객체를 받아서 NSObject형 객체를 반환합니다. 객체마다 NSObject로의 변환 과정이 다르기 때문에 여러 케이스를 고려해야 되는데, 이를 하나의 메소드로 추상화해놓은 것입니다.

        ```swift
        static func store(_ value: Any) -> NSObject {
            if let val = value as? NSObject { // NSObject의 서브타입이면 굳이 래핑할 필요가 없습니다.
                return val 
            } else if let opt = value as? Unwrappable, opt.unwrap() == nil { // 옵셔널이여서 언래핑했는데 nil인 경우를 처리합니다.
                return NSNull()
            } else {
                #if canImport(ObjectiveC) 
                    // AnyObject로 캐스팅하면 자동으로 박싱이 이루어집니다. 
                    // SwiftNative 타입은 개별적인 박스 타입을 가지고 있는 경우가 많고, 사용자 타입의 경우는 __SwiftValue로 박싱이 이루어집니다.
                    let boxed = (value as AnyObject)
                    if !(boxed is NSObject) { // 박스가 NSObject가 아닌 경우는 다시 박싱합니다. 어떤 예외 케이스가 있는지는 아직 확인해보지 못했습니다.
                        return __SwiftValue(value)
                    } else {
                        return boxed as! NSObject
                    }
                #else
                    return (value as AnyObject) as! NSObject // __SwiftValue로 박싱이 되기 때문에 그냥 NSObject로 캐스팅만 해서 내보냅니다.
                #endif
            }
        }
        ```
            
    2. fetch: AnyObject를 Any로 바꿔줍니다. store의 역연산이라고 볼 수 있습니다. 이 역시 여러 케이스를 고려하여야 되기 때문에 하나의 메소드로 묶어 놓은 형태입니다. fetch의 코드를 보면 다음과 같습니다.  

        ```swift
        static func fetch(nonOptional object: AnyObject) -> Any {
            #if canImport(ObjectiveC) // Objective-C를 import할 수 있으면, 애플 플랫폼이면 이것이 성립합니다.

            if type(of: object as Any) == objCNSNullClass { // object가 NSNull 객체일 경우입니다. 여기서 objcNSNullClass는 __SwiftValue의 타입 프로퍼티입니다.
                return Optional<Any>.none as Any 
            }

            if type(of: object as Any) == swiftStdlibSwiftValueClass { // object가 Swift의 타입일 때 입니다. swiftStdlibSwiftValueClass 역시 타입 프로퍼티 입니다.
                return object // 박스에 싸인 그대로 반환합니다. 이 박스는 실제 타입으로 캐스팅 될때, 자동으로 벗겨집니다.
            }
            
            #endif
            
            // object가 Foundation에서 사용하는 타입들이였다면
            if object === kCFBooleanTrue { // Boolean값의 경우는 별도로 처리됩니다.
                return true
            } else if object === kCFBooleanFalse {
                return false
            } else if let container = object as? __SwiftValue { // Objective-C를 Import 할 수 없는 경우에는 Swift 타입이 여기서 처리됩니다.
                return container.value
            } else if let val = object as? _StructBridgeable { // Foundation의 타입 중에서 Swift의 Struct로 캐스팅이 가능한 경우를 처리합니다.
                return val._bridgeToAny()
            } else { // 
                return object
            }
        }
        ```
* **결론**  
   사실 여기서 알아본 부분들은 크게 신경쓰지 않아도 될 부분들이긴 합니다. 그렇지만 Objective-C의 영향은 Swift로 넘어온 지금까지도 많은 영향을 미치고 있기 때문에, 이렇게 가려진 부분들을 살펴보다보면 좀 더 플랫폼에 대한 깊은 이해가 가능할 것이라 생각합니다.