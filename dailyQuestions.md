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

#### 2020-06-26

##### 排序算法之简单排序

这一切源于**[287] 寻找重复数**，由于此题限制条件颇多，很难说能够直接想到满足条件的解法。一开始，大多数人能通过两次循环遍历比较查出重复数，算法步骤来看像极了选择排序，很明显时间复杂度是$\Omicron(n^2)$, 虽然不满足条件，但是引起了我对排序算法的关注，我想很有可能此题与排序算法有联系。

回家的高铁上我在想这道题，但是百思不得其解，之后的一天我觉得从排序算法入手，看看能否产生什么灵感之类的东西。

**备注**：以下排序算法只经过几个测试条件，未经过严格测试，可能存在边界条件考虑不足，但算法思想基本就这样

###### 冒泡排序

外层循环总是需要走完全程`arr.length`

```java
public static void bubbleSort(Integer[] arr){
        int temp;
        for (int i = 0; i < arr.length; i++) {
            for (int j = 0; j < arr.length - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }
    }
```

###### 插入排序

插入新值时需要将本该在后面的所有数都向后移动，所有现在写的这个算法可能有问题，看样子是$\Omicron(n^3)$的时间复杂度了

```java
public static void insertSort(Integer[] arr){
        int temp;
        for (int i = 0; i < arr.length; i++) {
            for (int j = 0; j < i; j++) {
                if (arr[i] < arr[j]) {
                    temp = arr[i];
                    moveTheRest(arr, j, i - 1);
                    arr[j] = temp;
                }
            }
        }
    }

    /**
     *
     * @param arr
     * @param start the beginning of the arr which need to be moved backward
     * @param end the end of the arr which need to be moved backward
     */
    private static void moveTheRest(Integer[] arr, int start, int end) {
        for (int i = end; i > start - 1; i--) {
            arr[i + 1] = arr[i];
        }
    }
```

###### 选择排序

```java
public static void selectSort(Integer[] arr){
        int temp;
        int index;
        for (int i = 0; i < arr.length; i++) {
            index = i;
            temp = Integer.MAX_VALUE;
            for (int j = i; j < arr.length; j++) {
                if (arr[j] < temp) {
                    temp = arr[j];
                    index = j;
                }
            }
            arr[index] = arr[i];
            arr[i] = temp;
        }
    }
```

###### 归并排序

比较典型的分治算法，采用二分，所谓分而治之，先通过divide函数将`arr` divide直到`left == right`也就是单个元素，之后通过merge函数将分散的数经过排序之后合起来

归并排序的时间复杂度是$nlogn$，空间复杂度是$n$，我想的是分治可能能够在**[287] 寻找重复数**中发挥作用

```java
		public static void mergeSort(Integer[] arr) {
        int left = 0, right = arr.length - 1;
        Integer[] temp = new Integer[arr.length];
        divide(arr, left, right, temp);
        for (Integer i:
            temp ) {
            System.out.print(i + " ");
        }
    }
    private static void divide(Integer[] arr, int left, int right, Integer[] temp) {
        if (left < right) {
            int mid = (right + left)/2;
            divide(arr, left, mid, temp);
            divide(arr, mid + 1, right, temp);
            merge(temp, arr, left, right, mid);
        }
    }
    private  static void merge(Integer[] temp, Integer[] arr, int left, int right, int mid) {
        // index1 is an index of whole new arr, index2 is an index of old right half part
        int i = left, j = mid + 1, k = 0;
        while (i < mid + 1 && j < right + 1) {
            temp[k++] = arr[i] > arr[j] ? arr[j++] : arr[i++];
        }
        while (j < right + 1) {
            temp[k++] = arr[j++];
        }
        while (i < mid + 1) {
            temp[k++] = arr[i++];
        }
        for (i = 0;i < k; i++){
            arr[left + i] = temp[i];
        }
    }
```

###### 快速排序

// to do



##### [287] 寻找重复数

#### 2020-06-27

##### [64] 最小路径和

*给定一个包含非负整数的 m x n 网格，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。*

一开始直接用递归，很明显会超时，必须对重复计算的位置进行缓存，所以首先想到用动态规划的自顶向下的备忘录法。需要开辟一个二位数组空间进行缓存

```java
public class Solution {
  // 多个函数需要调用此数组，需要定义为类变量
    int[][] arr;
    public int minPathSum(int[][] grid) {
      // 边界条件 为空
        if (grid.equals(null)) {
            return 0;
        }
        int row = grid.length - 1, column = grid[0].length - 1;
      // 开辟数组初始化
        arr = new int[row + 1][column + 1];
      // 边界条件 仅存在一行
        if (row == 0) {
            return rowDone(grid, column);
        }
      // 边界条件 仅存在一列
        if (column == 0) {
            return columnDone(grid, row);
        }
        return  Integer.min(minPath(grid, row, column - 1), minPath(grid, row - 1, column)) + grid[row][column];
    }
  // 递归求解当前节点的最短路径 状态函数为 len(row,column) = min{len((row - 1), column),len(row,(column - 1))} + value(row,column)
    private int minPath(int[][] grid, int row, int column) {
      // 若已经存在缓存,直接从缓存中拿
        if (arr[row][column] != 0) {
            return arr[row][column];
        }
      // base case 之到了原点
        if (row == 0 && column == 0) {
            arr[row][column] = grid[0][0];
            return grid[0][0];
        }
      // base case 之到了第一行
        if (row == 0) {
            arr[0][column] = rowDone(grid, column - 1) + grid[0][column];
            return arr[0][column];
        }
      // base case 之到了第一列
        if (column == 0) {
            arr[row][0] = columnDone(grid, row - 1) + grid[row][0];
            return arr[row][0];
        }
        if (row > 0 && column > 0) {
            arr[row][column] = Integer.min(minPath(grid, row, column - 1), minPath(grid, row - 1, column)) + grid[row][column];
            return arr[row][column];
        }
        return 0;
    }
  // 第一行的base case
    private int rowDone(int[][] grid, int column) {
        if (column == -1) {
            return 0;
        }
        if (arr[0][column] != 0) {
            return arr[0][column];
        }
        else  {
            arr[0][column] = grid[0][column] + rowDone(grid, column - 1);
            return arr[0][column];
        }
    }
  // 第一列的base case
    private int columnDone(int[][] grid, int row) {
        if (row == -1) {
            return 0;
        }
        if (arr[row][0] != 0) {
            return arr[row][0];
        }
        else  {
            arr[row][0] = grid[row][0] + columnDone(grid, row - 1);
            return arr[row][0];
        }
    }
}
```

- Your runtime beats 99.77 % of java submissions
- Your memory usage beats 36.36 % of java submissions (42.3 MB)

再用自底向上的方式做一遍

```java
public class Solution {
    public int minPathSum(int[][] grid) {
        int [][] arr;
        if (grid.equals(null)) {
            return 0;
        }
        arr = new int[grid.length][grid[0].length];
        arr[0][0] = grid[0][0];
        // 初始化行
        for (int i = 1; i < grid[0].length; i++) {
            arr[0][i] = grid[0][i] + arr[0][i - 1];
        }
        // 初始化列
        for (int i = 1; i < grid.length; i++) {
            arr[i][0] = grid[i][0] + arr[i - 1][0];
        }
        for (int i = 1; i < grid.length; i++) {
            for (int j = 1; j < grid[0].length; j++) {
                arr[i][j] = grid[i][j] + (arr[i - 1][j] < arr[i][j - 1] ? arr[i - 1][j] : arr[i][j - 1]);
            }
        }
        return arr[grid.length - 1][grid[0].length - 1];
    }
}
```

- Your runtime beats 32.71 % of java submissions
- Your memory usage beats 30.3 % of java submissions (42.4 MB)

==？运行时间居然比自顶向下的慢了很多，有人能解释一下吗？==

