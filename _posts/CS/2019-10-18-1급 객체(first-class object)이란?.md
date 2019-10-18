---
layout: post
title: 1급 객체(first-class object)란?
comments: true
tags: [CS, Programming, Language]
category: [ComputerScience]
---  

일급 객체라는 말이 요새는 많이 보편화가 된 것 같습니다. 제가 이 말을 처음 들었던 건 자바스크립트에서였는데, 단순히 자바스크립트에 국한된 개념은 아니더군요. 프로그래밍 언어론 수업시간에도 그렇게 강조되지는 않았던 것 같은데 이제는 대부분의 언어가 이 일급 객체 개념을 가지고 가고 있습니다. 오늘은 이 일급 객체에 대해서 알아보도록 하겠습니다.

일급 시민(first-class citizen 혹은 type,object,entity, value 라고도 합니다.)이란 무슨 혜택을 받는다는 게 아니라, 사용할 때 다른 요소들과 아무런 차별이 없다는 것을 뜻합니다. (No "arbitrary" restrictions apply to their use)  

이 개념은 [Christopher Strachey](https://en.wikipedia.org/wiki/Christopher_Strachey)라는 분의 강의 노트에서 처음으로 언급이 되었다고 합니다. 원문은 다음과 같습니다.

> First and second class objects. In ALGOL, a real number may appear in an expression or be assigned to a variable, and either of them may appear as an actual parameter in a procedure call. A procedure, on the other hand, may only appear in another procedure call either as the operator (the most common case) or as one of the actual parameters. There are no other expressions involving procedures or whose results are procedures. Thus in a sense procedures in ALGOL are second class citizens—they always have to appear in person and can never be represented by a variable or expression (except in the case of a formal parameter)... 
>
> 번역 : 일급 객체와 이급 객체. ALGOL에서 실수는 표현식에서도 나올 수 있고, 변수에 대입할 수도 있고, 프로시저 호출에 매개변수로 넘길 수도 있다. 하지만 프로시저는 다른 프로시저 호출에서 연산자나 실질적 매개변수로 밖에 사용할 수 없다.(실질적 매개변수에 대해서는 [다음을 참조](https://chortle.ccsu.edu/Java5/Notes/chap34A/ch34A_3.html)) 프로시저를 포함하거나 프로시저를 반환값으로 가지는 표현식은 존재하지 않는다. 따라서  ALGOL에서 프로시저는 이급 시민이다. 이는 다른 요소들 사이에서만 존재할 수 있고, 단일 변수나 표현식으로 나타날 수 없다는 것이다(형식적 매개변수의 경우를 제외하고)  

위 문맥에서는 명확한 정의를 내리고 있지 않습니다. 하지만 이후 [Robin Popplestone](https://en.wikipedia.org/wiki/Robin_Popplestone)이라는 분이 좀더 명확한 기준을 제시하였습니다.  

1. 모든 일급 객체는 함수의 실질적인 매개변수가 될 수 있다.
2. 모든 일급 객체는 함수의 반환값이 될 수 있다.
3. 모든 일급 객체는 할당의 대상이 될 수 있다.
4. 모든 일급 객체는 비교 연산(==, equal)을 적용할 수 있다.
   
우리가 그동안 자연스럽게 써온 변수들은 대부분이 일급 객체입니다. (단, C언어등에서 배열과 문자열 등은 인자로 넘길 때 포인터 형태로 넘어가야 하기 때문에 일급 객체로 볼 수 없습니다.) 이를 중요하게 여기는 곳은 주로 함수형 언어, 혹은 그 영향을 받은 언어들입니다. 이들은 함수 역시 일급 객체로 취급하기 때문입니다.

제가 swift를 사용하기 때문에 swift의 경우로 살펴보도록 하겠습니다. 


1. 함수의 실질적인 매개 변수가 될 수 있다. 

    ```swift
    let f:(Data) -> Void = { data in
        // ...
    } 

    func doSomething(withData data: Data,action: (Data)-> Void) {
        action(data)
    }

    doSomething(data, action) 
    ```  

2. 함수의 반환 값이 될 수 있다.

    ```swift
    func getClosure() -> (Data) -> Void {
        return f
    }
    ```  

3. 할당의 대상이 될 수 있다. -> 1번에서 이미 할당을 해주고 있습니다.

4. 비교 연산이 가능합니다. 다만 이를 swift는 권장하지 않습니다([참조](https://stackoverflow.com/questions/24111984/how-do-you-test-functions-and-closures-for-equality))  

이렇게 함수를 일급 객체로 취급하는 것은 특히 함수형 언어에서 필수적인 속성입니다. 막상 살펴보니 어려운 개념은 아니였습니다.