---
layout: post
title: Monad와 Swift
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

Swift는 함수형 패러다임을 적용한 언어입니다. 이 과정에서 함수형 언어의 영향을 많이 받았는데, 그중에서도 하스켈의 영향을 많이 받았습니다. 하스켈이란 언어에서 가장 인상적인 개념을 하나 꼽으라고 한다면 단연 모나드(Monad)를 꼽을 수 있습니다. 이번 포스트에서는 하스켈의 모나드 개념을 최대한 간략히 살펴보고, Swift에서 이것이 어떻게 활용되는 지 알아보도록 하겠습니다.  


* **모나드로의 여정**  
  하스켈이라는 언어를 조금이라도 본 분들이라면 Monad라는 이름과 그 난해함을 들어보셨을 것입니다. 이를 이해하기 위해서 첫 단계부터 차근 차근 나가보겠습니다.  

  첫 단계로 알아볼 것은 functor입니다. functor는 하스켈에서 다음과 같이 정의됩니다. 

  ```haskell
  class Functor f where  
    fmap :: (a -> b) -> f a -> f b
  ```  

  여기서 functor는 구체 타입이 아니라 타입 생성자입니다. Swift로 치면 Generic이라 볼 수 있겠죠. 위 구현을 풀어쓰면, (a->b) 타입의 함수와 functor f로 감싸진 a타입의 변수를 입력 받아 functor f로 감싸진 b를 반환하는 함수입니다. swift로 이를 바꿔보면 다음과 같습니다.  

  ```swift
   class Functor<T> {
      func fmap<U>(_ transform: T -> U) -> Functor<U> 
  }
  ```  

  어디서 많이 보던 패턴이라면, 이해가 빠르신 것입니다. 이는 바로 swift의 map에 해당하는 것입니다. 즉, functor는 간단하게 생각하면 **map을 적용할 수 있는 타입**입니다. functor를 값을 담을 수 있는 상자라고 생각하면, 그 상자를 풀지 않아도 map을 통해 영향을 줄 수 있게 됩니다. 이 상자는 값에 대한 추가적인 상태를 담는 그릇으로, 컨텍스트(Context)라고 부릅니다. 

  함수형 언어에서는 함수 역시 일급 객체로 취급됩니다. 따라서 functor안에는 함수도 들어갈 수 있습니다. 이렇게 함수를 가진 functor를 다른 값을 가진 functor에 적용하려면 어떻게 할까요? 이를 가능하게 해주는 것이 applicative functor입니다. 
  applicative functor는 다음과 같이 정의됩니다. 이를 이용하면 일반 함수를 컨텍스트를 가진 값에 적용할 수 있게 됩니다.

  ```haskell
  class (Functor f) => Applicative f where
    pure :: a -> f a
    (<*>) :: f (a -> b) -> f a -> f b
  ```  

  첫 줄은 Applicative가 functor의 일종이라는 선언입니다. Swift에서는 상속으로 나타나는 것이죠. 두번째 줄은 a라는 타입의 값을 받아서 Applicative로 래핑하는 함수입니다. 세번째 줄은 함수가 담긴 Applicative를 또 다른 Applicative에 매핑하는 연산자입니다. 이 때 첫번째 인자로 주어지는 Applicative는 a->b 형식의 함수이고, 이 함수를 a 타입 변수가 담긴 Applicative에 적용하여 b 타입의 변수가 담긴 Applicative를 반환합니다. 여기서 흥미로운 것은 a와 b가 또 다른 함수 타입이여도 된다는 것입니다. 이를 swift에서 재현하기는 어렵지만, 다음과 같이 비슷하게 해볼 수는 있습니다.  

  ```swift  
  class Applicative<T>: Functor<T> {
      func apply<U>(applicative: Applicative<(T)->(U)>) -> Applicative<U>
  } 
  precedencegroup LeftAssociative {
        associativity: left
    }

  infix operator <*>: LeftAssociative

  func <*><T, U>(lhs: Applicative<(T)->(U)>, rhs: Applicative<T>) -> Applicative<U> {
      return rhs.apply(lhs)
  }
  ```  

  지겹지만 이제 드디어 모나드에 도착했습니다. 그나마도 모나드 전체를 보지 않고, Monad의 핵심이 되는 바인딩 연산자(>>=)만 볼 것입니다. 

  ```haskell
  class Monad m where
   (>>==) :: m a -> (a -> m b) -> m b  
  ```  

  > Monad는 Applicative Functor의 발전된 형태입니다. 그런데 Monad에는 Applicative 제약이 없습니다. 이는 단지 Monad가 Applicative보다 먼저 Haskell에 도입되었기 때문으로, 제약이 없더라도 모든 Monad는 Applicative Functor입니다.  

  바인딩 연산자는 a 타입이 담긴 모나드 변수와 a 타입 변수를 받아 b 타입 변수가 담긴 모나드를 반환하는 함수를 받아서 b 타입 변수가 담긴 모나드를 반환하는 연산자 입니다. 이를 swift로 바꿔보면 다음과 같습니다.  

  ```swift
  class Monad<T>: Applicative<A> {
      func bind<U>(_ f: (T) -> (Monad<U>) ) -> Monad<U>
      }
  }
  ```  

  이 패턴도 익숙하지 않나요? 이는 flatMap과 유사합니다. 실제로 하스켈에서는 어떤 함수를 모나드로 만들기 위한 liftM이라는 함수를 제공하는데, 이 함수가 flatMap에 해당합니다.  

* **결론**  
  먼 길의 결론은 이것입니다. 

  1. map을 적용할 수 있는 것은 functor이다.
  2. flatMap을 적용할 수 있는 것은 Monad이다.  

  > Applicative Functor는 Swift에서 기본 구현체를 제공해주지 않기 때문에 결론에서 제외했습니다. 하지만 Functor를 확장을 통해 손쉽게 Applicative Functor로 만들 수 있기 때문에 크게 문제가 되는 부분은 아닙니다.  

  이렇게 모나드 개념이 필요한 이유는 무엇일까요? 이는 어떤 타입에 대한 추가적인 컨텍스트를 제공하고, 이를 한 묶음으로 처리할 수 있도록 해줍니다. 물론 실제 모나드의 개념과 그 유용성은 이렇게만으로 설명하기에는 한참 모자랍니다. 하지만 입문으로써, 그 가치가 조금이라도 있다면 그것만으로 이 글의 의미는 충분할 것 같습니다.

  Swift에서 제공하는 모나드의 예시로는 Optional, Sequence 등이 있습니다. 이 중 Optional에서 모나드 개념이 어떻게 사용되는지 다음 포스트를 통해 더 알아보도록 하겠습니다.  
  
---  

> 참조문헌  
> [Learn You a Haskell for Great Good! A Beginner's Guide(번역서: 가장 쉬운 하스켈 책)](http://learnyouahaskell.com)  
> [Swift Functors, Applicatives, and Monads in Pictures](https://www.mokacoding.com/blog/functor-applicative-monads-in-pictures/)  