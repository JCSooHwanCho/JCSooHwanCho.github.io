---
layout: post
title: Swift String 효율적으로 쓰기
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

Swift의 문자열 타입인 String은 깊은 설계적 고민이 녹아있는 타입입니다. 내부는 굉장히 복잡하지만, 사용하는 입장에서는 내부 구현을 알지 못해도 잘 쓸 수 있습니다. 하지만 C, C++ 등의 문자열과는 그 특성이 다르기 때문에, 이를 숙지해야 효율적인 코드를 짤 수 있습니다. 이번 포스트에서는 이러한 테크닉들을 알아보도록 하겠습니다.  

* **Swift의 String은 왜 다를까?**  
  Swift의 문자열과 다른 언어의 문자열 타입의 가장 큰 차이는 Int 기반의 인덱스 참조가 가능한지 여부입니다. 다른 언어에서는 Int값을 통해서 원하는 글자를 마음대로 참조하고 변경할 수 있지만, Swift에서는 Int 참조가 불가능합니다. 왜 이런 차이가 발생할까요? 

  레퍼런스에서 말하는 String은 다음과 같습니다.  

  > A string is a series of characters, such as "Swift", that forms a collection. 

  즉 String은 Character의 Collection 입니다. 여기서 Collection은 Array라고 생각하면 됩니다. 다른 언어에서도 String은 Character 타입의 배열로 나타냅니다. 그런데 왜 Swift만 다를까요? 답은 Character 타입이 무엇을 가리키는 지에 있습니다.  

  C++나 Java의 char타입은 고정된 크기를 가집니다. 하지만 Swift의 Character는 1개 이상의 Unicode Scalar로 이루어져 있습니다. 즉 크기가 가변적이라는 것입니다. 실제로 String은 하나의 값에 다양한 뷰를 제공합니다.  

  ```swift
  let str = "🇵🇷" // 국기 이모지는 Unicode Scalar 두개로 이루어져 있습니다.

  print(str.count)
  for c in str {
      print(type(of: c),c)
  }

  print("--------------------")

  print(str.unicodeScalars.count)
  for c in str.unicodeScalars {
      print(type(of: c),c)
  }

  print("--------------------")

  print(str.utf16.count)
  for c in str.utf16 {
      print(type(of: c),c)
  }

  print("--------------------")
  print(str.utf8.count)
  for c in str.utf8 {
      print(type(of: c),c)
  }

  // 1
  // Character 🇵🇷
  // --------------------
  // 2
  // Scalar 🇵
  // Scalar 🇷
  //  --------------------
  // 4
  // UInt16 55356
  // UInt16 56821
  // UInt16 55356
  // UInt16 56823
  // --------------------
  // 8
  // UInt8 240
  // UInt8 159
  // UInt8 135
  // UInt8 181
  // UInt8 240
  // UInt8 159
  // UInt8 135
  // UInt8 183
  ```  

  이렇게 다양한 뷰를 제공하기 때문에, 단순히 Int 타입으로 글자들을 참조하기에는 어떠한 뷰를 기준으로 인덱싱을 지원해야 하는지에 대해서 애매한 부분이 존재합니다. 이 때문에 String은 별도로 설계된 [String.Index](https://developer.apple.com/documentation/swift/string/index)라는 구조체를 별도로 제공하여 인덱싱을 지원합니다. 각 View는 별도의 Index 구조체를 제공하지만, 구조 자체는 동일하여 서로 비교하거나, 다른 뷰의 인덱스로써 사용할 수 있는 호환성을 제공하기도 합니다.
  
  ```swift
  let str = "🇵🇷🇵🇷"

  let index = str.index(str.startIndex,offsetBy: 1)

  let i = str.utf8.index(str.utf8.startIndex, offsetBy: 3)

  print(str.utf16[i...])

  //UTF16View(_slice: Swift.Slice<Swift.String.UTF16View>(_startIndex: Swift.String.Index(_rawBits: 196608), _endIndex: Swift.String.Index(_rawBits: 1048577), _base: StringUTF16("🇵🇷🇵🇷")))
  ```  

* **String 효율적으로 사용하기**  
  윗 문단과 같은 기능을 제공하기 위해 Swift의 String은 다른 언어에서 자연스럽게 제공하는 기능을 포기했습니다. 바로 Random Access 기능입니다. 이는 Collection 내에서 임의의 거리만큼 떨어진 원소를 O(1)의 시간복잡도로 참조할 수 있는 능력입니다. Swift에서는 [RandomAccessCollection](https://developer.apple.com/documentation/swift/randomaccesscollection) 프로토콜로 이러한 기능을 제공합니다. 이에 대비되는 것이 [BidirectionalCollection](https://developer.apple.com/documentation/swift/bidirectionalcollection) 인데, 이는 무조건 자기의 앞 뒤의 원소만을 참조할 수 있습니다. 따라서 임의의 거리의 원소를 참조하기 위해서는 그 이전 단계의 원소를 모두 차례대로 거쳐야만 합니다.
  
  Swift의 String은 BidirectionalCollection을 채택했지만, RandomAccess
  Collection은 채택하고 있지 않습니다. 이로 인해 일부 연산에서 우리의 직관에 위배되는 동작을 수행하게 됩니다.  

  1. String의 길이를 구하는 연산은 O(n)의 시간이 소요됩니다. 만약 RandomAccessCollection이였다면 O(1)의 시간복잡도를 가집니다.  
  
  2. String.Index 타입은 + 연산, - 연산 등을 지원하지 않습니다. 따라서 현재 인덱스에서 n만큼 떨어진 글자를 참조하기 위해서는 O(n)의 시간복잡도가 필요합니다. RandomAccessCollection이였다면 O(1)만에 가능합니다.  

  특히 Swift로 PS(Problem Solving, 문제 풀이)를 한다면 이러한 부분에서 TLE(Time Limit Exceed, 시간 초과)가 발생할 우려가 있으니 반드시 주의하셔야만 합니다.  

  따라서 Swift로 문자열을 다룰 때면 다음과 같은 테크닉을 활용해 보시기를 추천드립니다.  

  1. 문자열의 길이를 자주 참조한다면, 처음 한번만 구해 별도의 변수로 저장해 놓습니다. 문자열의 길이가 변하더라도 이를 측정할 수 있다면 매번 count를 구하는 것보다는 별도 변수로 직접 카운팅 하는 것이 좋습니다.  
  
  2. Index를 여러 번 구해야 할때는, 이전에 구해놓은 인덱스를 재활용해서 연산 수를 줄이는 것이 좋습니다. 무조건 첫 인덱스부터 구하면 불필요한 연산이 많아집니다.  
  
  3. 만약 String의 글자들에 대한 임의 접근이 반드시 필요하다면, String을 Character의 Array로 바꾸고 작업을 한 뒤 마지막에 String으로 바꿔주는 것도 좋습니다. 또한 문자열을 빈번하게 순회해야 한다면, String.Index보다는 Character Array가 훨씬 효율적입니다.

  ```swift
  let str = "Hello, World!"

  let arr = Array(str)

  print(arr) 

  // ["H", "e", "l", "l", "o", ",", " ", "W", "o", "r", "l", "d", "!"]
  ```  
---   

> 참고자료  
> [Apple Documentation - String](https://developer.apple.com/documentation/swift/string)  
> [Apple Documentation - Character](https://developer.apple.com/documentation/swift/character)  
> [Apple Documentation - BidirectionalCollection](https://developer.apple.com/documentation/swift/bidirectionalcollection)  
> [Apple Documentation - RandomAccessCollection](https://developer.apple.com/documentation/swift/randomaccesscollection)