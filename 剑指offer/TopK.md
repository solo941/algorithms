## 最小的K个数

题目描述：输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4。

解题思路：最简单的思路是排序，这种思路的时间复杂度O（nlogn）。但如果使用基于pivot的单路快排，不需要考虑pivot前后范围各自的元素排列顺序，只要找到第数组中k-1大的元素即可，此时时间复杂度为O(n)。

```java
public ArrayList<Integer> GetLeastNumbers_Solution(int[] nums, int k) {
    ArrayList<Integer> ret = new ArrayList<>();
    if (k > nums.length || k <= 0)
        return ret;
    findKthSmallest(nums, k - 1);
    /* findKthSmallest 会改变数组，使得前 k 个数都是最小的 k 个数 */
    for (int i = 0; i < k; i++)
        ret.add(nums[i]);
    return ret;
}

public void findKthSmallest(int[] nums, int k) {
    int l = 0, h = nums.length - 1;
    while (l < h) {
        int j = partition(nums, l, h);
        if (j == k)
            break;
        if (j > k)
            h = j - 1;
        else
            l = j + 1;
    }
}

private int partition(int[] nums, int l, int h) {
    int p = nums[l];     /* 切分元素 */
    int i = l, j = h + 1;
    while (true) {
        while (i != h && nums[++i] < p) ;
        while (j != l && nums[--j] > p) ;
        if (i >= j)
            break;
        swap(nums, i, j);
    }
    swap(nums, l, j);
    return j;
}

private void swap(int[] nums, int i, int j) {
    int t = nums[i];
    nums[i] = nums[j];
    nums[j] = t;
}
```

但这种方法存在局限性，原地操作，会改变输入数组。在海量数据场景，一次加载所有数据到内存是不可能的。这时候从辅助空间（磁盘）中读数，需要使用容器保持最小的k个数，并与读的数进行比较。这个容器我们可以通过堆来实现，java中常用的大根堆与小根堆的实现方式是`PriorityQueue`。本题使用大根堆，每次添加至容器时，最大的元素出队列。

```java
public ArrayList<Integer> GetLeastNumbers_Solution(int[] nums, int k) {
    if (k > nums.length || k <= 0)
        return new ArrayList<>();
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>((o1, o2) -> o2 - o1);
    for (int num : nums) {
        maxHeap.add(num);
        if (maxHeap.size() > k)
            maxHeap.poll();
    }
    return new ArrayList<>(maxHeap);
}
```

堆是基于二叉树的实现，因此时间复杂度为`O(nlogk)`。

## 数据流中的中位数

题目描述：

如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。我们使用Insert()方法读取数据流，使用GetMedian()方法获取当前读取数据的中位数。

解题思路：

《剑指offer》中，给出了4中主要的解题思路。第一种和第二种是最好想的：数组和链表。数组插入时间复杂度O(1)，查找中位数使用快排pivot的时间复杂度O(n)；链表在插入时进行排序，因此时间复杂度O(n)，同时定义两个指针指向中间元素（奇数个指针指向位置相同，偶数相邻），查找时间复杂度O(1)。第三种使用树结构，使用二叉搜索树，插入平均时间复杂度O(logn)，查找平均O(logn)，但树如果倾斜严重，最差时间复杂度是O(n)。AVL树实现复杂，这就引出了第四种方法，双堆（大根和小根）。我们只需要分别保存中位数的两边最大值和最小值就可以得到中位数的取值。

| 数据结构 | 插入平均时间复杂度 | 查找平均时间复杂度 |
| -------- | ------------------ | ------------------ |
| 数组     | O(1)               | O(n)               |
| 链表     | O(n)               | O(1)               |
| 树       | O(logn)            | O(1)               |
| 双堆     | O(logn)            | O(1)               |

```java
public class Solution {
    private PriorityQueue<Integer> left = new PriorityQueue<>((o1, o2) -> o2 - o1);
    private PriorityQueue<Integer> right = new PriorityQueue<>();
    private int n = 0;
    public void Insert(Integer num) {
        if(n % 2 == 0){
            left.add(num);
            right.add(left.poll());
        }
        else{
            right.add(num);
            left.add(right.poll());
        }
        n++;
    }

    public Double GetMedian() {
        if(n % 2 == 0)
            return (double)(right.peek() + left.peek()) / 2;
        else
            return (double) right.peek();
    }
}
```

插入时要判断存在的元素个数，如果偶数，默认插入最小堆。但直接插入不能保证堆顶元素一定大于等于最大堆堆顶元素，因此过程为插入最大堆->移动堆顶元素到最小堆。

