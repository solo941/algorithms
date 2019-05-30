## 96. Unique Binary Search Trees

题目描述：给定n，求能构造多少种结构不同的二叉搜索树（BST）。二叉搜索树存储1到n的值。

```
Example:
Input: 3
Output: 5
Explanation:
Given n = 3, there are a total of 5 unique BST's:

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

解题思路：首先观察例子，发现这些BST的特点：以i为根节点，1到i-1在i的左侧，i+1到n在i的右侧。我们需要定义两个函数：

G(n)：长度为n的搜索树的种数。

F(i,n)：以i为根结点，长度为n的搜索树的种数。

我们轻易得到以下等式成立：

```
G(n) = F(1, n) + F(2, n) + ... + F(n, n). 
```

其中G(0) = G(1) = 1。

同时通过定义，i结点左边共i-1个结点，右边n-i个结点，因此，以下等式成立：

```
F(i, n) = G(i-1) * G(n-i)	1 <= i <= n 
```

综合两个式子，我们得到下面的代码：

```java
public int numTrees(int n) {
    int [] G = new int[n+1];
    G[0] = G[1] = 1;
    
    for(int i=2; i<=n; ++i) {
    	for(int j=1; j<=i; ++j) {
    		G[i] += G[j-1] * G[i-j];
    	}
    }

    return G[n];
}
```

这是一道非典型的dp问题，G[i]与G[0] ...G[i-1]相关，转移方程需要根据BST的特点给出。

## 95. Unique Binary Search Trees II

95题与96题相似，所不同的是，95题需要以列表形式返回所有的树。由于

```
G(n) = F(1, n) + F(2, n) + ... + F(n, n). 
```

长度为n的BST构成与1...n-1的BST相关，所以使用数组arr保留长度1...n的BST构成。与96类似，使用G(n)表示长度为n的所有搜索树，arr[n] =G[n]。内部循环存储以i为根，长度为n的所有BST(F(i, n))。

二叉树构成如下：

```
	F（i,n)
	/	 \
 G(i-1)  G(n-i)
```

这里需要注意，在96题，我们只需要知道长度为n的搜索树的种数，因此，右子树为[i+1,...n]子数组构成的BST个数与G（n-i）个数相同；但求二叉树的构成时，右子树每个结点value需要增加i，这里使用clone函数递归完成操作，代码如下：

```java
class Solution {
    public List<TreeNode> generateTrees(int n) {
	
	List<TreeNode>[] res = new List[n+1];
	res[0] = new ArrayList();
	if(n == 0) return res[0];
	res[0].add(null);
	res[1] = new ArrayList();
	res[1].add(new TreeNode(1));
	for(int i=2;i<=n;i++){
	    res[i] = new ArrayList();
		for(int j=1;j<=i;j++){
			for(TreeNode nodeL: res[j-1]){
				for(TreeNode nodeR: res[i-j]){
					TreeNode node = new TreeNode(j);
					node.left = nodeL;
					node.right = clone(nodeR, j);
					res[i].add(node);
				}
			}
		}
	}
	return res[n];
}


    private TreeNode clone(TreeNode node, int offset){
        if(node == null) return null;
        TreeNode newNode = new TreeNode(node.val + offset);
        newNode.left = clone(node.left, offset);
        newNode.right = clone(node.right, offset);
        return newNode;
    }
}
```

