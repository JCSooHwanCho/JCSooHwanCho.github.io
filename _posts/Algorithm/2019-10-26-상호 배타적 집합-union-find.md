---
layout: post
title: 상호 배타적 집합-union-find
comments: true
tags: [Algorithm, 집합]
category: [Algorithm]  
---  

* **정의**  
    상호 배타적 집합(disjoint set)은 전체 집합에서 공통 원소를 가지지 않는 여러 부분 집합들을 저장하고 조작하는 자료구조입니다. 

    상호 배타적 집합은 다음과 같은 세가지 연산을 지원해야 합니다.  

    1. 초기화: 모든 원소들이 모두 별도 집합에 속하도록 초기화합니다.  

    2. 합치기(union): 두 원소 a,b가 주어질 때, 각각이 속한 집합을 하나로 합칩니다.  

    3. 찾기(find): 어떤 원소가 주어졌을 때, 해당 원소가 속한 집합을 반환합니다.  

    이 때, 합치기와 찾기 연산을 지원한다고 하여 이를 union-find라고도 부릅니다. 

* **구현**  
   1. 간단한 구현  
      가장 간단한 방법은 1차원 배열 하나로 이를 표현하는 것입니다. 

      ```swift
      struct NaiveUFSet {
        var root: [Int] // index가 속한 집합을 나타내는 배열

        init(_ N: Int) {
            root = Array(0..<N) // 각 원소를 개별적인 집합에 속하게 한다.
        }

        mutating func find(_ x: Int) -> Int {
            return root[x]
        }

        mutating func union(_ x:Int, _ y:Int) {
            let a = self.find(x)
            let b = self.find(y)

            // a가 속한 집합을 b가 속한 집합에 합친다
            for (i, _) in root.enumerated() {
                if root[i] == a {
                    root[i] = b
                }
            }
        }
      }
      ```  
      이 경우 find 연산의 시간 복잡도는 O(1)입니다. 하지만 union 연산에는 O(N)의 시간이 소요됩니다. Union 연산이 많다면 이는 비효율적일 수 있습니다.  

    2. 트리를 이용한 구현  
        집합을 트리로 구현한다면 어떨까요? 이렇게 하면 두 집합의 동일성을 루트 노드를 비교함으로써 수행할 수 있고, Union연산은 한 집합을 다른 집합의 서브트리로 넣는 것으로 구현할 수 있습니다. find 연산과 union 연산의 변화를 눈여겨 보시기 바랍니다.

        ```swift
        struct UFSet {
            var root: [Int]

            init(_ N: Int) {
                root = Array(0..<N)
            }

            mutating func find(_ x: Int) -> Int {
                if root[x] == x {
                    return x
                }
                // 자신이 속한 집합의 루트 노드를 찾아 반환한다.
                return self.find(root[x])
            }

            mutating func union(_ x:Int, _ y:Int) {
                let a = self.find(x)
                let b = self.find(y)

                // b가 속한 집합을 a의 서브트리로 포함시킨다.
                root[b] = a 
            }
        }
        ```  
        이때 find 연산은 트리의 높이 만큼의 시간이 걸리기 때문에 평균적으로 O(logN), union 연산은 합치기 자체는 O(1)이자만 내부적으로 find를 사용하기 때문에  시간 복잡도는 find와 동일합니다. 

        하지만 만약 트리가 한쪽으로 치우쳐져 있다면, find는 O(N)에 가까워지고, union도 마찬가지로 O(N)에 가까워집니다. 이러면 오히려 배열로 하는 것만도 못해지죠.  

* **최적화**  
  union-find를 최적화 하기 위해서는 결국 높이가 심각하게 증가하는 것을 막아야 합니다. 이를 위한 방법은 두가지가 있습니다.  

  1. 랭크에 의한 합치기(union-by-rank): 높이가 낮은 트리를 항상 높이가 높은 트리의 서브트리로 넣을 수 있도록 하는 것입니다. 이렇게 되면 트리의 높이가 증가하는 경우는 두 트리의 높이가 같은 경우 뿐입니다. 
  
  2. 경로 압축 최적화(path compression): 매번 루트를 찾을 필요 없이 아예 해당 노드를 루트의 직계 자손 노드로 바꿔버리는 것입니다. 이렇게 되면 트리의 높이가 낮아져서 이후 find를 호출했을 때의 연산이 확연이 줄어들게 됩니다. 1번에서 높이를 기준으로 합치는 최적화와 함께 적용하면, find를 호출할 때마다 높이가 변화하므로 1번의 기준을 높이 대신 다른 기준을 잡을 필요가 있습니다.(그래서 높이가 아닌 랭크라는 용어를 사용합니다) 

  아래 코드는 이를 구현한 코드입니다. 여기서는 랭크를 집합의 원소의 수를 활용했습니다.  

    ```swift
    struct UFSet {
        var root: [Int]
        var count: [Int]

        init(_ N: Int) {
            root = Array(0..<N)
            count = Array(repeating: 1, count: N)
        }

        mutating func find(_ x: Int) -> Int {
            if root[x] == x {
                return x
            }

            root[x] = self.find(root[x]) // 현재 노드의 루트 노드를 바꿔준다.

            return root[x]
        }

        mutating func union(_ x:Int, _ y:Int) {
            let a = self.find(x)
            let b = self.find(y)

            let totalCount = count[a] + count[b] 

            if a == b {
                return
            } else if count[a] > count[b] {
                root[b] = a
                count[a] = totalCount // 랭크를 조정해준다.
            } else {
                root[a] = b
                count[b] = totalCount
            }
        }
    }
    ```  

    union-find는 자체적으로 쓰이지만 다른 알고리즘의 일부로써 사용되는데, 대표적으로 크루스칼 알고리즘이 있습니다.  

* **관련 문제**
  * [LeetCode 684 - Redundant Connection](https://leetcode.com/problems/redundant-connection/)  
  * [Programmers - 섬 연결하기](https://programmers.co.kr/learn/courses/30/lessons/42861)