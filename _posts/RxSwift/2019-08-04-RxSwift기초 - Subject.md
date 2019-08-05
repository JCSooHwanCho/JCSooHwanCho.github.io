---
layout: post
title: RxSwift기초 - Subject
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

RxSwift 스터디를 준비하면서 정리한 내용을 블로그에도 올리고자 합니다. 오늘은 Subject에 대해서 알아보도록 하겠습니다.

다음은 Subject의 [ReactiveX 공식 홈페이지](https://reactivex.io)의 정의입니다.

> A Subject is a sort of **bridge or proxy** that is available in some implementations of ReactiveX that **acts both as an observer and as an Observable**

즉, Subject는 어떤 둘 사이의 관계에 들어가는 다리입니다. 그리고 이 다리는 옵저버와 옵저버블의 역할을 동시에 수행해야 합니다. 옵저버는 옵저버블을 구독하는 역할이고, 옵저버블은 옵저버에 이벤트를 전달하는 역할이므로 다음과 같이 정리할 수 있을 것 같습니다.

> 어떤 옵저버블로부터 이벤트를 전달 받아서(옵저버) 자신을 구독하고 있는 옵저버들에게 해당 이벤트를 전달하는 (옵저버블) 것

이러한 기능을 수행하기 위해 Subject는 자신이 맡은 역할의 핵심 메소드들을 가지게 되는데, 이는 다음과 같습니다.
  - 옵저버블의 특성 : subsciribe() 메소드
  - 옵저버의 특성 : on() 메소드

--- 

1.  BehaviorSubject

    - 상태값을 가집니다. 이 상태값은 subject 생성시 주어지는 인자로 초기화되고,  자신이 마지막으로 내보낸 값으로 바뀝니다.

    - 상태값을 value() 메소드로 참조 가능합니다.

    - value() 메소드는 BehaviorSubject에 error가 발생했거나 dispose된 경우는 에러를 던집니다. 

    - 구독을 할 경우, 자신의 현재 상태 값을 첫번째 이벤트로 내보냅니다.

    - next 이벤트가 들어오면, 자신의 상태값을 갱신하고 자신을 구독하고 있는 모든 옵저버들에게 이벤트를 내보냅니다.

    - completed나 error 이벤트(이하, 종료 이벤트)가 발생한 경우, 해당 Subject는 종료되고 종료 이벤트가 모든 옵저버에 전달됩니다.

    - 종료된 Subject에 접근할 경우, 해당 subject가 마지막으로 발생시킨 종료 이벤트를 받게 됩니다.(completed면 completed, error면 error)  

    ![BehaviorSubject]({{"/img/BehaviorSubject.png"}})

    ```swift
    public func on(_ event: Event<Element>) {
            dispatch(self._synchronized_on(event), event) //자신을 구독한 옵저버 리스트와 이벤트를 인자로 받아 리스트의 모든 옵저버에 이벤트를 방출한다.
        }

    func _synchronized_on(_ event: Event<Element>) -> Observers {
            self._lock.lock(); defer { self._lock.unlock() }
            if self._stoppedEvent != nil || self._isDisposed {
                return Observers()
            }
            
            switch event {
            case .next(let element):
                self._element = element
            case .error, .completed:
                self._stoppedEvent = event
            }
            
            return self._observers
        }

    public override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
            self._lock.lock()
            let subscription = self._synchronized_subscribe(observer)
            self._lock.unlock()
            return subscription
        }

    func _synchronized_subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
            if self._isDisposed {
                observer.on(.error(RxError.disposed(object: self)))
                return Disposables.create()
            }
            
            if let stoppedEvent = self._stoppedEvent {
                observer.on(stoppedEvent)
                return Disposables.create()
            }
            
            let key = self._observers.insert(observer.on)
            observer.on(.next(self._element))
        
            return SubscriptionDisposable(owner: self, key: key)
        }
    ```

2. PublishSubject

   - BehaviorSubject와 비슷하나, 가장 큰 차이점은 상태값이 없다는 것입니다.

   - 상태값이 없기 때문에, 구독해도 바로 이벤트를 받을 수는 없습니다.

   - next 이벤트가 들어오면, 자신을 구독한 모든 옵저버에게 이벤트를 전달합니다.

   - 종료 이벤트를 받으면, 종료 이벤트를 내보내고 해당 Subject는 종료됩니다.  더이상의 구독을 유지할 이유가 없기 때문에 현재의 구독을 모두 해지합니다.

   - 종료된 PublishSubject를 구독하면, 마지막으로 발생한 종료 이벤트를 받게 됩니다.

    ![PublishSubject]({{"/img/PublishSubject.png"}})

    ```swift
    //PublishSubject의 on메소드와 subscribe 메소드 
    public func on(_ event: Event<Element>) {
            dispatch(self._synchronized_on(event), event)
        }

    func _synchronized_on(_ event: Event<Element>) -> Observers {
            self._lock.lock(); defer { self._lock.unlock() }
            switch event {
            case .next:
                if self._isDisposed || self._stopped {
                    return Observers()
                }
                
                return self._observers
            case .completed, .error:
                if self._stoppedEvent == nil {
                    self._stoppedEvent = event
                    self._stopped = true
                    let observers = self._observers //구독을 끊기 전, 종료 이벤트를 보내기 위해 구독 정보를 복사한다. (observers 목록은 struct이므로 값이 복사된다)
                    self._observers.removeAll() //구독 정보를 모두 지운다.
                    return observers
                }

                return Observers()
            }
        }

    public override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
            self._lock.lock()
            let subscription = self._synchronized_subscribe(observer)
            self._lock.unlock()
            return subscription
        }

    func _synchronized_subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
            if let stoppedEvent = self._stoppedEvent {
                observer.on(stoppedEvent)
                return Disposables.create()
            }
            
            if self._isDisposed {
                observer.on(.error(RxError.disposed(object: self)))
                return Disposables.create()
            }
            
            let key = self._observers.insert(observer.on)
            return SubscriptionDisposable(owner: self, key: key)
        }
    ```

3. AsyncSubject

   - completed 되어야만 값이 나옵니다.

   - completed 이전의 next 이벤트에 대해서는 아무것도 내보내지 않습니다.

   - 구독해도 subject가 종료 될 때까지는 아무것도 받아볼 수 없습니다.

   - subject가 종료 이벤트를 받으면 마지막으로 발생된 next이벤트를 방출하고, 종료 이벤트를 뒤 이어 보낸 뒤 모든 구독을 끊습니다.

   - subject가 값을 내보내고 종료된 후에 구독할 경우, 내보낸 값과 completed이벤트를 받게 됩니다. 

   - 만약 값이 없이 그냥 completed된 경우는 값없이 그냥 completed만 받게 되며, error의 경우에는 error를 받게 됩니다.

    ![AsyncSubject]({{"/img/AsyncSubject.png"}})

    ```swift
    public func on(_ event: Event<Element>) {
            let (observers, event) = self._synchronized_on(event)
            switch event { // 인자로 받은 이벤트가 아니라 _synchronized_on(event)의 반환값으로 나온 event
            case .next:
                dispatch(observers, event)
                dispatch(observers, .completed)
            case .completed:
                dispatch(observers, event)
            case .error:
                dispatch(observers, event)
            }
        }

    func _synchronized_on(_ event: Event<Element>) -> (Observers, Event<Element>) {
            self._lock.lock(); defer { self._lock.unlock() }
            if self._isStopped {
                return (Observers(), .completed)
            }

            switch event {
            case .next(let element):
                self._lastElement = element
                return (Observers(), .completed) //아무에게도 이 이벤트를 전달하지 않는다.
            case .error:
                self._stoppedEvent = event

                let observers = self._observers
                self._observers.removeAll()

                return (observers, event)
            case .completed:

                let observers = self._observers
                self._observers.removeAll()

                if let lastElement = self._lastElement {
                    self._stoppedEvent = .next(lastElement)
                    return (observers, .next(lastElement))
                }
                else {
                    self._stoppedEvent = event
                    return (observers, .completed)
                }
            }
        }

    public override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
            self._lock.lock(); defer { self._lock.unlock() }
            return self._synchronized_subscribe(observer)
        }

    func _synchronized_subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
            if let stoppedEvent = self._stoppedEvent {
                switch stoppedEvent {
                case .next:
                    observer.on(stoppedEvent)
                    observer.on(.completed)
                case .completed:
                    observer.on(stoppedEvent)
                case .error:
                    observer.on(stoppedEvent)
                }
                return Disposables.create()
            }

            let key = self._observers.insert(observer.on)

            return SubscriptionDisposable(owner: self, key: key)
        }
    ```

4. ReplaySubject
   - 특정 횟수만큼의 과거 이벤트를 저장하고 있는 BehaviorSubject라 생각하면 됩니다.

   - 내부에 큐를 가지고 있습니다.
   
   - next 이벤트가 발생할 때마다 해당 큐에 값이 들어가고, 정해진 갯수를 넘을 경우 큐에서 값들을 제거합니다.

   - 새로 구독할 때, 오래된 값부터 최근 값까지 정해진 수 만큼 연속적으로 이벤트를 받게 됩니다.

   - subject가 종료될 경우, 종료 이벤트를 보낸 후에, 모든 구독을 해제합니다.

   - subject가 종료된 이후에  구독을 할 경우, 가지고 있던 값들을 모두 이벤트로 발생 시킨 후, 종료 이벤트를 내보냅니다.

   - 다른 subject와 다르게, ReplaySubject는 추상 클래스이고, 서브클래스들도 모두 private class이기 때문에 직접 인스턴스를 만들 수 없습니다. 
   
   - create라는 팩토리 메소드를 통해 인스턴스를 만들어 사용해야 합니다

    ![ReplaySubject]({{"/img/ReplaySubject.png"}})

    ```swift
    override func on(_ event: Event<Element>) {
        dispatch(self._synchronized_on(event), event)
    }

    func _synchronized_on(_ event: Event<Element>) -> Observers {
        self._lock.lock(); defer { self._lock.unlock() }
        if self._isDisposed {
        return Observers()
        }

        if self._isStopped {
        return Observers()
        }

        switch event {
        case .next(let element):
            self.addValueToBuffer(element) // 현재의 이벤트를 저장한다.
            self.trim() // 정해진 갯수를 넘어갈 경우 오래된 이벤트부터 버퍼에서 제거한다.
            return self._observers
        case .error, .completed:
            self._stoppedEvent = event
            self.trim()
            let observers = self._observers
            self._observers.removeAll()
            return observers
        }
    }

    override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
        self._lock.lock()
        let subscription = self._synchronized_subscribe(observer)
        self._lock.unlock()
        return subscription
    }

    func _synchronized_subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
    if self._isDisposed {
        observer.on(.error(RxError.disposed(object: self)))
        return Disposables.create()
    }

    let anyObserver = observer.asObserver()

    self.replayBuffer(anyObserver) //이전에 발생했던 이벤트들을 지금 구독을 한 옵저버에게 보낸다.
    if let stoppedEvent = self._stoppedEvent {
        observer.on(stoppedEvent)
        return Disposables.create()
    }
    else {
        let key = self._observers.insert(observer.on)
        return SubscriptionDisposable(owner: self, key: key)
    }
    }
    ```

---  

다음에는 Relay에 대해서 알아보도록 하겠습니다.