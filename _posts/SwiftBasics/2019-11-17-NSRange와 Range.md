---
layout: post
title: NSRange와 Range
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

이번 포스트에서는 NSRange와 Swift에서 많이 쓰이는 Range에 대해서 알아보도록 하겠습니다. 

* **NSRange**  
  NSRange는 Foundation에서 제공하는 구조체 타입입니다. 
  
  ```swift
  public struct _NSRange {
    public var location: Int
    public var length: Int
  }
  ```

  이는 연속적인 개념을 가지는 객체(String에서 character들, Array에서의 원소들. 이들은 개념상 연속적으로 이어져 있습니다)들에서 일정 부분을 표현하기 위해 고안된 것으로, 시작점인 location, Range의 길이인 length로 이루어져 있고 둘 다 Int 타입입니다. NSRange가 나타내는 범위는 [location, location + length) 입니다.  

   NSRange는 Range가 유효하지 않음을 나타내기 위해서, [NSNotFound](https://developer.apple.com/documentation/foundation/nsnotfound)라는 특수한 값을 사용합니다. location이 이 값으로 설정되어 있다면 해당 Range는 유효하지 않습니다. 이러한 특수한 값을 사용하는 이유는 Objective-C에는 Optional이 없기 때문입니다.

  NSRange는 Foundation에서 문자열과, 문자열 관련 타입을 사용할 때 빈번하게 등장합니다.(NSAttributedString, NSRegularExpression) 이를 보면 처음부터 다른 Foundation의 특정요소들만을 위해 정의된 것이라 추측할 수 있습니다. 또한 관련 메소드들([NSIntersectionRange](https://developer.apple.com/documentation/foundation/1413065-nsintersectionrange), [NSUnionRange](https://developer.apple.com/documentation/foundation/1412317-nsunionrange?language=objc))을 보면 집합(Set)으로써의 면모가 잘 드러납니다.

* **Range & ClosedRange**  
  Swift에서의 Range는 표준 라이브러리이기 때문에 아무것도 import하지 않아도 사용할 수 있습니다.  

  ```swift
  public struct Range<Bound: Comparable> {
    public let lowerBound: Bound
    public let upperBound: Bound
  }

  public struct ClosedRange<Bound: Comparable> {
    public let lowerBound: Bound
    public let upperBound: Bound
  }

  public struct PartialRangeUpTo<Bound: Comparable> {
    public let upperBound: Bound
  }
  
  public struct PartialRangeThrough<Bound: Comparable> {  
     public let upperBound: Bound
  }

  public struct PartialRangeFrom<Bound: Comparable> {
    public let lowerBound: Bound
  }
  ```  
  
  [location, location+length) 로 표현되던 NSRange와 달리 [lowerBound, upperBound) 방식으로 표현됩니다. 또한 Generic으로 구현되어 다양한 타입의 범위를 표현할 수 있게 되었으며, 반개구간(half-open range)만 표현할 수 있었던 NSRange와 달리 폐구간을 표현하는 ClosedRange, 무한한 구간을 표현하기 위한 PartialRange 계열의 타입이 추가되었습니다. 이 경우 [lowerBound, upperBound]로 표현됩니다.  

  Range로 표현될 수 있는 타입은 Comparable을 구현하여 대소 관계가 명확한 타입이여야 합니다. 또한 위 정의에서 보다시피 Range는 단순히 상한선과 하한선만 정해져 있는 구조체 값이고, 기본적으로는 제한된 기능만 제공해줍니다. 다른 Range들의 구현도 비슷하니, 가장 기준이 되는 Range의 구현을 기준으로 살펴보겠습니다. 다음은 Range 전체 구현의 일부입니다.

  ```swift
  public struct Range<Bound: Comparable> {
    public let lowerBound: Bound
    public let upperBound: Bound

    // 범위 검사를 하지 않는 초기화
    public init(uncheckedBounds bounds: (lower: Bound, upper: Bound)) {
        self.lowerBound = bounds.lower
        self.upperBound = bounds.upper
    }

    // 범위 내에 원하는 원소가 있는지 검사한다.
    public func contains(_ element: Bound) -> Bool {
        return lowerBound <= element && element < upperBound
    }

    // 범위가 비어있는지 검사한다.
    public var isEmpty: Bool {
        return lowerBound == upperBound
    }
  }

  extension Range {
      // 현재의 범위와 다른 범위의 교집합을 구한다
    public func clamped(to limits: Range) -> Range {
        let lower =         
        limits.lowerBound > self.lowerBound ? limits.lowerBound
            : limits.upperBound < self.lowerBound ? limits.upperBound
            : self.lowerBound
        let upper =
        limits.upperBound < self.upperBound ? limits.upperBound
            : limits.lowerBound > self.upperBound ? limits.lowerBound
            : self.upperBound
        return Range(uncheckedBounds: (lower: lower, upper: upper))
    }

    // 범위가 겹치는 지 확인한다.
    public func overlaps(_ other: Range<Bound>) -> Bool {

        let isDisjoint = other.upperBound <= self.lowerBound
        || self.upperBound <= other.lowerBound
        || self.isEmpty || other.isEmpty
        return !isDisjoint
    }

    public func overlaps(_ other: ClosedRange<Bound>) -> Bool {
        let isDisjoint = other.upperBound < self.lowerBound
        || self.upperBound <= other.lowerBound
        || self.isEmpty
        return !isDisjoint
    }
  }
  ```  
  
  만약 Bound가 Stridable 프로토콜을 채택하고, Stridable이 BinaryInterger 프로토콜을 채택한다면(쉽게 말하면 Int, UInt 계열의 타입입니다.) Range는 Collection을 채택하게 되고, Collection 프로토콜의 메소드들을 모두 사용할 수 있게 됩니다. 

  ```swift
  var range = Range<Int>(uncheckedBounds: (0,10))

  print(range) // 0..<10

  range.popLast() // 9

  print(range) // 0..<9

  ```  
  이를 나타내기 위한 별도의 typealias 또한 존재하는데, 다음과 같습니다. 

  ```swift  
  public typealias CountableRange<Bound: Strideable> = Range<Bound>
    where Bound.Stride: SignedInteger

  public typealias CountablePartialRangeFrom<Bound: Strideable> = PartialRangeFrom<Bound>
    where Bound.Stride: SignedInteger  

  public typealias CountableClosedRange<Bound: Strideable> = ClosedRange<Bound>
    where Bound.Stride: SignedInteger
  ```
  
* **Range Expression**  
  Swift에서는 Range객체를 직접 만들기 보다는, Range표현식을 통해서 만드는 것을 권장합니다. 이는 Comparable의 extension으로 제공되며, 연산자로 사용할 수 있습니다. 이렇게 만드는 것을 권장하는 이유는, 범위가 유효한지 검사하는 로직이 들어가 있기 때문입니다.  

  ```swift
  extension Comparable {
      // minimum ..< maximum
    public static func ..< (minimum: Self, maximum: Self) -> Range<Self> {
        _precondition(minimum <= maximum, "Can't form Range with upperBound < lowerBound")
        return Range(uncheckedBounds: (lower: minimum, upper: maximum))
    }

    // ..< maximum
    public static prefix func ..< (maximum: Self) -> PartialRangeUpTo<Self> {
        return PartialRangeUpTo(maximum)
    }

    // ... maximum
    public static prefix func ... (maximum: Self) -> PartialRangeThrough<Self> {
        return PartialRangeThrough(maximum)
    }

    // minimum ...
    public static postfix func ... (minimum: Self) -> PartialRangeFrom<Self> {
        return PartialRangeFrom(minimum)
    }

    // minimum ... maximum
    public static func ... (minimum: Self, maximum: Self) -> ClosedRange<Self> {
    _precondition( minimum <= maximum, "Can't form Range with upperBound < lowerBound")
        return ClosedRange(uncheckedBounds: (lower: minimum, upper: maximum))
    }
  ```  
  이러한 식들은 주로 for문이나 배열을 쪼갤 때 사용합니다. 

* **NSRange와 Range간의 변환**  
   Swift의 네이티브 타입 중 일부는 NS가 붙은 Foundation 타입으로의 브릿징을 지원합니다. 하지만 NSRange와 Range 간에는 브릿징이 불가능합니다. 

   1. NSRange와 Range의 프로퍼티는 의미상으로 동일하지 않습니다. 
   
   2. Range는 Generic이 적용되어 있지만, NSRange는 아닙니다.

  대신 서로 해당 타입을 인자로 받는 생성자를 제공합니다. 다음은 그 중 일부입니다.  

  ```swift  
  extension NSRange {
    public init(_ x: Range<Int>) {
        location = x.lowerBound
        length = x.count
    }
  }  

  extension Range where Bound == Int {
    // NSRange가 유효하지 않을 수 있기 때문에 failable 생성자로 제공된다.
    public init?(_ range: NSRange) {
        guard range.location != NSNotFound else { return nil }
        self.init(uncheckedBounds: (range.lowerBound, range.upperBound))
    }
  }
  ```  
* **결론**  
  1. Swift를 쓴다면 Range를 쓰면 됩니다.
  2. Foundation에서 NSRange를 요구한다면 어쩔 수 없이 써야겠지만, Range를 NSRange로 바꿔서 사용하면 됩니다.

---  

> 참고자료
> [swift 공식 저장소](https://github.com/apple/swift)
> [Foundation 공식 저장소](https://github.com/apple/swift-corelibs-foundation)