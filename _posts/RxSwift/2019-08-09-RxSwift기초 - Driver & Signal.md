---
layout: post
title: RxSwift기초 - Driver & Signal
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

오늘은 Driver와 Signal에 대해 알아보도록 하겠습니다. Driver와 Signal은 다른 구현체에는 존재하지 않고, UI에 사용하기 위해 RxCocoa에 도입된 옵져버블의 특수 케이스입니다.  

Driver와 Signal을 구현하기 위한 여러 겹의 코드가 있지만, 그 중 핵심 코드를 하나 들라하면 다음을 들 수 있습니다. 

```swift
 source.share(replay: 1, scope: .whileConnected) // driver의 핵심 선언부
 source.share(replay: 0, scope: .whileConnected) // signal의 핵심 선언부
```

원래는 새로운 옵저버가 옵저버블을 구독할 때마다 새로운 스트림이 생겨납니다. 부수 효과(Side-effect)가 없고 짦은 시간 동작하고 사라질 스트림이라면 상관 없지만, 부수 효과가 있거나 긴 시간 동작하면서 메모리에 상주하거나 아예 종료되지 않는 것이라면 필연적으로 메모리 문제가 생깁니다.  

이러한 문제를 해결하기 위해서 하나의 스트림의 값을 공유하는 방법이 필요한데 크게 두가지 방법이 있습니다.  

하나는 이전에 살펴본 [Subject](/2019-08-04-RxSwift기초-Subject)를 프록시로 두는 방법이고, 다른 하나는 share 연산자를 사용하는 것입니다.(본래 Rx에서 [Replay](http://reactivex.io/documentation/operators/replay.html)에 해당하는 것입니다.)  

이를 좀 더 자세히 풀어보면, '구독이 1개 이상 존재하는 상황에서만 유지되는 공유 옵져버블'입니다. UI이벤트의 소스는 하나뿐이고, 같은 이벤트가 모든 옵저버에게 보여야 하니까 이런식으로 사용하는 것이 적절하다고 할 수 있습니다.  

Driver와 Signal에는 또 다음과 같은 특징이 있습니다.  

1. 절대 종료되지 않습니다. 
2. 무조건 MainScheduler에서 돌아가게 되고, 실제로도 Mainscheduler에서 쓰는 것을 전제로 사용해야 합니다. observeOn()으로 동작하는 스케쥴러를 바꿔줄 수는 있겠지만 권장되는 방법은 아닙니다.
3. 구독 역시 MainScheduler에서 하는 것이 권장됩니다. 처음 구독을 다른 스케쥴러에서 하면 Signal은 몰라도 Driver의 경우, 구독시 가장 최근의 이벤트를 받아오기 때문에 이를 다른 스케쥴러에서 실행해버리는 문제가 생길 수 있습니다.  

Driver와 Signal의 차이는 다음 표로 정리해 볼 수 있습니다.  

  |                                       | Driver  | Signal |
  | :-----------------------------------: | :-----: | :----: |
  |              구독 메소드              | drive() | emit() |
  | 구독시 가장 최근 이벤트를 받는지 여부 |    O    |   X    |

이를 보면 Driver는 BehaviorRelay, Signal은 PublishRelay와 많은 공통점을 가지고 있고 실제로 같이 쓰는 경우가 많습니다.


>일반 옵져버블도 Driver나 Signal로 변환해줄 수 있습니다.
>```swift
>// asDriver의 함수 정의 중 하나, Signal도 이와 비슷하다.
>public func asDriver(onErrorJustReturn: Element) -> Driver<Element> {
>	let source = self
>  	.asObservable()
>  	.observeOn(DriverSharingStrategy.scheduler) // MainScheduler를 나타낸다.
>		.catchErrorJustReturn(onErrorJustReturn) // error나 completed가 된 경우 대신 내보낼 값. 
>	return Driver(source)
>```
>여기서 Driver와 Signal의 특징인 
>1. MainScheduler에서 동작한다.
>2. error나 Completed가 발생하지 않는다.  
>가 명확하게 드러납니다.


---   

Driver와 Signal에 대한 개념을 간단하게 정리해보았습니다. 사실 파고들면 더 많은 부분을 설명해야 하지만, 사용할 때는 이 정도만 알아도 충분히 사용할 수 있을거라 생각됩니다.