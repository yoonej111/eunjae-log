# 938. Range Sum of BST

Given the root node of a binary search tree and two integers low and high, return the sum of values of all nodes with a value in the inclusive range [low, high].

[Leetcode](https://leetcode.com/problems/range-sum-of-bst/)

#### 💻Soultion

```javascript
var rangeSumBST = function(root, low, high) {
    let sum = 0;
    // null 노드일 경우 0 반환
    if(!root) return 0;
    // 현재 노드의 값이 주어진 범위에 있는 경우, sum에 추가
    if(root.val >= low && root.val <= high) sum += root.val;
    // 왼쪽 자식이 존재하면, 왼쪽 서브트리 탐색
    if(root.left) sum += rangeSumBST(root.left, low, high);
    // 오른쪽 자식이 존재하면, 오른쪽 서브트리 탐색
    if(root.right) sum += rangeSumBST(root.right, low, high);

    return sum;
};
```


#### 💡TIP

##### 이진 트리[Binary Tree] VS 이진 탐색 트리[Binary Search Tree]

**이진 트리** 
이진 트리는 각 노드가 최대 두 개의 자식을 가질 수 있는 트리 구조로, 주요 종류 로는 포화 이진트리가, 완전 이진트리, 균형 이진트리가 있다.

* **포화 이진트리(Full Binary Tree):** 모든 노드가 0개 또는 2개의 자식을 가지고 있는 트리로, 모든 레벨이 꽉 차 있는 트리.
* **완전 이진트리(Complete Binary Tree):** 마지막 레벨을 제외한 모든 레벨이 완전히 채워져 있고, 마지막 레벨의 노드들은 왼쪽부터 차례대로 채워지는 트리.
* **균형 이진트리(Balanced Binary Tree):** 왼쪽 서브트리와 오른쪽 서브트리의 높이 차이가 최대 1인 트리


**이진 탐색 트리**
이진 탐색 트리는 이진 트리의 한 종류로, 데이터를 효율적으로 탐색하고 관리하기 위해 고안된 자료구조로 아래 조건을 만족해야 한다.
```
1. 왼쪽 서브트리의 모든 노드 값은 현재 노드 값보다 작다.
2. 오른쪽 서브트리의 모든 노드 값은 현재 노드 값보다 크다.
3. 이 조건은 모든 노드에 재귀적으로 적용된다.
```
이진 탐색 트리는 탐색, 삽입, 삭제 등의 연산을 평균적으로 O(log n) 시간 안에 수행할 수 있지만, 최악의 경우 트리가 한 쪽으로 치우치게 되면 O(n)의 시간 복잡도를 가질 수 있다.


즉, **이진 트리**는 노드가 최대 두 개의 자식을 가질 수 있는 트리로 다양한 형태로 존재할 수 있고, **이진 탐색 트리**는 특정 조건을 만족하는 이진 트리로 효율적인 탐색과 정렬된 데이터 유직에 적합하다.