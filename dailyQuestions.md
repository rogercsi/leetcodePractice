#### 2020-06-21

##### [94] 二叉树的中序遍历

递归实现

```java
public class TreeNode {
     int val;
     TreeNode left;
     TreeNode right;
     TreeNode(int x) { val = x; }
 }
class Solution {
    List<Integer> list = new ArrayList<>();
    public List<Integer> inorderTraversal(TreeNode root) {
        if (root==null) {
            return list;
        }
        if (root.left==null) {
            list.add(root.val);
        }else{
            inorderTraversal(root.left);
            list.add(root.val);
        }
        inorderTraversal(root.right);
        return list;
    }
}
```

#### 2020-06-24

##### [120]三角形最小路径和

首先想到的就是dfs

```java
class Solution {
    Integer min = Integer.MAX_VALUE;
    public int minimumTotal(List<List<Integer>> triangle) {
        Integer row = 0, column = 0, sum;
        if (triangle==null||triangle.get(0)==null){
            return 0;
        }
        sum = triangle.get(row).get(column);
        dfs(row,column,sum,triangle);
        return min;
    }
    void dfs(Integer row, Integer column, Integer sum, List<List<Integer>> triangle){
        if (row == triangle.size() - 1){
            if (sum < min){
                min = sum;
            }
        }
        else{
            dfs(row+1, column, sum + triangle.get(row+1).get(column), triangle);
            dfs(row+1, column+1, sum + triangle.get(row+1).get(column+1), triangle);
        }
    }
}
```

超时

存在`重复子问题`，`sum+ triangle.get(row+1).get(column)`被反复计算，可以选择缓存，用HashMap作缓存

也可以用分治

```java
class Solution {
    public int minimumTotal(List<List<Integer>> triangle) {
        return dfs(0,0,triangle);
    }
    Integer dfs(Integer row, Integer column, List<List<Integer>> triangle){
        if (row == triangle.size()){
            return 0;
        }
        return compare(dfs(row+1,column,triangle),dfs(row+1,column+1,triangle))+triangle.get(row).get(column);
    }
    Integer compare(Integer a, Integer b){
        if (a > b) return b;
        return a;
    }
}
```

同样超时

说明这两种方法都需要优化

二叉树的子问题没有交集，所以一般的二叉树用递归+剪枝或分治能解决

triangle存在重复情况，子问题有交集，可以用动态规划

// to do

#### 2020-06-25

// continue

*自底向上*

`从子问题出发，先初始化第一行，即先解决最小的子问题，通过递推公式依次向上解决问题`，一般动态规划问题常常用于求解最值，所以此时以求最小值为例，递推公式一般为

```java
memo[row][column] = min{memo[row - 1][column],memo[row - 1][column + 1]} + value[row][colum]
```

代码

```java
class Solution {
    public int minimumTotal(List<List<Integer>> triangle) {
        // 初始化
        Integer size = triangle.size(); //4
        List<List<Integer>> memo = new ArrayList<>();
        List<Integer> rowList = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            rowList.add(triangle.get(size-1).get(i));
        }
        memo.add(rowList);
        // 按照递推关系层层赋值
        for (int row = 1; row < size; row++) { 
            memo.add(new ArrayList<>());
            rowList = memo.get(row);
            for (int column = 0; column < size - row; column++) {
                rowList.add(compare(memo.get(row - 1).get(column),memo.get(row - 1).get(column + 1)) + triangle.get(size - 1 - row).get(column));
            }
        }
        // 返回最上一层
        return memo.get(size - 1).get(0);
    }
    Integer compare(Integer a, Integer b){
        if (a > b) return b;
        return a;
    }
}
```

由于用list作为容器存储memo，最终结果

> -  runtime beats 12.72 % of java submissions
> - Your memory usage beats 8.7 % of java submissions (39.8 MB)

说明还有可以进行优化的地方

首先，将存储memo的容器替换为数组类型`Integer[][] arr = new Integer[size][size];` 

> - Your runtime beats 31.8 % of java submissions
> - Your memory usage beats 8.7 % of java submissions (39.8 MB)

这应该和ArrayList的get方法效率相关?

*备忘录法*

当解决问题的时候，通过加缓存的方式，若遇到之前曾经遇到过的子问题，直接从缓存中抽取数据

**具体使用场景**

> 满足两个条件
>
> - 满足以下条件之一
>   - 求最大/最小值（Maximum/Minimum ）
>   - 求是否可行（Yes/No ）
>   - 求可行个数（Count(*) ）
> - 满足不能排序或者交换（Can not sort / swap ）

##### [128] 最长连续序列

直接存入HashMap，再依次遍历

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> set = new HashSet<>();
        Integer result = 0, current = 0;
        for (int i = 0; i < nums.length; i++) {
            set.add(nums[i]);
        }
        for (int i = Integer.MIN_VALUE; i < Integer.MAX_VALUE; i++) {
            if (set.contains(i)) {
                current++;
            }else{
                if (current > result) {
                    result = current;
                }
                current = 0;
            }
        }
        return result;
    }
}
```

直接超时。。

说明必须要通过区间的方式将范围缩小，且不能按照从Integer.MINVALUE->Integer.MAXVALUE的方式遍历

首先遍历nums，将其存入map，键是nums中的数，值是当前连续序列的个数，因此第一个插入的value为1；

之后由于存在连续序列，需要获取当前数的左右的已经存在的连续序列，将左边序列+右边序列+1（此数），并将左边序列最左边和右边序列最右边的键值对中含义为已经存在的连续序列的值进行更新

```java
public class Solution {
    public int longestConsecutive(int[] nums) {
        Integer size = nums.length;
        Map<Integer,Integer> map = new HashMap<>();
        Integer result = 0, current = 0;
        Integer left, right, value;
        Integer index = 0;
        while (index < size) {
            if (!map.containsKey(current = nums[index])) {
              // 当前数的左右的已经存在的连续序列
                left = map.get(current - 1);
                right = map.get(current + 1);
                if (left == null) {left = 0;}
                if (right == null) {right = 0;}
                value = left + right + 1;
                if (result < value) {
                    result = value;
                }
              // 连续序列的值进行更新
                map.put(current,value);
                map.put(current - left,value);
                map.put(current + right,value);
            }
            index++;
        }
        return result;
    }
}
```

> - Your runtime beats 15.94 % of java submissions
> - Your memory usage beats 8.33 % of java submissions (39.9 MB)

还是存在优化的方法