---
layout: post
title: Equatable, Comparable, Hashable
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

swift에서는 사용자 지정 타입에서도 라이브러리에서 제공하는 여러가지 기능을 사용할 수 있게 만들 수 있습니다. 이러한 기능을 사용하기 위해서는 사용자 지정 타입이 특정 메세지를 처리할 수 있는 지 여부를 알아야 하는데, Swift는 컴파일 타임에 이를 파악할 수 있도록 이를 프로토콜로 만들어 제공합니다. 프로그래머는 이렇게 제공되는 프로토콜을 사용자 지정 타입에서 채택하고 구현하기만 하면 됩니다. 이번 포스트에서는 이 중 가장 대표적인 프로토콜인 Equatable, Comparable, Hashable에 대해서 알아보겠습니다.  

* **Equatable**  
  Equatable은 동일성(Equality)을 비교할 수 있도록 하는 프로토콜입니다. 이를 채택한 타입은 ==, != 연산을 사용할 수 있습니다. 또한 Sequence나 Collection에서 Contains 등의 비교를 사용하는 기능등을 추가로 사용할 수 있습니다. 또한 이번 포스트에서 다룰 Comparable, Hashable의 기반 프로토콜이기도 합니다.  

  Equatable 프로토콜을 채택하기로 했다면, [==](https://developer.apple.com/documentation/swift/equatable/1539854) 연산을 구현해야 합니다. Equatable의 나머지 연산들은 모두 ==연산을 기반으로 기본 구현체를 제공해 주기 때문에, == 연산의 구현만 하면 됩니다. 또한 다음 조건을 만족할 경우, 컴파일러가 == 연산의 구현을 자동적으로 해 줄 수 있습니다.  

  1. 구조체 타입의 경우, 모든 저장 프로퍼티가 Equatable을 채택해야 합니다. Swift에서 기본적으로 제공해주는 타입들은 대부분 이 조건을 만족합니다. 
  
  2. 열거형 타입의 경우, 모든 연관 값들이 Equatable을 채택해야 합니다. 연관값이 없는 경우에는 Equatable을 명시적으로 채택하지 않아도 자동으로 컴파일러가 채택해줍니다.  

  위 조건에 맞지 않는 경우에만 == 연산을 구현해 주면 됩니다. 아래 예제 코드는 클래스이기 때문에 == 연산을 수동으로 구현하는 것을 보여줍니다.

  ```swift
  class StreetAddress {
    let number: String
    let street: String
    let unit: String?

    init(_ number: String, _ street: String, unit: String? = nil) {
        self.number = number
        self.street = street
        self.unit = unit
    }
  }  

  extension StreetAddress: Equatable {
    static func == (lhs: StreetAddress, rhs: StreetAddress) -> Bool {
        return
            lhs.number == rhs.number &&
            lhs.street == rhs.street &&
            lhs.unit == rhs.unit
    }
  }
  ```  

  동일성을 가진 두 인스턴스는, 해당 인스턴스의 값을 사용하는 코드에서 상호 대체가 가능합니다. 또한 == 연산은 다음과 같은 특성을 만족해야 합니다.  

  1. a == a 는 항상 참이여야 한다.(Reflexivity)  
  
  2. a == b 와 b == a의 결과가 동일해야 한다. (Symmetry)  
  
  3. a == b, b == c이면 a == c 여야 한다. (Transitivity)  
  
  > Equality와 identity(정체성)는 구별되어야 합니다. identity는 클래스 타입에서 인스턴스의 id(ObjectIdentifier)가 동일한지 여부를 의미합니다. 두 개의 서로 다른 인스턴스가 id외에 모든 프로퍼티 값이 같다면 값이 같다(Equal)고는 할 수 있지만, 두 인스턴스의 id는 다르므로 정체성은 같지 않습니다. 
  > 정체성 비교는 === 연산자를 이용해 수행할 수 있습니다.  

* **Comparable**  
  Comparable은 타입이 표현하는 값이 내재적인 순서를 가지고 있을 때 사용할 수 있는 프로토콜입니다. 이 프로토콜을 채택하면 부등호(>, >=, <, <=)로 비교 연산이 가능해지며, sequence나 collection등에서 정렬 등의 추가 기능을 사용할 수 있게 됩니다.  
  Comparable 프로토콜은 Equatable을 상속받기 때문에 == 연산 구현을 해줘야 하고, 추가적으로 [<(less)](https://developer.apple.com/documentation/swift/comparable/1538311) 연산을 구현해줘야 합니다. 이렇게만 해주면 나머지 부등호 연산들은 ==와 < 을 이용해서 구현할 수 있기 때문에 프로토콜 차원에서 기본 구현체를 제공합니다. Comparable은 컴파일러가 자동으로 구현을 제공할 수 없기 때문에 필요하면 프로그래머가 반드시 구현을 해줘야 합니다. < 연산은 다음 조건을 만족해야 합니다.  

  1. a < a 는 항상 거짓이여야 한다.(Irreflexivity)  
  
  2. a < b 면 !(b < a) 이다.(Asymmetry)  
  
  3. a < b, b < c 면 a < c 이다.(Transitivity)  

  > 구현하려는 타입에서 일반적이지 않은 특별한 값을 제공하는 경우가 있습니다. (부동소수점 타입의 NaN(Not a Number) 등) 이러한 값들은 일반적인 값들의 순서에 포함되지 않아도 됩니다. 즉 이런 특별한 값은 다른 일반적인 값보다 작지도, 크지도, 같지도 않습니다.  

* **Hashable**  
   [Hasher](https://developer.apple.com/documentation/swift/hasher)를 통해서 Int 타입의 해쉬 값을 만들어 낼 수 있도록 만들어주는 프로토콜입니다. 이 프로토콜을 적용한 타입은 Set과 Dictionary에서 키로 사용할 수 있게 됩니다. 
   Hashable을 채택하기 위해서는 Hashable의 부모 프로토콜인 Equatable을 구현함과 동시에  [hash()](https://developer.apple.com/documentation/swift/hashable/2995575-hash) 함수를 구현해야 합니다. 이 함수는 Hasher를 인자로 받아서 Hasher에 [combine()](https://developer.apple.com/documentation/swift/hasher/2995578-combine) 메소드를 통해 값을 제공해주도록 구현됩니다. 이 때 해쉬 값이 같게 나오기 위해서는 같은 타입의 값을 같은 순서로 넣어주어야 합니다.   
   Hashable도 컴파일러가 기본 구현을 제공해 줄 수 있는데, 그 조건은 Equatable과 거의 동일합니다. 즉, Hashable을 채택한 타입을 저장 프로퍼티나 연관 값으로 제공해주면 그 타입은 자동으로 Hashable을 채택하게 됩니다.  

   > Hasher의 시드 값은 프로그램이 실행될 때 마다 변화합니다. 또한 해쉬 알고리즘도 두가지 버젼이 있고, 이것도 해쉬를 할 때 마다 바뀔 수 있으므로 해쉬값을 따로 저장해 놓는 것은 소용이 없습니다.  