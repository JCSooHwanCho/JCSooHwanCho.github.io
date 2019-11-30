---
layout: post
title: Key-Value Observing(Key-Value Observing)
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

이번 포스트에서는 프로퍼티의 변화를 감시하고 이에 대해 알림을 받아 특정한 동작을 수행하도록 하는 KVO(Key-Value Observing)에 대해서 알아보도록 하겠습니다.  

* **KVO란?**  
  KVO는 객체의 프로퍼티 변화를 감시하고, 변화할 때에 맞춰서 필요한 동작을 수행할 수 있도록 합니다. 한 예시로 MVC 아키텍처에서 모델이 변화하면, 다른 요소들도 그에 맞춰 변화해서 시스템 전체의 일관성을 유지해야 합니다. 컨트롤러에서 모델에 주기적으로 쿼리를 넣어서 변화 여부를 확인하는 방법도 가능하겠지만, 변화가 매번 일어나는 게 아니다 보니 아무래도 비효율적입니다. 대신, 컨트롤러는 모델에 감시자(Observer)를 붙이고, 모델이 변화할 때 모델은 이 감시자에게 알림을 주면 감시자는 컨트롤러의 로직을 수행할 수 있도록 하면,  변화가 일어날 때만 필요한 동작을 수행할 수 있게 되어서 좀 더 효율적입니다. 

* **KVO 사용하기**  
   KVO를 사용하기 위해서는, 관련된 객체 모두가 KVO를 지원해야 합니다. 이를 위해서는 해당 객체가 NSObject를 상속받아야 하고(그렇기 때문에 당연히 클래스만 가능합니다) 변화를 관찰하려는 프로퍼티를 Objective-C 런타임에 노출시키고 Message Dispatch를 사용하도록 dynamic 키워드를 붙여야 합니다. @objc 어노테이션을 붙여도, dynamic을 붙이지 않으면 KVO가 동작하지 않습니다. 

   다음은 KVO를 사용하는 예제 코드입니다.  

   ```swift
    import Foundation
    import CoreGraphics

    class Derived: NSObject {
        @objc dynamic var rect: CGRect = CGRect(x: 0, y: 0, width: 0, height: 0)
    }

    class Observer: NSObject {
        let d = Derived()

        override init() {
            super.init()
            d.addObserver(self, forKeyPath: "rect", options: [.new, .old], context: nil)
        }

        override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
            if let old = change?[.oldKey] as? CGRect,
                let new = change?[.newKey] as? CGRect {
                print("\(old) -> \(new) updated")
            }
        }

        func changeValue(_ rect: CGRect) {
            self.d.rect = rect
        }
    }

    let d = Observer()

    d.changeValue(CGRect(x: 0, y: 0, width: 100, height: 100))
    // (0.0, 0.0, 0.0, 0.0) -> (0.0, 0.0, 100.0, 100.0) updated
   ```  

   KVO를 적용하려면 다음 과정을 거치게 됩니다.  

   1. Observer 등록 : [addObserver(_:forKeyPath:options:context:)](https://developer.apple.com/documentation/objectivec/nsobject/1412787-addobserver) 메소드를 통해서, 옵저버를 등록합니다.
      
      1. observer: Observer 역할을 할 객체 입니다. NSObject 타입의 객체여야 합니다.
      
      2. keyPath: 감시하고자 하는 프로퍼티를 지정합니다. 여기서는 KeyPath를 String으로 지정해야 합니다. 
      
      3. options: 값이 변화할 때, 어떤 시점의 값을 돌려받을 지 지정합니다. 기본 값은 아무런 옵션이 없는 상태이고, 옵션들을 배열에 담아 넘기는 방식으로 여러개 옵션을 지정할 수 있습니다. 이렇게 지정된 옵션은 dictionary에 담겨서 반환되게 됩니다.
         
        1. old: 변경 이전 값을 dictionary에 담습니다. 이는 [oldKey](https://developer.apple.com/documentation/foundation/nskeyvaluechangekey/1415014-oldkey)로 참조할 수 있습니다.  
         
        2. new: 새로 변경된 값을 dictionary에 담습니다. 이는 [newKey](https://developer.apple.com/documentation/foundation/nskeyvaluechangekey/1416259-newkey)로 참조할 수 있습니다.  
         
        3. initial: Observer 등록이 완료되기 이전에 Observer에게 알림을 한 번 보내주도록 합니다. 이 경우, 현재의 값을 newKey에 담아서 보내줍니다.
        
        4. prior: 변경이 일어날 때, 변경 전후로 알람을 별도로 보냅니다. willChange 계열의 메소드를 직접 구현하려 한다면 유용하게 사용할 수 있습니다.  
  
  2. [observeValue(forKeyPath:of:change:context:)](https://developer.apple.com/documentation/objectivec/nsobject/1416553-observevalue) 오버라이딩: Observer로 지정된 객체는, 해당 메소드를 오버라이드 해야 합니다. 
     
     1. keyPath: observer가 감시하고 있던 프로퍼티의 KeyPath입니다.
     
     2. object: observer가 감시하는 객체의 참조입니다.
     
     3. change: 객체의 변화에 대한 내용을 담은 dictionary 객체입니다. 
     
     4. context: 감시하고 있던 객체의 추가 context입니다.   
  
  3. [removeObserver(_:forKeyPath:)](https://developer.apple.com/documentation/objectivec/nsobject/1408054-removeobserver): 만약 감시를 중단하고싶다면, removeObserver로 Observer를 제거할 수 있습니다. 

  > iOS 9 이전에는 객체가 해제되기 전, removeObserver를 해주지 않으면 메모리 릭이나 크래시 등이 발생했지만, iOS 9 이후로는 시스템이 자동으로 이를 해주기 때문에 굳이 호출해주지 않아도 됩니다.  

  Swift 4 이후로는 이 KVO 문법도 Swift스럽게 바뀌었습니다. 변경점은 클로저를 활용함으로써, 오버라이딩이나 context등이 사라진 것입니다. 대신, addObservser 대신 사용하는 observe 메소드가 NSKeyValueObservation이라는 객체를 반환하고, 해당 객체가 힙에 유지되고 있어야만 감시가 이루어진다는 점이 차이점입니다. 즉, Observer를 등록( add)하는 방식에서, Observation  객체를 소유하는 객체가 Observer 역할을 암시적으로 수행하는 방식으로 바뀐 것입니다.

  ```swift
    class Derived: NSObject {
        @objc dynamic var num: CGRect = CGRect(x: 0, y: 0, width: 0, height: 0)
    }

    class Observer: NSObject {
        let d = Derived()
        let observing: NSKeyValueObservation

        override init() {
            observing = d.observe(\Derived.num, options: [.new, .old]) { d, change in
                if let old = change.oldValue {
                    print("\(old) -> \(d.num) updated")
                }
            }
            super.init()
        }

        func changeValue(_ n: CGRect) {
            self.d.num = n
        }
    }

    let d = Observer()

    d.changeValue(CGRect(x: 0, y: 0, width: 100, height: 100))
  ```

* **Swift의 Property Observer와의 비교**  
  Swift는 KVO와 비슷한 Property Observer를 제공합니다.

  ```swift
    struct Observer {
        var rect: CGRect {
            willSet {
                print("\(self.rect) will change to \(newValue)")
            }
            didSet {
                print("\(oldValue) changed to \(self.rect)")
            }
        }

        mutating func changeValue(_ new: CGRect) {
            rect = new
        }
    }

    var d = Observer(rect: CGRect(x: 0, y: 0, width: 0, height: 0))

    d.changeValue(CGRect(x: 0, y: 0, width: 100, height: 100))  
    // (0.0, 0.0, 0.0, 0.0) will change to (0.0, 0.0, 100.0, 100.0)
    // (0.0, 0.0, 0.0, 0.0) changed to (0.0, 0.0, 100.0, 100.0)
  ```  
  
  KVO와 비슷하지만, NSObject를 상속받을 수 없는 값 타입에도 적용 할 수 있다는 점이 장점입니다. 하지만 Property Observer는 타입 선언의 일부이기 때문에 정적이고, KVO는 런타임에 동적으로 추가하는 것이라는 차이가 있습니다.  

---  

Apple의 프레임워크는 KVO를 적극적으로 활용하고 있습니다. 특히 요즘 많이 등장하는 개념인 데이터 바인딩(Data Binding)을 외부 프레임워크에 의존하지 않고 구현할 수 있다는 점에서, 반드시 알아두어야 알 기술이라 할 수 있겠습니다. 다만 기본적으로 Objective-C의 기능을 활용하는 것이고, Swift 스타일이 적용된지 얼마 되지 않았기 때문에 대부분의 코드가 옛날 스타일로 되어 있다는 점에 주의할 필요가 있겠습니다.