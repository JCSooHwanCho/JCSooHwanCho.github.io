---
layout: post
title: enum 더 살펴보기 - CaseIterable, RawPresentable, 메모리 구조
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

[지난 포스트](/2019-08-18-enum-살펴보기-사용법/)에서는 enum의 기능과 사용법에 대해 알아보았습니다. 이번에는 enum에 대해 더 깊게 알아보도록 하겠습니다.

> 이 포스트는 공식문서들과 다음 글들을 참조하였음을 밝힙니다.  
> [CaseIterable](https://medium.com/@09mejohn/swift-4-2-enumerable-enum-748a2dca9e41)  
> [swift 타입의 메모리 구조](https://github.com/apple/swift/blob/master/docs/ABI/TypeLayout.rst)  
> [Can associated values and raw values coexist in Swift enumeration?](https://stackoverflow.com/questions/24171814/can-associated-values-and-raw-values-coexist-in-swift-enumeration/28927425)  
> [Old Begemann - The RawRepresentable Protocol in Swift](https://oleb.net/blog/2016/11/rawrepresentable/)

* CaseIterable  
    enum의 모든 경우를 순회하고 싶을 경우, CaseIterable 프로토콜을 적용하는 것을 알아보았습니다. 하지만 만약에 associated value를 가지는 case가 한 개라도 있을 경우, 오류가 발생하게 됩니다. 어째서일까요?

    CaseIterable의 선언을 보면 다음과 같습니다. (주석은 제거했습니다)

    ```swift  
    public protocol CaseIterable {
        associatedtype AllCases : Collection where Self == Self.AllCases.Element

        static var allCases: Self.AllCases { get }
    }
    ```  

    AllCases 타입은 반드시 제공되어야 하지만, allCases 프로퍼티의 타입으로 추론이 가능하므로 실질적으로는 allCases 계산 프로퍼티만 제공하면 됩니다. 하지만 우리는 CaseIterable을 적용하면서 저런 변수를 실제로 구현하지 않았는데도 사용에는 전혀 문제가 없었습니다. 이는 컴파일러가 자동으로 allCases 프로퍼티를 만들어 제공해주었기 때문입니다. 

    하지만 Associated Value가 하나라도 있다면, 컴파일러는 이 allCases 프로퍼티를 자동으로 만들어 줄 수 없습니다. Associated Value의 값이 다른 경우를 같은 case로 취급해야 하는지, 다른 case로 취급해야하는지 컴파일러가 판단할 수 없기 때문입니다. 따라서 이 경우는 직접 allCases 프로퍼티를 제공해줘야 합니다.
    
    > allCases의 타입은 프로토콜의 선언을 따라 enum 자신을 원소로 가지는 Collection 타입으로 제한됩니다.
     
    ```swift
    enum SomeEnum: CaseIterable {
    
        case one(Int)
        case two
    
        static var allCases: [someEnum] {
            return [.one(0), .two]
        }
    }
    ```

---  

* RawRepresentable  
   Raw Value가 될 수 있는 값은 String, Character, 정수/ 실수 타입 이라고 했었는데, 정확히는 **'숫자/문자 리터럴로 초기화 할 수 있는 타입'**이 enum의 raw value가 될 수 있습니다. 
   어떤 타입을 리터럴로 초기화하기 위해서는 [ExpressibleBy~ 가 붙은 프로토콜](https://developer.apple.com/documentation/swift/swift_standard_library/initialization_with_literals) 를 최소 하나 이상 채택해야 하는데, enum이 지원하는 프로토콜은 다음과 같습니다.
    * 숫자 리터럴
      * [ExpressibleByIntegerLiteral](https://developer.apple.com/documentation/swift/expressiblebyintegerliteral) : 정수 리터럴로 초기화 가능한 타입을 나타냅니다. 
      * [ExpressibleByFloatLiteral](https://developer.apple.com/documentation/swift/expressiblebyfloatliteral) : 실수 리터럴로 초기화 가능한 타입을 나타냅니다.  
    * 문자 리터럴
      * Character
        * [ExpressibleByUnicodeScalarLiteral](https://developer.apple.com/documentation/swift/expressiblebyunicodescalarliteral) : 유니코드 하나로 나타낼 수 있는 글자들로 초기화 가능한 타입을 나타냅니다.
        * [ExpressibleByExtendedGraphemeClusterLiteral](https://developer.apple.com/documentation/swift/expressiblebyextendedgraphemeclusterliteral) : 유니코드 여러개로 나타낼 수 있는 글자들로 초기화 가능한 타입을 나타냅니다.
      * String
        * [ExpressibleByStringLiteral](https://developer.apple.com/documentation/swift/expressiblebystringliteral) : 문자열 리터럴로 초기화 가능한 타입을 나타냅니다.  

    위 프로토콜을 구현한 뒤 하나 더 적용해야 할 프로토콜이 있는데, 바로 [Hashable](https://developer.apple.com/documentation/swift/hashable) 입니다. 

    > 원래의 원칙은 Hashable 을 구현하는 것입니다만, [Equatable](https://developer.apple.com/documentation/swift/equatable) 만 구현해도 동작은 합니다.

    위 내용을 따라 예제를 구현해보면 다음과 같습니다. 숫자를 입력 받아 10으로 나눈 몫과 나머지를 저장하는 클래스입니다.

    ```swift
    struct SomeStruct: Hashable, ExpressibleByIntegerLiteral {
        typealias IntegerLiteralType = Int

        static func == (lhs: SomeStruct, rhs: SomeStruct) -> Bool {
            return lhs.a == lhs.a
        }
        
        let a: Int
        let b: Int
        
        init(integerLiteral value: IntegerLiteralType) {
            a = value / 10
            b = value % 10
        }

        func hash(into hasher: inout Hasher) {
            hasher.combine(a);
            hasher.combine(b);
        }
    }

    enum SomeEnum: SomeStruct {
        
        case one // rawValue = SomeStruct(a: 0, b: 0)
        case two // rawValue = SomeStruct(a: 0, b: 1)
    }
    ```  

    > ExpressibleByStringLiteral, ExpressibleByIntegerLiteral을 구현한 클래스는 컴파일러가 암시적으로 값을 줄 수 있습니다. 하지만 Float,Character 계열의 Expressible 프로토콜을 채택한 경우에는 컴파일러가 암시적으로 값을 넣어주지 않기 때문에 반드시 명시적으로 값을 제공해야 합니다. 
    > 다만 실수 타입을 사용한 enum은 초기값을 안줘도 잘 동작하는데, 이는 실수 타입이 ExpressibleByIntegerLiteral을 채택하고 있기 때문입니다. 또 이 프로토콜 덕에 정수 리터럴로 실수 타입이 초기화가 가능한 것이기도 합니다.

    아직 끝이 아닙니다. 우리는 enum에 rawValue 프로퍼티를 구현한 적이 없습니다. 그런데 실제 사용할때는 자연스럽게 사용하게 됩니다. 어떻게 된 걸까요?  

    답은 [RawRepresentable](https://developer.apple.com/documentation/swift/rawrepresentable/1538354-init)을 컴파일러가 자동으로 제공해주었기 때문입니다. RawRepresentable 프로토콜은 다음과 같이 정의되어 있습니다.  

    ```swift
    public protocol RawRepresentable {
    associatedtype RawValue

    init?(rawValue: Self.RawValue)

    var rawValue: Self.RawValue { get }
    }
    ```  

    RawRepresentable 프로토콜은 컴파일러가 자동으로 추가해주는 프로토콜이기 때문에 일반적으로는 추가할 필요가 없습니다. 하지만 RawRepresentable 프로토콜이 유용하게 사용할 수 있는 부분이 있는데, 바로 리터럴로 표현할 수 없는 타입을 rawValue로 사용할 때 입니다.

    컴파일러가 RawRepresentable 프로토콜을 추가해주기 위해서는, 모든 raw value값이 컴파일 타임에 결정되어야 합니다.(그리고 이것이 enum에서 리터럴을 강제했던 이유기도 합니다.) 하지만 RawRepresentable 프로토콜을 프로그래머가 직접 구현한다면, 이러한 제한을 해제할 수 있습니다. 다만 enum에 직접 구현하지 말고 extension으로 별도로 구현해줘야 합니다.

    ```swift
    enum SomeEnum {
    
    case odd
    case even
    }

    extension SomeEnum: RawRepresentable {
        typealias RawValue = Bool

        init?(rawValue: Bool) {
            if rawValue {
                self = .even
            } else {
                self = .odd
            }
        }
        
        var rawValue: Bool {
            switch self {
            case .even:
                    return true
            case .odd:
                    return false
            }
        }
    }
    ```  

    > swift에서는 한 타입에 대해서 같은 프로토콜을 두번 이상 구현하는 것을 금지하고 있습니다. 따라서 RawRepresentable을 별도로 구현할 것이라면, enum에는 rawValue 타입을 제공해서는 안됩니다.  

    별도로 구현하는 것에는 또 다른 장점이 있는데, 바로 raw value와 associated value를 같이 쓸 수 있다는 것입니다. 예를 들어 다음과 같은 코드는 오류를 발생시킵니다.

    ```swift
    enum SomeEnum: Int { //'SomeEnum' declares raw type 'Int',
    //but does not conform to RawRepresentable and conformance could not be synthesized
    
        case odd(Int) // Enum with raw type cannot have cases with arguments
        case even
    }
    ```  

    하지만 별도로 구현하면 오류 없이 실행이 가능합니다.

    ```swift
    enum SomeEnum {
    
    case odd(Int)
    case even(Int)
    }

    extension SomeEnum: RawRepresentable {
        typealias RawValue = Bool

        init?(rawValue: Bool) {
            if rawValue {
                self = .even(0)
            } else {
                self = .odd(0)
            }
        }
        
        var rawValue: Bool {
            switch self {
            case .even(let num):
                    return num%2 == 0
            case .odd(let num):
                    return num%2 == 1
            }
        }
    }

    print(SomeEnum.even(10).rawValue) // true
    print(SomeEnum.odd(10).rawValue) // false
    ```  
    > rawValue의 구현을 보면 눈치 챌 수도 있지만, rawValue는 계산 프로퍼티이기 때문에, enum타입의 인스턴스 크기에 영향을 미치지 않습니다.

--- 

* enum의 메모리 구조  
   raw value, associated value등이 사용될 때의 enum 타입은 과연 어느 정도의 크기를 가질까요? 런타임에서 enum 타입을 저장하는 전략은 다음과 같이 나눌 수 있습니다.

   > 여기서는 간략히만 설명합니다. 더 자세히 알고 싶으신 분은 [swift 타입의 메모리 구조](https://github.com/apple/swift/blob/master/docs/ABI/TypeLayout.rst) 문서를 참고해주세요.
    
    1. 아무런 case를 가지지 않는 경우 : 내부적으로 Empty Type(실제 사용할 수 있는 타입은 아닙니다.)으로 지정되고, 0의 크기를 가집니다.  
      
    2. case가 1개인 경우  
       1. associated Value가 없는 경우 : Empty Type이 되고, 0의 크기를 가집니다.  
       
       2. associated Value가 있는 경우 : associated value의 메모리 구조와 완전히 동일하게 됩니다.  
   
    3. case가 여러개인 경우
       1. associated Value가 없는 경우(C-Like Enums) : 선언된 순서대로 태그에 번호를 매기고, 해당하는 태그를 모두 포함할 수 있는 최소한의 비트만 사용합니다. 실제로 인스턴스 크기는 바이트 단위로 표현되기 때문에 최소 1바이트는 가지게 됩니다. 
       
       2. associated Value가 있는 경우 : 태그를 나타내기 위한 공간의 크기 + associated value의 크기가 가장 큰 case의 associated value 크기로 결정됩니다. 이때 associated value의 크기는 선언된 타입들의 크기의 합입니다. case가 많을 경우 태그를 나타내기 위한 공간의 크기가 증가할 수도 있습니다.  
    
    > swift는 최적화를 위해 태그명을 저장하지 않습니다.  
     
---  

enum에 대해서 많은 것을 알아보았습니다. 원래 그런가 보다 싶었던 것들도 파고보면 상당히 복잡하게 이루어져 있음을 다시 한번 느낍니다.