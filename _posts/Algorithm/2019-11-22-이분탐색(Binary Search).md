---
layout: post
title: 이분탐색(Binary Search)
comments: true
tags: [Algorithm, 자료구조]
category: [Algorithm]  
---  

이번 포스트에서는 배열에서 원하는 원소의 존재 여부를 빠르게 탐색할 수 있게 만드는 이분 탐색에 대해서 알아보고, Swift에서 이를 활용하는 방법에 대해서 알아보겠습니다.  

* **선형 탐색**  
   어떤 배열에서 원하는 원소를 찾고자 한다면 어떻게 해야 할까요? 가장 간단한 방법은 배열의 첫 원소부터 모든 원소를 검사하는 것입니다.  

   ```swift
    extension Array where Element: Equatable {
        func linearSearch(_ element: Element) -> Int? {
            return self.filter
        }
    }
   ```  

   이러한 방식은 O(n)의 시간복잡도를 가집니다. 배열이 크고, 검색이 자주 일어난다면 이러한 시간복잡도는 부담스러울 수 있습니다. 하지만 배열의 원소가 정렬이 되어 있지 않고, 정렬할 수도 없을 경우, 이 방법 이상으로 확실하게 원하는 원소를 찾는 방법은 없습니다.  

* **이분탐색**  
  만약 배열이 정렬되어 있다면, 좀 더 효율적인 방법을 생각해볼 수 있습니다. 찾고자 하는 배열의 중간 값을 뽑아낸 뒤, 이 중간 값을 찾고자 하는 원소와 비교합니다. 찾고자 하는 원소를 element, 중간 값을 mid, 배열을 정렬할 때 사용한 비교 연산을 comparer 라고 하면 다음 세가지 경우 중 한 가지 경우가 됩니다.

    1. mid == element, 원하는 값을 발견했습니다.
    
    2. comparer(mid,element) == true: 찾고자 하는 값은 mid의 오른쪽에 있습니다. [mid+1...] 범위에서 다시 탐색을 수행합니다.
    
    3. comparer(mid,element) == false: 찾고자 하는 값은 mid의 왼쪽에 있습니다. [..<mid] 범위에서 다시 탐색을 수행합니다.  

  이렇게 탐색을 수행하다가 원하는 값을 찾거나, 찾는 범위의 길이가 0이 될 때 까지 이 탐색을 수행합니다. 이렇게 되면 찾는 범위는 계속해서 절반으로 줄어들고, 그 때 마다 O(1)의 비교 과정을 수행하므로, 전체 탐색 시간 복잡도는 O(log n)이 됩니다.  

  ```swift
    extension Array where Element: Comparable {
        // 배열이 comparer를 기준으로 정렬되어 있어야 올바르게 동작합니다.
        func binarySearch(_ element: Element, comparer: (Element, Element) -> Bool) -> Int? {
            if isEmpty {
                return nil
            }
            
            var begin = 0
            var end = count - 1

            while begin <= end {
                let mid = (begin + end) / 2

                if self[mid] == element {
                    return mid
                }

                if comparer(self[mid],element) {
                    begin = mid + 1
                } else {
                    end = mid - 1
                }
            }

            return nil
        }
    }
  ```  

* **활용성**  
  이분 탐색을 활용하기 위해서는 배열이 정렬되어 있어야 한다는 전제가 필요합니다. 따라서 임의의 배열에 이분 탐색을 활용하기 위해서는 먼저 평균 O(n log n)의 시간 복잡도를 가지는 정렬을 수행해야 합니다. 따라서 배열의 변화가 빈번하게 일어난다면 다른 자료구조를 찾아보는 게 낫습니다. 하지만 만약 배열의 앞뒤에서만 삽입,삭제가 일어난다면 원소의 삭제가 배열의 정렬 상태를 흐트러뜨리지 않기 때문에 삽입할 때만 정렬 상태를 유지할 수 있다면 배열을 힙 대신 활용할 수 있습니다. 이는 특히 힙의 기본 구현을 제공하지 않는 swift에서 더 유용합니다. 힙의 구현 보다는 이분 탐색을 구현하는 게 훨씬 간편하기 때문입니다.  

  ```swift
    extension Array where Element: Comparable {
        // 역시 배열이 comparer를 기준으로 정렬 되어 있음을 가정합니다.
        mutating func binaryInsert(_ element: Element, comparer: (Element, Element) -> Bool) {
            if isEmpty {
                self.append(element)
            }

            var begin = 0
            var end = count - 1

            while begin <= end {
                let mid = (begin + end)/2

                if comparer(self[mid],element) {
                    end = mid - 1
                } else {
                    begin = mid + 1
                }
            }
            self.insert(element, at: begin)
        }
    }
  ```

---  

이분 탐색은 그 유용성과 간편성 때문에 그 자체만으로뿐 아니라 여러가지 알고리즘의 일부로써 널리 쓰입니다.