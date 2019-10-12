---
layout: post
title: TableView의 변화를 처리하는 방법(1) - Batch Update
comments: true
tags: [iOS 프로그래밍,Swift,Apple]
category: [iOS]
---  

TableView의 데이터가 변했을 때, 우리는 종종 다음 메소드를 호출하고는 합니다.

```swift
tableView.reloadData()
```  

이 메소드는 테이블 뷰 전체를 완전히 처음부터 구성하기 때문에 비용이 큽니다. 하지만 보통 TableView에 일어나는 변화는 이 정도로 대규모로 일어나지 않습니다. 그렇기 때문에 TableView는 일부분만을 변화시키는 방법을 제공합니다. 이번 포스트에서는 이러한 방법들 중 batch Update 방법에 대해서 알아보겠습니다.  

> 이 포스트는 다음 가이드를 참고하여 작성되었습니다.  
> [Batch Insertion, Deletion and Reloading of Rows and Sections](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/TableView_iPhone/ManageInsertDeleteRow/ManageInsertDeleteRow.html#//apple_ref/doc/uid/TP40007451-CH10-SW9)  

* **tableView를 변경하는 메소드들**  
    tableView에는 다음과 같이 셀 혹은 섹션을 추가, 삭제, 리로드하는 메소드를 제공합니다.

    ```swift
    // 1개 이상의 Row를 지정된 indexPath에 추가한다.
    func insertRows(at indexPaths: [IndexPath], 
           with animation: UITableView.RowAnimation)

    // 1개 이상의 Section을 지정된 index에 추가한다.
    func insertSections(_ sections: IndexSet, 
               with animation: UITableView.RowAnimation)

    // 1개 이상의 Row를 지정된 indexPath에서 제거한다.
    func deleteRows(at indexPaths: [IndexPath], 
           with animation: UITableView.RowAnimation)

    // 1개 이상의 Section을 지정된 index에서 제거한다.
    func deleteSections(_ sections: IndexSet, 
               with animation: UITableView.RowAnimation)

    // 1개 이상의 Row를 리로딩한다.
    func reloadRows(at indexPaths: [IndexPath], 
           with animation: UITableView.RowAnimation)

    // 1개 이상의 Section을 리로딩 한다.
    func reloadSections(_ sections: IndexSet, 
               with animation: UITableView.RowAnimation)
    ```
    이 메소드들을 호출할 경우, 테이블뷰가 애니매이션을 동반하며 변경되며 변경된 뷰의 상태에 맞게 delegate와 datasource에 쿼리가 들어가게 됩니다. 이 메소드들은 단순히 뷰를 변경해주는 메소드들로, 실제 데이터에는 영향을 주지 않습니다. 하지만 데이터를 변경하지 않고 뷰만 변경할 경우, delegate와 datasource에서는 이 뷰의 변경을 알 수 없기 때문에 오류가 발생하며 앱이 크래시가 나게 됩니다. 따라서 위 메소드들을 호출해주기 전에 반드시 데이터를 먼저 변경해 주어야 합니다.

* **begin/endUpdates()**  
    tableView를 변경하는 메소드들을 여러번 호출해야 하는 경우가 있습니다. 예를 들어 특정 셀을 지우고나서, 새로운 셀을 추가하는 작업을 한다던지 하는 경우죠. 그렇게 되면 위 메소드들은 각각이 호출될 때 마다 delegate와 datasource에 쿼리가 들어가게 됩니다. 이렇게 되면 불필요한 쿼리가 늘어나고, 데이터 정합성을 맞추기도 어렵습니다. 또 애니매이션이 자연스럽지 못하게 되는 문제도 있습니다. 그렇기 때문에 이들을 트랜잭션처럼 취급하는 방법 역시 제공되고 있습니다.

    tableView에서는 다음 두가지 메소드를 제공합니다.
    ```swift
    func beginUpdates()
    func endUpdates()
    ```  

    이 두 메소드는 애니매이션 블록을 만들어주는 메소드로, tableView의 insert, delete, reload 등의 작업을 한번의 애니메이션으로 동시에 처리할 수 있도록 해줍니다. 할 일은  메소드들을 호출할 때, 앞 뒤로 beginUpdates()와 endUpdates()를 추가해주는 것입니다. 저 안에서 테이블 뷰 변경 메소드들을 호출할 경우, 쿼리가 들어가는 시점은 endUpdates()이후가 됩니다. 만약 두 메소드 사이에 아무것도 호출하지 않는다 하더라도, 쿼리는 무조건 들어가게 됩니다. 따라서 delegate 메소드를 호출해서 셀 높이를 변경한다던지 하는 것이 가능합니다.

    또 테이블 뷰의 요소들을 직접 얻어와서 애니매이션을 적용해주는 경우가 있는데(select), 이 경우에는 쿼리는 들어가지 않지만 애니메이션을 수반할 수 있다는 점에서 beginUpdate(), endUpdate() 사이에 넣어주면 됩니다.

    ```swift
    tableView.beginUpdates()
    // view 변경 코드들...
    tableView.endUpdates()
    ```

    iOS 11부터는 이를 대신해서 다음과 같은 메소드가 추가 되었습니다. (기존 함수가 아직 deprecated 되지는 않았습니다.)

    ```swift
    func performBatchUpdates(_ updates: (() -> Void)?, completion: ((Bool) -> Void)? = nil)
    ```  

    이는 애니매이션 블록을 지정하는 방법을 클로저 형태로 바꿔주고, 애니매이션의 성공 여부에 따른 동작을 지정해 줄 수 있는 클로저 블록을 추가해 준 형태입니다. 

* **애니매이션 블록에서의 작업 실행 순서**  
    beginUpdated()/endUpdates() 혹은 performBatchUpdates()로 만든 애니매이션 블록에서 테이블 뷰 변경 메소드를 실행 할 때는 선언된 순서대로 함수들이 실행되지 않습니다. 대신 다음과 같은 우선순위에 따라 실행이 됩니다.  

    1. delete, reload계열 메소드
    2. insert, select계열 메소드  

    이렇게 순서가 정해져 있기 때문에, indexPath를 사용할 때는 그것을 반드시 감안하고 사용해야 합니다.


---  

