# leetcode for 號中

###### tags: `interview`




```
If input array is sorted then
    - Binary search
    - Two pointers

If asked for all permutations/subsets then
    - Backtracking

If given a tree then
    - DFS
    - BFS

If given a graph then
    - DFS
    - BFS

If given a linked list then
    - Two pointers

If recursion is banned then
    - Stack

If must solve in-place then
    - Swap corresponding values
    - Store one or more different values in the same pointer

If asked for maximum/minumum subarray/subset/options then
    - Dynamic programming

If asked for top/least K items then
    - Heap

If asked for common strings then
    - Map
    - Trie

Else
    - Map/Set for O(1) time & O(n) space
    - Sort input for O(nlogn) time and O(1) space

```




sorted -> 可以用兩個指標 一個頭一個尾巴 l++ r-- 逼近目標
duplicate -> sort, 再處理


https://gist.github.com/johnhckuo/cd94243b858d597cd401eb33ff846a1f

note:
js copy array:
let new_arr = [...old_arr]
let new_arr = old_arr.slice()

# Array

## 53. Maximux subarray
Kadane's algorithm

`
The problem to find sum or maximum or minimum in an entire array or in a fixed-size sliding window could be solved by the dynamic programming (DP) approach in linear time.
`



## 169. Majority Element
Boyer-Moore Voting Algorithm

```
Object.keys(dictionary).forEach(function(key) {
    console.log(key, dictionary[key]);
});
```



## 122 Best Time to Buy and Sell Stock II
畫圖, recursive

## 26  Remove Duplicates from Sorted Array
two pointer來操作array

## 118 pascak triangle

2D Array list 操作
reference: https://stackoverflow.com/questions/16956720/how-to-create-an-2d-arraylist-in-java
```
class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> lists = new ArrayList<List<Integer>>(numRows);  
        for (int i = 0 ; i < numRows ; i++){
            lists.add(new ArrayList<Integer>());
        }
        
        
        for (int i = 0 ; i < numRows; i++){
            for (int j = 0; j < i+1 ; j++){
            
                
                if ( (j == 0) || (j == i) ){
                    lists.get(i).add(1);
                }else{
                    int previous = lists.get(i-1).get(j-1);
                    int previous2 = lists.get(i-1).get(j);
                    lists.get(i).add(previous + previous2);
                }
            }
        }
        return lists;
    }
}
```



## 27 remove element
找欲移除的(或是不欲移除的)，記錄他們的indices，放在arraylist中,然後在loop他們，並和原來array中的元素對調

Arraylist.size()


## 448 Find All Numbers Disappeared in an Array
如果陣列中的值是遞增的，並且可以間接的代表陣列的索引，但便可以利用這一點來找出陣列中遺失/多餘 的元素, 並以Ｏ（Ｎ）的時間和O(1)的空間複雜度(in-place detect)

運用的手法就是,讀到array中某個元素的值, 再跳到他value所map到的index, 標誌該index的value（如把正值調成負值）,第二次iterate時若有哪個元素並非負數，那就表示這個value所指向的index沒有被visit過



## 11 Container With Most Water

再次運用兩個指標(head, rear pointer)和一些小技巧, 可以有效把O(n2)轉化為O(n)




## 167. Two Sum II - Input array is sorted
當陣列是經過排序之後, 若要找總合是否等於其中兩數, 直覺會是使用hashmap來達到O(n), 然而這樣空間複雜度就會等於O(n), 除非使用兩個pointer, 一個指向頭, 一個指向尾, 兩者相加若小於目標, 則頭＋1, 反之則尾-1, **只適用於已排序過的情境下**

https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/discuss/51239/Share-my-java-AC-solution.


## 15. 3 sum

**It's easier to deal with duplicates if the array is sorted**

note: js hashmap lookup key exists: `map.hasOwnProperty("1,3")`

A Set is a generic set of values with no duplicate elements.

A TreeSet is a set where the elements are sorted.

A HashSet is a set where the elements are not sorted or ordered. It is faster than a TreeSet. The HashSet is an implementation of a Set.

hashmap vs hashset
https://stackoverflow.com/questions/2773824/difference-between-hashset-and-hashmap


js hashset:

```
var set = new Set();
set.add(1);
set.add(2);

set.has(1)    //
set.delete(1)

for (let item of set.values()) console.log(item)


```


## 219. Contains Duplicate II

運用hash set來當作sliding window




## 56. Merge Intervals
要**先sort過後** 比較方便找互相重疊的interval 因為就在旁邊

一個陣列中移除一個元素
1. 快速作法 O(1) 但會打亂順序
    將元素和最後一個元素交換
    移除最後一個元素(直接delete或copy n-1長度至另一個陣列)
2. 較慢但會維持原順序 O(n)
    將 0~i+1的陣列貼到0~i的部分
    copy(a[i:], a[i+1:])
    然後把最後一個元素移除
    a = a[:len(a)-1]
    
## 238. Product of Array Except Self
 
**事先計算**從後面數過來的乘積
 在算前面數過來的
 
 然後一次iteration將兩個部分合併即可得到解
 
## 560. Subarray Sum Equals K
for loop條件不要想太多 先設簡單一些 不然會自己搞得很複雜

-----1------|----2----
-----------3----------

-> 3 - 2 = 1
-> sum[j] - k = sum[i]

可使用hashmap來處理達到O(n)



### 119 Pascal's Triangle II

dynamic programming 
或運用前一排 即可算下一排
不需要每個都存
memory locality!