---
layout: post
title: Opaque Type 살펴보기
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

SwiftUI의 예제를 보다보면 낮선 문법이 눈에 띕니다.

```swift
struct FeatureCard: View {
   var landmark: Landmark
   
   var body: some View { // some?
      landmark.featureImage
         .resizable()
         .aspectRatio(3/2, contentMode: .fit)
         .overlay(TextOverlay(landmark))
   }
}
```  

이 some 문법은 swift 5.1에 추가된 **Opaque Type** 이라는 기능입니다. 이 기능이 왜 필요한지, 어떻게 사용될 수 있는지에 대해 알아보도록 하겠습니다.

> 이 포스트는 다음 글을 참조하여 작성되었음을 밝힙니다.  
> [Swift Language Guide - Opaque Type](https://docs.swift.org/swift-book/LanguageGuide/OpaqueTypes.html)  
> [Proposals- SE-0244 - Opaque Result Types](https://github.com/apple/swift-evolution/blob/master/proposals/0244-opaque-result-types.md)  

> 해당 기능은 macOS 10.15, iOS 13부터 적용되는 사항입니다. 해당 버전 이하에서는 사용할 수 없습니다.
  
Opaque Type은 구체적인 타입을 숨기고, 해당 타입이 채택하고 있는 프로토콜의 관점에서 함수의 반환 값이나 프로퍼티를 사용하게 해줍니다. 실제 타입 정보에 대한 것을 감추어 모듈과 모듈 코드의 결합성을 줄이는 이점이 있게 됩니다. 

* 필요성
 
    다음 코드를 한번 보도록 합시다. 

    ```swift
    protocol Shape {
        func draw() -> String
    }

    struct Triangle: Shape {
        var size: Int
        func draw() -> String {
            var result = [String]()
            for length in 1...size {
                result.append(String(repeating: "*", count: length))
            }
            return result.joined(separator: "\n")
        }
    }
    let smallTriangle = Triangle(size: 3)
    print(smallTriangle.draw())
    // *
    // **
    // ***
    ```  

    여기까지는 문제가 없습니다. 그런데 이 Shape를 뒤집은 모양을 나타내는 클래스를 하나 더 정의 한다면 어떨까요? 

    ```swift
    struct FlippedShape<T: Shape>: Shape {
        var shape: T
        func draw() -> String {
            let lines = shape.draw().split(separator: "\n")
            return lines.reversed().joined(separator: "\n")
        }
    }
    // Type ; FlippedShape<Triangle>
    let flippedTriangle = FlippedShape(shape: smallTriangle)
    print(flippedTriangle.draw())
    // ***
    // **
    // *
    ```  

    FilppedShape는 해당 타입을 만들기 위해 사용한 타입 매개 변수를 알 수 밖에 없게 되었습니다. 거기에 다음 코드가 얹어진다면 어떨까요?  

    ```swift
    struct JoinedShape<T: Shape, U: Shape>: Shape {
        var top: T
        var bottom: U
        func draw() -> String {
            return top.draw() + "\n" + bottom.draw()
        }
    }
    // Type : JoinedShape<Triangle, FlipedShape<Triangle>>
    let joinedTriangles = JoinedShape(top: smallTriangle, bottom: flippedTriangle)
    print(joinedTriangles.draw())
    // *
    // **
    // ***
    // ***
    // **
    // *
    ```

    모양이 복잡해질수록, Generic이 덕지덕지 붙고, 불필요한 세부 사항이 계속해서 노출되게 됩니다. 이를 해결하기 위해서는 해당 구조체들이 모두 같은 Shape를 채택하고 있다는 점을 이용해 타입의 정의는 감추고, 모듈의 public 인터페이스에서는 Shape라고만 알려줄 필요가 있습니다. 그렇다면 프로토콜을 반환형으로 하는 메소드를 만든다면 어떨까요?

    ```swift
    func flip<T: Shape>(_ shape: T) -> Shape {
        return FlippedShape(shape: shape)
    }

    func join<T: Shape, U: Shape>(_ shape1: T, _ shape2: U) -> Shape {
        return JoinedShape(top: shape1, bottom: shape2)
    }
    ```  

    괜찮아 보이지만, 실제로 쓰다보면 다음과 같은 문제가 있습니다. 

    ```swift
    let flippedTriangle = flip(smallTriangle)
    let sameThing = flip(smallTriangle)
    flippedTriangle == sameThing  // Error! == 메소드가 없습니다.
    ```  

    프로토콜로 타입 반환을 받으면, 해당 타입에 대한 정보를 이용할 수 없고, 프로토콜의 일부 인터페이스 만 이용할 수 있게 되는 문제가 생기게 됩니다. Shape에 Equatable을 추가할라고 쳐도 타입 정보를 이용할 수 없기 때문에, 자기 자신의 타입 정보가 필요한 == 메소드를 제대로 호출할 수 조차 없습니다.  

    또 다른 문제는 프로토콜로 반환한 값은 중첩해서 사용할 수 없다는 것입니다. 

    ```swift
    let flippedTriangle = flip(smallTriangle)
    let original = flip(filppedTriangle) // Error! 
    ```  

    이는 프로토콜 타입으로 반환한 값은 해당 프로토콜을 채택하지 않은 것으로 여겨지기 때문입니다. 이는 타입 정보를 이용할 수 없는 것의 부작용입니다. 

    결국 프로토콜 타입의 반환값의 문제점은 **실제 타입에 대한 정보를 얻을 수 없다**로 귀결됩니다. 이러한 문제를 해결하기 위해 Opaque Type이라는 것이 도입되게 됩니다.

* Opaque Type의 도입  

    Opaque Type의 예제를 먼저 보겠습니다.
    ```swift
    struct Square: Shape {
    var size: Int
    func draw() -> String {
        let line = String(repeating: "*", count: size)
        let result = Array<String>(repeating: line, count: size)
        return result.joined(separator: "\n")
    }   
    }   

    func makeTrapezoid() -> some Shape { // 반환 값의 실제 타입은 가려집니다.
        let top = Triangle(size: 2)
        let middle = Square(size: 2)
        let bottom = FlippedShape(shape: top)
        let trapezoid = JoinedShape(
            top: top,
            bottom: JoinedShape(top: middle, bottom: bottom)
        )
        return trapezoid
    }

    let trapezoid = makeTrapezoid()
    print(trapezoid.draw()) 
    // *
    // **
    // **
    // **
    // **
    // *
    ```
    프로토콜을 사용한 함수에서 some 키워드만 추가한 형태입니다. 하지만 프로토콜로 구현했을 때와는 다르게 타입 정보를 보존하고 있기 때문에, 해당 타입의 메소드들을 그대로 사용할 수 있게 됩니다.

    Opaque Type을 이용해서 위에서 보았던 filp과 join 을 바꾸어 보면 다음과 같습니다.  

    ```swift  
    func flip<T: Shape>(_ shape: T) -> some Shape { 
        return FlippedShape(shape: shape)
    }

    func join<T: Shape, U: Shape>(_ shape1: T, _ shape2: U) -> some Shape {
        return JoinedShape(top: shape1, bottom: shape2)
    }

    let opaqueJoinedTriangles = join(smallTriangle, flip(smallTriangle))
    print(opaqueJoinedTriangles.draw())
    // *
    // **
    // ***
    // ***
    // **
    // *
    ```  

    Opaque Type은 Generic의 반대라고 볼 수 있습니다. Generic은 호출하는 쪽에서 구체적인 타입을 지정해 주고 호출 당하는 쪽에서는 추상적인 타입이 유지되다면, Opaque Type은 반대로 호출 당하는 쪽에서 구체적인 타입을 제공해주고, 호출하는 쪽에서는 추상적인 타입으로 사용되게 됩니다.

    ```swift
    // 실제로 동작하는 코드가 아닙니다.
    // 개념상으로만 그렇다는 것입니다.
    func reverseGeneric() -> <T: Shape> T { return Rectangle(...) }

    let x = reverseGeneric() // reverseGeneric의 구현에 따라 x의 타입이 결정됩니다.
    ```  

    Opaque Type은 실제로 어떤 타입일까요? Generic과 비슷한 개념이니만큼, Generic과 마찬가지 룰이 적용됩니다. 즉, 같은 인자(Generic에서는 타입 매게변수, Opaque Type에서는 반환값의 타입)로 호출 된 경우에는 항상 같은 타입을 가집니다. 

    ```swift
    func foo<T: Equatable>(x: T, y: T) -> some Equatable {
    let condition = x == y // OK, x와 y는 같은 타입
    return condition ? 1738 : 679
    }

    let x = foo("apples", "bananas")
    let y = foo("apples", "some fruit nobody's ever heard of")

    print(x == y) // OK, x와 y는 같은 Opaque Type  
    ```  

    만약 Opaque Type이 연관 타입(associated type)을 노출시키고 있다면, 이 연관 타입에 대한 정보도 유지됩니다. 이는 Collection과 같은 API를 자유롭게 사용할 수 있게 해줍니다.  

    ```swift
    // 두가지 이상의 프로토콜을 다음과 같이 엮을 수 있습니다.
    func makeMeACollection<T>(with: T) -> some RangeReplaceableCollection & MutableCollection { ... } 

    var c = makeMeACollection(with: 17)
    c.append(c.first!) // OK, RangeReplaceableCollection의 인터페이스입니다.
    c[c.startIndex] = c.first! // OK, MutableCollection의 인터페이스입니다.
    print(c.reversed()) // OK: 모든 Collection/Sequence 타입의 인터페이스 입니다.

    func foo<C: Collection>(_ : C) { }
    foo(c) // OK, C는 makeMeACollection<Int>의 Opaque Type으로 추론됩니다.  
    ```  

    게다가, Opaque Type은 다른 타입과 합쳐저도, Opaque Type 으로의 정체성을 유지합니다.  
    ```swift
    var cc = [c] 
    cc.append(c) // OK, cc의 원소 타입 == makeMeACollection<Int> 의 결과 타입
    var c2 = makeMeACollection(with: 38)
    cc.append(c2) //  OK, cc의 원소 타입 == makeMeACollection<Int> 의 결과 타입
    ```  
    하지만, 만약 함수가 Generic을 사용할 경우, 타입 매개변수가 다르면 아무리 실제 타입이 같아도 다른 타입으로 취급합니다.  

    ```swift
    var d = makeMeACollection(with: "seventeen")
    c = d // Error!
    ```  

    또한, 정적 타입 시스템에서 Opaque Type은 실제 구체타입과 같은 것으로 취급되지 않습니다. 

    ```swift  
    func foo() -> some BinaryInteger { return 219 }
    var x = foo() // Int의 Opaque Type
    let i = 912 // Int
    x = i // Error!
    ```  
    하지만 dynamic casting을 이용하면 Opaque Type의 실제 타입을 검사할 수 있습니다.  

    ```swift
    if let x = foo() as? Int {
    print("It's an Int, \(x)\n")
    } else {
    print("Guessed wrong")
    }

    //It's an Int, 219
    ```  

* Opaque Type의 실제 사용  

    1. 함수, 메소드 : 반환 값에 Opaque Type을 적용할 수 있습니다. 다만 다음과 같은 조건을 만족해야 합니다.  
       
       * return문에서 반환하는 모든 값의 타입은 동일해야 합니다. 또한 Opaque Type의 모든 제한을 충족해야 합니다.  다만 동일한 조건의 Generic을 적용하는 것은 가능합니다.  
    
    2. 프로퍼티, 첨자
        * 계산 프로퍼티, 첨자 : getter의 반환 값이 Opaque Type의 실제 타입이 됩니다.
        * 저장 프로퍼티 : 구체 타입을 반환하는 생성자로 초기화를 반드시 시켜야만 사용 가능합니다.  
        
        > 프로퍼티와 첨자가 다루는 타입이 getter에서 결정되기 때문에, setter의 인자 타입은 getter의 구현을 확인해야만 확실하게 알 수 있습니다.

    3. 타입 인터페이스 : 기본적으로 Opaque Type에 직접 이름을 주는 방법은 없습니다. 하지만 프로토콜의 연관 타입 기능을 이용하면 간접적으로 이런 일이 가능합니다.  
    
    ```swift
    protocol GameObject {
    associatedtype ObjectShape: Shape

    var shape: ObjectShape { get }
    }

    struct Player: GameObject {
    var shape: some Shape { /* ... */ } // ObjectShape 가 Shape의 Opaque Type의 typealias가 됩니다.

    }

    let pos: Player.ObjectShape // OK
    pos = Player().shape // OK
    ```
    ---  

    이상으로 Opaque Type에 알아보았습니다. 아직 정식 버젼에는 없는 새로나온 기능이다보니, 이해하기 힘든 면도 있었습니다.