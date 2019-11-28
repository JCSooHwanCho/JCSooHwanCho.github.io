---
layout: post
title: Key-Value Coding(KVC)
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

Key-Value Coding, 통칭 KVC는 애플 프래임워크에서 중요한 위치를 차지하고 있는 기술입니다. 이번 포스트에서는 이 KVC에 대해서 알아보겠습니다.  

* **KVC의 기초**  
   KVC는 어떤 객체에 문자열(Key)를 이용하여 해당 객체의 프로퍼티 값(Value)에 간접적으로 접근하게 만들어주는 Objective-C의 특징적인 기술입니다. Objective-C 에서 객체의 프로퍼티에 접근하기 위해서는 getter, setter등의 접근자(accesor)를 이용하거나, 해당 프로퍼티 아래의 실제 값에 직접 접근하는 방식을 사용합니다. 이러한 방식은 직관적이고 간결하지만, 프로퍼티가 늘어날수록 이러한 프로퍼티들의 접근자 코드 역시 비례해서 늘어납니다. KVC가 적용된 객체는 이러한 프로퍼티 접근을 Objective-C에서의 메소드 호출 방식인 메시징 인터페이스로 일원화시켜줍니다.  

   KVC는 Objective-C 런타임에 의존하는 기술이기 때문에 Swift에서 사용하기 위해서는 해당 객체가 NSObject를 상속받아야 하며, 프로퍼티에 @objc 어노테이션을 붙여주어서 해당 프로퍼티를 Objective-C 런타임에 노출시켜야 합니다. 기본적인 구현은 모두 NSObject가 가지고 있기 때문에 추가적으로 구현해 줄 것은 없습니다. 특히 Swift의 모든 타입은 객체이기 때문에 객체가 아닌 타입이 존재하는 Objective-C보다 더 간단하게 적용할 수 있습니다.  

   ```swift
    class Derived: NSObject {
    @objc let x: Int
        override init() {
            x = 1
            super.init()
        }
    }

    let d = Derived()

    print(d.value(forKey: "x"))
    // Optional(1)
   ```  

* **KVC 사용하기**  
   상술했듯이,  KVC를 활용하기 위해서는 NSObject를 상속 받아야 하고, 원하는 프로퍼티를 Objective-C 런타임에 노출시켜야 합니다. 상속이 필요하기 때문에 클래스에서만 사용할 수 있습니다.  

   > Swift의 일부 타입은 Objective-C로 변환될 수 없습니다. 예를 들어 swift의 Int는 객체지만, Objective-C에서는 객체가 아니기 때문에 그냥 쓰면 괜찮지만, 옵셔널이 될 경우 Objective-C 런타임이 이해할 수 없게 되면서 컴파일 오류가 나게 됩니다. 이럴 때는 NSNumber등으로 한번 감싸줌으로써 해결할 수 있습니다.

   기본적으로는 [value(forKey:)](https://developer.apple.com/documentation/objectivec/nsobject/1412591-value) 로 값을 가져오고, [setValue(_:forKey:)](https://developer.apple.com/documentation/objectivec/nsobject/1415969-setvalue)로 값을 설정할 수 있습니다.  

   ```swift
    class Derived: NSObject {
    @objc var x: NSNumber?
    override init() {
        x = nil
        super.init()
    }
    }

    let d = Derived()

    print(d.value(forKey: "x"))

    d.setValue(10, forKey: "x")

    print(d.value(forKey: "x"))

    // nil
    /// Optional(10)
   ```

   만약 여기서 키에 해당하는 프로퍼티가 없을 경우에는 [value(forUndefinedKey:)](https://developer.apple.com/documentation/objectivec/nsobject/1413457-value)나 [setValue(_:forUndefinedKey:)](https://developer.apple.com/documentation/objectivec/nsobject/1413490-setvalue)가 호출되는데, 이 함수는 기본적으로 런타임 에러를 발생시키도록 되어 있습니다. (다만, 서브클래스에서 재정의하여 다른 동작을 취하도록 할 수는 있습니다)

   이에 대한 변형으로, 여러개의 key를 한번에 검색해서 dictionary에 담아 반환해주는 [dictionaryWithValues(forKeys:)](https://developer.apple.com/documentation/objectivec/nsobject/1411319-dictionarywithvalues), [key:value] 형태의 dictionary를 넘겨서 여러 key를 한꺼번에 변경해줄 수 있는 [setValuesForKeys(_:)](https://developer.apple.com/documentation/objectivec/nsobject/1417515-setvaluesforkeys) 도 존재합니다.  

    forKey: 라는 라벨이 붙은 메소드들은 기본적으로 자기 자신의 프로퍼티만 변경할 수 있습니다. 하지만 객체는 또 다른 객체를 소유할 수 있고, 이런 식으로 객체는 계층 구조를 가집니다. 이런 것을 매번 검색해서 순차적으로 찾아가는 것은 여간 번거로운 것이 아닙니다. 이를 위해 **KeyPath**라는 것이 존재합니다. 이는 단순히 원하는 프로퍼티로 가는 경로를 나타내는 것으로 "(propertyName).(propertyName)" 형식으로 .으로 연결된 문자열입니다. [value(forKeyPath:)](https://developer.apple.com/documentation/objectivec/nsobject/1416468-value)와 [setValue(_:forKeyPath:)](https://developer.apple.com/documentation/objectivec/nsobject/1418139-setvalue) 등의 forKeyPath 라벨이 붙은 메소드들이 이를 지원해줍니다.  

    Swift 4부터는 KeyPath가 표준 라이브러리에 별도 타입으로 들어오게 되었습니다. 이렇게 되면서 String 타입이였을 때는 할 수 없었던 컴파일 타임 유효성 체크가 가능하게 되었습니다. 다만, Dictionary 방식으로 여러 값에 동시에 접근 하는 방식으로는 활용할 수 없습니다. 문법도 아래와 같이 subscript를 이용하는 방법으로 변경되었습니다.

    ```swift
    class Derived: NSObject {
    @objc let value : NSNumber?
        override init() {
            value = 1
            super.init()
        }
    }

    let d = Derived()

    print(d[keyPath: \Derived.value])
    // Optional(1)
    ``` 

---  

기본적인 KVC에 대해서 알아보았습니다. 이후 포스트를 통해서 KVC와 관련된 기술들에 대해서 좀 더 알아보도록 하겠습니다.

> 참고 자료 
> [Key-Value Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/index.html#//apple_ref/doc/uid/10000107i)