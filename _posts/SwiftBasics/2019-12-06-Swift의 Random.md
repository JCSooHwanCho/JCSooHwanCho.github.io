---
layout: post
title: Swift의 Random
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  
이번 포스트에서는 Swift로 임의의 수를 만들어 내는 방법에 대해서 알아보도록 하겠습니다.  

* **swift 4.2 이전**  
  Swift이전에는 다음과 같은 함수들을 사용했습니다.  

  ```swift
  // arc4 랜덤 알고리즘을 사용해서, 0...2^32-1 사이의 정수 값을 반환한다.
  public func arc4random() -> UInt32 
  // 주어진 buffer를 랜덤 값으로 채운다.
  public func arc4random_buf(_ __buf: UnsafeMutableRawPointer!, _ __nbytes: Int) 
  // 랜덤 알고리즘의 Seed를 바꿔준다.
  public func arc4random_stir()
  // arc4random과 같으나, 확률분포가 균일하다. 
  public func arc4random_uniform(_ __upper_bound: UInt32) -> UInt32
  // 48비트의 seed를 이용한 선형 합동 알고리즘을 사용해서 0...1 사이의 실수값을 반환한다.
  public func drand48() -> Double // Seed를 시스템에서 제공해주는 버전
  public func erand48(_: UnsafeMutablePointer<UInt16>!) -> Double // Seed를 직접 넣어주는 버전
  ``` 

  > arc4 알고리즘과 선형 합동 알고리즘에 대한 자세한 내용은 위키피디아 등을 참조하시가 바랍니다  
  > [arc4](https://en.wikipedia.org/wiki/RC4)  
  > [선형 합동 생성기](https://en.wikipedia.org/wiki/Linear_congruential_generator)  

  네이밍 컨벤션이 이질적인데, 이는 이들이 Swift 자체 함수가 아니라, 커널에서 제공해주는 C함수를 브릿징한 것이기 때문입니다. 특히 arc4계열 함수는 반환값이 UInt32이기 때문에 Swift에서 일반적으로 쓰이는 Int나 UInt등으로 바꿔주는 과정이 필요할 수도 있습니다..  

  ```swift
  let n = Int(arc4random_uniform(42))
  ```  

* **swift 4.2 이후의 random**  
  Swift4.2 이후에는 Swift의 모든 정수와 실수 타입은 자체적인 random함수를 제공해줍니다. 이때 범위를 Range 혹은 ClosedRange로 제공해줘야 하며, 해당 범위내의 랜덤 값을 제공받을 수 있습니다.

  ```swift
  let num = Int.random(in: 0...10) // 0~10 사이의 Int 값
  let num2 = Double.random(in: 0.0...2.0) // 0.0~2.0 사이의 Double 값
  ```  

  또한 4.2이전에는 배열의 인덱스를 통해서 간접적으로 할 수 밖에 없었던, 배열에서 임의의 원소를 뽑거나 배열을 섞는 함수도 자체적으로 제공해줍니다.  

  ```swift
  public func randomElement() -> Element? // 배열 내 임의의 원소를 반환한다.
  public func shuffled() -> [Element] // 현재의 배열을 임의로 섞은 복사본을 반환한다.
  public mutating func shuffle()  // 현재 배열을 임의로 섞는다.
  ```  
  이 함수들은 두번째 인자를 받는 버전이 별도로 존재하는데, 해당 인자의 타입은 RandomNumberGenerator입니다. 이는 프로토콜로 다음과 같이 정의되어 있습니다.

  ```swift  
  public protocol RandomNumberGenerator {
      mutating func next() -> UInt64 // 임의의 64비트 정수를 반환한다.
  }
  ```  

  이 프로토콜을 채택하면, 원하는 랜덤 알고리즘을 적용할 수 있게 됩니다. 별도의 알고리즘이 필요하지 않다면, 자체적으로 제공해주는 Generator를 사용하면 됩니다.  

  ```swift
  public struct SystemRandomNumberGenerator: RandomNumberGenerator {
    public init()
    public mutating func next() -> UInt64 
  }
  ```
---  
> 참고 자료  
> [How To: Random Numbers in Swift](https://learnappmaking.com/random-numbers-swift/)  
> [arc4random() man page](https://linux.die.net/man/3/arc4random)  
> [drand48 man page](https://linux.die.net/man/3/drand48)  