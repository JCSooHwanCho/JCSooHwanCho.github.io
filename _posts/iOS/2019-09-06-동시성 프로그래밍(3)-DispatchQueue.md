---
layout: post
title: 동시성 프로그래밍(3) - DispatchQueue
comments: true
tags: [iOS 프로그래밍,Swift,Apple]
category: [iOS]
---  

이번 포스트에서는 OperationQueue와 대비되는 동시성 기술인 DispatchQueue에 대해서 알아보도록 하겠습니다.

> 이 포스트는 다음 가이드를 참고로 하여 작성되었습니다.  
> 
>  [Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091-CH1-SW1)


## DispatchQueue란
DispatchQueue는 여러 작엄들을 비동기적이고 동시적으로 수행하게 해주는 쉬운 방법을 제공하는 기술입니다. 이러한 작업들을 적절히 쪼개서 함수나 클로저에 넣어서 DispatchQueue에 넣는 것만 하면 나머지 관리는 시스템이 알아서 해줍니다.

DispatchQueue는 OperationQueue와는 다르게 무조건 먼저 들어간 작업을 먼저 수행하는 것이 보장됩니다.

시스템에서는 몇가지 기본적인 DispatchQueue를 제공하고, 필요하다면 직접 만드는 것도 가능합니다.

| 타입                                    |                             설명                             |
| --------------------------------------- | :----------------------------------------------------------: |
| Serial   | 한 번에 하나의 작업만 수행하며, 실행 순서는 Queue에 추가된 순서대로입니다. 실행되는 작업은 별도의 스레드 상에서 실행됩니다.(작업에 따라 달라질 수는 있습니다) 주로 특정 리소스에 동기적인 접을 할 때 사용됩니다.<br />여러 개의 SerialQueue를 사용하게 되면, 이 Queue 별로 하나의 작업을 수행하게 되면서 전체적으로 보면 동시에 여러 작업이 수행되는 것도 가능합니다. |
| Concurrent | 한 개 이상의 작업을 동시에 실행시킵니다. 하지만 실행순서는 여전히 큐에 들어온 순서입니다.  역시 작업은 별도 스레드에서 수행됩니다. 동시에 실행되는 작업의 수는 시스템의 상태에 따라 달라집니다.<br />지금은 ConcurrentQueue를 사용자가 직접 만들 수 있게 되었고, 여러 전역 Queue가 기본으로 제공되기도 합니다.|
| Main Dispatch Queue                     | 메인 스레드와 연결된 SerialQueue 입니다. 여기에 들어간 작업은 애플리케이션의 메인 RunLoop와 함께 동작하면서 다른 Input Source들의 실행 단계 사이에 끼어들어가 실행되게 됩니다. 메인 스레드에서 동작하기 때문에, 중요한 동기화 지점이 됩니다. |

스레드를 사용할 때에 비해서, DispatchQueue가 얻을 수 있는 이점은 여러가지가 있습니다.

1. 스레드를 만들고 관리하는 것을 시스템이 하기 때문에 훨씬 간결합니다. 시스템은 현재 시스템 상태와 자원에 따라 유동적으로 스레드 수를 조절할 수 있고, 종종 더 빠르게 작업을 시작할 수도 있습니다.

2. Serial의 경우는, 작업의 수행순서가 보장되기 때문에 lock을 사용하지 않고도 동기화가 가능해집니다. 덕분에 불필요한 커널 모드 진입을 최소화할 수 있습니다. (물론 여러개의 SerialQueue가 같은 자원에 접근하게 하면 문제가 됩니다)

3. 스레드를 만들때 커널 영역과 유저 영역 메모리를 모두 확보해야하는 비용이 드는데, DispatchQueue는 이러한 비용이 들지 않습니다.

4. DispatchQueue를 통해 생성된 스레드는 블록되지 않고 계속해서 연산을 수행합니다.

그 외에도 DispatchQueue가 가지는 특징들은 다음과 같습니다.

1. DispatchQueue의 순차성은 한 Queue 단위에서만 적용됩니다.

2. 아무리 Queue를 많이 만들어도, 동시에 수행되는 작업의 수는 시스템에 의해 결정됩니다.

3. 시스템은 새로이 수행할 작업을 결정할 때, Queue의 우선순위를 고려합니다.

4. Queue에 작업이 들어갈 때, 이미 실행할 준비가 완료된 상태여야 합니다.

## DispatchQueue 관련 기술  
DispatchQueue는 같이 사용할 수 있는 몇가지 기술들과 함께 제공됩니다. 

* Dispatch Group : 여러개의 작업 단위 클로저를 묶어서 완료를 감시할 수 있는 방법입니다. (동기방법, 비동기 방법 모두 가능합니다.) Group은 다른 여러 작업들에 의존성을 가지는 작업에 대한 유용한 동기화 방법을 제공합니다.

* Dispatch Semaphores : 전통적인 세마포어와 비슷하지만, 좀 더 효율적인 방법을 제공합니다. 현재 세마포어가 사용이 가능한 경우에는 시스템 호출을 하지 않기 때문입니다.

* Dispatch Source : 특정 시스템 이벤트를 받아서, 정해진 작업을 자동으로 지정된 Queue에 집어넣는 역할을 합니다.

## DispatchQueue 사용하기
 DispatchQueue를 사용하기로 했다면 어떤 Queue를 사용할지를 결정해야 합니다. 또한 특정한 방법으로 Queue를 사용하고 싶다면 Queue를 직접 구성할 수도 있습니다.

* Global Concurrent Queue얻기
   Global Queue는 Concurrent Queue이므로, 여러 작업을 동시에 수행하는데에 유용합니다. 직접 Queue를 만들 필요가 없다면 이 메소드를 사용해 이미 있는 Queue를 사용하는 게 편합니다.

   ```swift
    class func global(qos: DispatchQoS.QoSClass = .default) -> DispatchQueue
   ```   

   인자로 주어지는 Qos인자는 Queue의 우선순위 값으로, 기본 값은 default입니다. 이렇게 얻은 Global Queue는 편하게 쓸 수 있지만, 작업 중단 및 재개 기능 등은 지원하지 않습니다.

* Serial Queue만들기  
  ConcurrentQueue와 다르게 Serial Queue는 직접 만들어 사용해야 합니다. 이 떼 Queue를 여러개 만들어 실행하면 동시에 실행은 되겠지만, 이런 용도라면 Concurrent Queue를 고려하는 게 좋을 것입니다.

  ```swift
  let serialQueue = DispatchQueue(label:"someIdentifier") // 나머지 인자는 모두 default로 둡니다. 
  ```  

  예외적으로 메인 스레드에 대해서는 시스템이 자동으로 SerialQueue를 만들어주고, 이를 DispatchQueue의 타입 프로퍼티로 사용할 수 있게 해줍니다.

  ```swift
  DispatchQueue.main.async {
      // Some Task...
  }
  ```  

## Queue에 작업 넣기
Queue에 작업을 넣는 것은 두가지 방법이 있습니다.

1. 비동기적(async) : Queue에 추가한 뒤 바로 원래 흐름으로 돌아옵니다. 일반적으로 많이 쓰입니다.

2. 동기적(sync) : Queue에 추가한 뒤, 현재 스레드를 블록하고 추가한 작업이 끝나야만 원래대로 돌아옵니다. 경쟁 상태나 기타 동기화 문제를 해결할 때 쓰입니다.

> 어떤 작업이 자신을 실행중인 Queue에 대해서 동기적으로 작업을 넣어서는 안됩니다. Serial Queue라면 확정적으로 데드락(deadlock) 상태가 되며, Concurrent Queue에서도 바람직한 상황은 아닙니다.

병렬화가 가능한 작업을 최적화한다면 [concurrentPerform(iterations:execute:)](https://developer.apple.com/documentation/dispatch/dispatchqueue/2016088-concurrentperform) 메소드를 쓰는 것이 좋습니다. 자동으로 여러 코어를 활용할 수 있도록 최적화가 되어있기 때문입니다. 다만 수행하는 작업이 너무 작으면 오히려 스케쥴링 오버헤드가 더 클 수도 있습니다.

반드시 메인 스레드에서 실행되어야 할 작업이라면, DispatchQueue의 main프로퍼티를 통해 메인스레드의 SerialQueue를 얻은 뒤, 여기에 작업을 넣으면 됩니다.

## Queue의 추가기능들

* 시스템이 제공해주는 Queue가 아니라면, [suspend()](https://developer.apple.com/documentation/dispatch/dispatchobject/1452801-suspend)와 [resume()](https://developer.apple.com/documentation/dispatch/dispatchobject/1452929-resume)을 이용해 Queue의 동작을 멈추거나 재개할 수 있습니다. 이 때, suspend() 메소드는 Queue 내부의 suspend 카운트를 하나 늘리는 역할을 하고, resume() 메소드는 이를 하나 낮추는 역할을 합니다. 이 카운트가 0보다 크면 새로운 작업이 실행되는 것이 중단됩니다. 따라서 suspend()와 resume()은 정확히 같은 수만큼 호출을 해줘야지만 제대로 Queue가 돌아갈 수 있습니다. 당연하게도 이미 실행중인 작업은 이 suspend() 메소드에 영향을 받지 않습니다.  

* [DispatchSemaphore](https://developer.apple.com/documentation/dispatch/dispatchsemaphore)를 이용하면 특정 자원에 동시 접근하는 작업의 수를 제한할 수 있습니다. 전통적인 세마포어에 비해 좀 더 성능상의 이점이 있기 때문에 이것을 사용하는 게 좋습니다. [wait()](https://developer.apple.com/documentation/dispatch/dispatchsemaphore/2016071-wait) 함수로 세마포어를 감소시키고(접근 권한을 얻고), signal로 세마포어를 증가시킵니다.(접근 권한을 반환합니다) 이때 감소시킨 세마포어가 0보다 작아지면, 다시 0보다 커질때까지 스레드가 대기 상태에 빠지게 됩니다. 

```swift
let semaphore = DispatchSemaphore(value: 2) // 최대 2개의 작업만 접근할 수 있는 세마포어

semaphore.wait()
// 중요한 작업...
semaphore.signal()
```

* [DispatchGroup](https://developer.apple.com/documentation/dispatch/dispatchgroup)은 그룹내의 다른 작업들이 모두 완료될 때 까지 스레드를 블록하는 방법을 제공합니다. 이는 Apple의 프레임워크가 joinable 스레드를 만들지 못하는 것에 대한 대안을 제시해줍니다.

```swift
   let queue = DispatchQueue.global()
   let group = DispatchGroup()
        
    queue.async(group: group) {
        // task1
    }

    queue.async(group:group) {
        // task2
    }

    group.wait() // 두 작업이 모두 완료 될 때까지 스레드가 블록됨
```

---  

DispatchQueue에 대해서 상세하게 알아보았습니다. 다음에 좀 더 흥미로운 주제를 가지고 더 알아보도록 하겠습니다!