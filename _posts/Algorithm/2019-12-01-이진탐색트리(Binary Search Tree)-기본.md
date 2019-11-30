---
layout: post
title: 이진탐색트리(Binary Search Tree)-기본
comments: true
tags: [Algorithm, 자료구조]
category: [Algorithm]  
---  

이번 포스트에서는 이진 탐색 트리(Binary Search Tree, BST)에 대해서 알아보고, 이를 실제로 구현해보도록 하겠습니다. 들어가기 전, 여기서 다루는 BST는 가장 기본적인 형태의 BST임을 밝힙니다.  

* **BST의 필요성**  
  [이분 탐색](/2019-11-22-이분탐색(Binary-Search)/)은 탐색시간을 효과적으로 줄여주었습니다. 하지만 그 특성상 배열에서 동작하기 때문에, 삽입과 삭제를 하기 위해서는 여러개의 원소를 움직여야 합니다. 이 과정은 O(n)이기 때문에, 삽입과 삭제가 빈번할 경우 탐색으로 얻은 시간 이득이 상쇄되어 버립니다. 이를 극복하기 위해 배열을 트리의 형태로 바꾸는 것이 BST의 기본 아이디어 입니다.

* **BST의 구현**  
  BST는 이진 탐색의 아이디어를 그대로 가져갑니다. 트리내의 모든 노드에서 왼쪽 서브트리는 현재 노드보다 작은 값들만 존재하고, 오른쪽 서브트리는 현재 노드보다 큰 값들만 존재합니다. BinaryTree 타입을 다음과 같이 정의하겠습니다. 

    ```swift
    struct BinaryTree<T: Comparable>: Equatable {
        // struct 자체는 재귀적인 구조를 지원하지 않기 때문에
        // indirect를 적용한 enum으로 재귀 구조를 표현한다.
        private indirect enum BinaryTreeNode<T: Comparable>: Equatable {
            case none
            case some(BinaryTree<T>, T, BinaryTree<T>)
        }
        
        private var root: BinaryTreeNode<T>

        init() {
            root = .none
        }

        private init(_ node: BinaryTreeNode<T>) {
            root = node
        }
    }
    ```  
  BST가 제공하는 연산은 다음과 같습니다.  

  1. 원소 존재 여부 확인(find): 찾고자 하는 트리가 존재하지 않는 경우가 베이스 케이스로 false를 반환합니다. 존재한다면, 루트 노드의 값과 비교해서 같다면 true를 반환합니다. 같지 않다면, 크기에 따라 왼쪽 서브트리(현재 노드의 값보다 작은 경우)나 오른쪽 서브트리(현재 노드의 값보다 큰 경우)에서 재귀적으로 탐색을 진행합니다.  

        ```swift
        extension BinaryTree {
            func find(_ value: T) -> Bool {
                switch root {
                case .none:
                    return false
                case let .some(left, current, right):
                    if current == value {
                        return true
                    } else if current < value {
                        return right.find(value)
                    } else {
                        return left.find(value)
                    }
                }
            }
        }
        ```  

  2. 원소 순회: 모든 원소를 오름차순으로 순회하기 위해서는, 중위 순회를 이용하면 됩니다.  

        ```swift
        extension BinaryTree {
            var list:[T] {
                switch root {
                case .none:
                    return []
                case let .some(left, current, right):
                    return left.list + [current] + right.list
                }
            }
        }
        ```  

  3. 원소 삽입: find와 비슷합니다. 다만 트리가 없는 경우에 새로운 노드를 만들어 삽입한다는 점에서 차이를 보입니다. 이 구현에서는 중복된 원소를 넣을 경우, 왼쪽 서브트리로 보내도록 구현했지만, 이 부분은 정책적인 부분입니다.

        ```swift
        extension BinaryTree {
            mutating func insert(_ value: T) {
                switch root {
                case .none:
                    self.root = .some(BinaryTree(), value, BinaryTree())
                case .some(var left, let current, var right):
                    if current < value {
                        right.insert(value)
                    } else {
                        left.insert(value)
                    }

                    self.root = .some(left, current, right)
                }
            }
        }
        ```  

  4. 원소 삭제: 이진 트리에서 가장 까다로운 부분입니다. 원하는 원소를 찾게 되면, 해당 원소를 제거하고, 왼쪽 서브트리와 오른쪽 서브트리를 합친 새로운 트리로 그 자리를 대체합니다. 이 때 합쳐진 트리의 루트는 왼쪽 서브트리로 만들었는데, 이 부분 역시 원하는 대로 할 수 있습니다. merge를 하는 부분을 주의 깊게 살펴보시길 바랍니다.  

        ```swift
        extension BinaryTree {
            mutating func remove(_ value: T) {
                switch root {
                case .none:
                    return
                case .some(var left, let current, var right):
                    if current == value {
                        self = left.merge(right)
                    } else if current < value {
                        right.remove(value)
                        self.root = .some(left, current, right)
                    } else {
                        left.remove(value)
                        self.root = .some(left,current,right)
                    }
                }
            }

            private func merge(_ treeB: BinaryTree<T>) -> BinaryTree<T> {
                if treeB.root == .none {
                    return self
                }

                switch self.root {
                case .none:
                    return treeB
                case let .some(left, current, right):
                    return BinaryTree(.some(left, current, right.merge(treeB)))
                }
            }
        }
        ```  

* **전체 구현**  
  위의 메소드들을 모두 합치고, 실제 사용할 수 있도록 인터페이스를 추가한 전체 구현은 다음과 같습니다.  

    ```swift  
    struct BinaryTree<T: Comparable>: Equatable {
        private indirect enum BinaryTreeNode<T: Comparable>: Equatable {
            case none
            case some(BinaryTree<T>, T, BinaryTree<T>)
        }

        private var root: BinaryTreeNode<T>

        init() {
            root = .none
        }

        private init(_ node: BinaryTreeNode<T>) {
            root = node
        }
    }

    extension BinaryTree {
        var list:[T] {
            switch root {
            case .none:
                return []
            case let .some(left, current, right):
                return left.list + [current] + right.list
            }
        }

        func find(_ value: T) -> Bool {
            switch root {
            case .none:
                return false
            case let .some(left, current, right):
                if current == value {
                    return true
                } else if current < value {
                    return right.find(value)
                } else {
                    return left.find(value)
                }
            }
        }

        mutating func insert(_ value: T) {
            switch root {
            case .none:
                self.root = .some(BinaryTree(), value, BinaryTree())
            case .some(var left, let current, var right):
                if current < value {
                    right.insert(value)
                } else {
                    left.insert(value)
                }

                self.root = .some(left, current, right)
            }
        }

        mutating func remove(_ value: T) {
            switch root {
            case .none:
                return
            case .some(var left, let current, var right):
                if current == value {
                    self = left.merge(right)
                } else if current < value {
                    right.remove(value)
                    self.root = .some(left, current, right)
                } else {
                    left.remove(value)
                    self.root = .some(left,current,right)
                }
            }
        }

        private func merge(_ treeB: BinaryTree<T>) -> BinaryTree<T> {
            if treeB.root == .none {
                return self
            }

            switch self.root {
            case .none:
                return treeB
            case let .some(left, current, right):
                return BinaryTree(.some(left, current, right.merge(treeB)))
            }
        }
    }
    ```  


* **BST의 효율성**  
  BST는 이상적인 경우, 삽입, 삭제, 탐색 모두 O(log n)의 시간이 소요됩니다. 하지만 트리가 균형잡히지 않고 한쪽으로 치우치게 되면(skewed) BST의 효율은 떨어지게 됩니다. 완전히 치우치게 된 경우는 연결리스트와 다를 바가 없어집니다. 이 경우 탐색, 삽입, 삭제 모두 O(n)이 됩니다. 이를 막기 위해서는 트리가 항상 균형된 상태를 유지하도록 해줘야 하는데, 이를 위한 알고리즘들은 일부 다른 포스트를 통해서 차차 알아보도록 하겠습니다.