---
layout: post
title: 동시성 프로그래밍(2) - OperationQueue
comments: true
tags: [iOS 프로그래밍,Swift,Apple]
category: [iOS]
---  

이번 포스트부터는 좀 더 구체적으로 동시성을 도입할 수 있는 기술들을 알아보도록 하겠습니다. 그 중에서 이번 포스트에서는 OperationQueue에 대해서 자세히 알아보도록 하겠습니다.

> 이 포스트는 다음 가이드를 참고로 하여 작성되었습니다.  
> 
>  [Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091-CH1-SW1)

> Apple이 동시성(Concurrency)을 구현하기 위해 비동기(asynchronous) 기법을 사용하였기 때문에, Apple의 가이드에서는 이 두가지 용어가 크게 구분되지 않고 사용됩니다. 

OperationQueue는 문자 그대로 보면 "Operation이 담긴 Queue"입니다. 따라서 Operation이 무엇인지부터 알아보겠습니다.

## Operation 객체  
  [Operation 객체](https://developer.apple.com/documentation/foundation/operation)는 Foundation 프레임워크에 정의되어 있으며, 애플리케이션이 수행하는 작업을 추상화하기 위해 사용합니다. Operation객체는 추상 클래스라 서브클래싱해서 사용해야 하지만, 서브클래스에서 해야할 일의 양을 최소화할 수 있는 기반을 제공합니다. 또한 Foundation 프레임워크는 Operation의 구체클래스도 제공하여 바로 사용할 수도 있습니다. 

  > Foundation 프레임워크는 원래 2개의 구체 Opearition 클래스를 제공합니다. 하나는 Selector를 기반으로 작동하는 [NSInvocationOperation](https://developer.apple.com/documentation/foundation/nsinvocationoperation), 다른 하나는 클로저를 기반으로 동작하는 [BlockOperation](https://developer.apple.com/documentation/foundation/blockoperation) 입니다. 하지만 Swift 버전에서는 BlockOperation만 제공하기 때문에, NSInvocationOperation 에 대한 설명은 생략합니다. 

    Operation 객체가 제공하는 기능은 다음과 같습니다.

    * Operation객체 간의 그래프 기반의 의존성을 지원합니다. 

    * Completion Block을 지원합니다. Operation의 메인 작업이 끝나면 실행됩니다.

    * 실행 상태를 모니터링할 수 있도록 KVO 프로그래밍을 지원합니다.

    * 우선순위를 줄 수 있어, 상대적인 실행 순서를 줄 수 있습니다.

    * Operation을 중간에 멈출 수 있는 기능을 제공합니다.

## 동기적 Operation vs 비동기적 Operation  
   Operation 객체를 주로 Queue에 넣어서 사용하지만, Operation 객체의 start() 메소드를 호출해서 수동으로 실행시킬 수도 있습니다. 하지만 이렇게 하면 Operation이 동시적으로 실행되는 것이 보장되지 않습니다. Operation 객체의 [isAsynchronous](https://developer.apple.com/documentation/foundation/operation/1408275-isasynchronous) 프로퍼티를 참조하면 이 Operation이 동시적으로 수행되는지 여부를 알 수 있는데, 기본 구현은 false입니다.(즉, 동시적으로 수행되지 않습니다.)

   따라서 동시적으로 Operation을 실행하려면 별도 스레드를 만들어 실행시키거나, 시스템이 제공하는 비동기 함수를 호출하거나, 기타 방법으로 start()메소드가 Operationd을 실행시키고 즉시 반환하도록 해야합니다.

   하지만 이런 작업들은 OperationQueue가 모두 대신 해줍니다. OperationQueue는 동시적이지 않은 Operation이 들어오면 해당 Operation을 실행시키는 스레드를 만들어 줍니다. 따라서 OperationQueue를 이용하지 않을 때만 수동으로 실행시키면 됩니다.

   > 이 포스트는 수동으로 Operation을 구성하고 실행하는 법에 대해서는 설명하지 않습니다.

## OperationQueue 사용법  
  OperationQueue를 만들고 유지하는 것은 애플리케이션의 책임입니다. Queue의 갯수는 제한이 없으나, 특정 시점에서 실행될 수 있는 작업의 수는 프로세서의 코어수와 시스템의 부하에 따른 실질적인 제한이 있습니다. 따라서 Queue를 늘린다고 해서 Operation을 더 많이 수행할 수 있는 것은 아닙니다.

  ```swift
  let queue = OperationQueue()

  queue.addOperation(someOperation) // operation을 Queue에 추가한다.
  queue.addOperations([someOperations]) // Operation을 배열에 담아 추가한다.
  queue.addOperation { // 클로저를 인자로 주어, 해당 클로저를 수행한다.
      // some task
  }
  ``` 
  OperationQueue는 [maxConcurrentOperationCount](https://developer.apple.com/documentation/foundation/operationqueue/1414982-maxconcurrentoperationcount) 프로퍼티를 통해서 동시에 수행될 수 있는 최대 Operation의 수를 제한할 수 있습니다. 또한 작업의 실제 업무량과 우선순위에 따라서도 실제 수행시간과 순서가 영향을 받게 됩니다. 즉, OperationQueue가 Serial DispatchQueue와 같은 동작을 제공해주기는 어렵습니다. 만약 작업 순서가 중요하다면, Operation에 의존성을 설정해줘야 합니다.  

  Operation이 Queue에 추가되면, 해당 Operation 객체는 Queue가 소유권을 갖게 되고 Queue에서 제거하는 것은 불가능합니다. 이를 제거하기 위해서는 Operation을 취소하는 방법밖에 없습니다. 이는 각 Operation에 대해 cancel() 메소드를 호출하거나 Queue의 CancelAllOperation() 메소드를 호출하면 됩니다. 

  단, Operation을 취소하는 것은 더이상 해당 Operation이 필요 없을 때만 수행되어야 합니다. Operation을 취소 상태로 만드는 것은 해당 Operation이 종료된 것으로도 취급하기 때문입니다. 따라서 앱 종료, 사용자의 취소 요청 등에 대해서 OperationQueue의 Operation을 선택적으로 취소하는 것보다 한꺼번에 취소하는 경우가 좀 더 흔하게 일어납니다.  

  만약 이 Operation으로 나온 결과가 필요해서 이 Operation의 종료를 기다려야 한다면, [waitUntilFinished()](https://developer.apple.com/documentation/foundation/operation/1409256-waituntilfinished) 메소드나 [waitUntilAllOperationsAreFinished()](https://developer.apple.com/documentation/foundation/operationqueue/1407971-waituntilalloperationsarefinishe)를 통해 현재 스레드를 블록할 수 있습니다. 하지만 이것은 가능하면 호출하지 않는 것이 좋습니다. 현재 스레드를 블록하는 것은 동시성에서 별로 좋지 않은 선택입니다

  OperationQueue의 isSuspended 프로퍼티를 이용하면 OperationQueue의 동작을 잠시 멈출수도 있습니다. 하지만 이미 실행되고 있는 작업을 멈추는 것은 아니고, 새로운 Operation이 실행되는 것을 막습니다. 이를 true로 설정하면 Queue가 기능을 잠시 멈추고, false로 설정하면 동작이 다시 재개됩니다.

  ---

 OperationQueue에 대해서 간단히 알아보았습니다. 다음 포스트에서는 DispatchQueue에 대해서 살펴보도록 하겠습니다.