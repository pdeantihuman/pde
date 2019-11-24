# 算法题

## LC33

```go
func bsearch(nums []int, v int, left int, right int) int {
	if left > right {
		return -1
	}
	mid := (left + right) >> 1
	//fmt.Printf("left:%d right:%d mid:%d v:%d\n", left, right, mid, nums[mid])
	if nums[mid] == v {
		return mid
	} else if nums[mid] < v {
		return bsearch(nums, v, mid+1, right)
	} else {
		return bsearch(nums, v, left, mid-1)
	}
}

func search(nums []int, target int) int {
	count := len(nums)
	turnIndex := findTurnIndex(nums)
	//fmt.Printf("count:%d\n", count)
	//fmt.Printf("turnIndex:%d\n", turnIndex)
	result := -1
	if turnIndex == 0 {
		result = bsearch(nums, target, 0, count-1)
	} else {
		//数组分为两段有序数组,左端的数组比右端的数组值要大
		if target < nums[0] {
			//比左端最小还要小,右端查找
			result = bsearch(nums, target, turnIndex, count-1)
		} else if target > nums[0] {
			//比左端最小大,左端查找
			result = bsearch(nums, target, 0, turnIndex-1)
		} else {
			result = 0
		}
	}

	return result
}

func findTurnIndex(nums []int) int {
	turnIndex := 0
	count := len(nums)
	for i := 1; i < count; i++ {
		if nums[i] < nums[i-1] {
			turnIndex = i
		}
	}
	return turnIndex
}
```

## LC93

```go
func restoreIpAddresses(s string) (ans []string) {
	backtrace(nil, s, &ans)
	return
}

func backtrace(stats []string, s string, ans *[]string) {
	// append
	if len(stats) == 4 && len(s) == 0 {
		*ans = append(*ans, strings.Join(stats, "."))
	}
	if len(stats) >= 4 {
		return
	}

	for i := 3; i >= 0; i-- {
		// 判断数字够不够, 是不是合法的, 能不能继续
		if len(s) < i {
			continue
		}
		cur := s[:i]
		if !valid(cur) {
			continue
		}
        
		left := s[i:]
		minNeed := 3 - len(stats)
        // 通过知识剪枝
		if len(left) > 3*minNeed {
			return
		}
        // 剩下的不够, 不需要求解
		if minNeed > len(left) {
			continue
		}

		stats = append(stats, cur)
		backtrace(stats, left, ans)
		stats = stats[:len(stats)-1]
	}
}

func valid(s string) bool {
	if len(s) > 1 && s[0] == '0' {
		return false
	}
	i, err := strconv.ParseInt(s, 10, 64)
	if err != nil {
		return false
	}
	if i < 0 || i > 255 {
		return false
	}
	return true
}
```



## LC105

```go
// 由前序遍历preorder的第一个节点即为根节点
// 因为树中没有重复元素，所以由中序遍历以根节点左右两边分别是左右子树
// 结合前序遍历，知道左右子树分别的前序遍历和中序遍历，递归调用即可

// 在前序遍历知道根节点之后，要在中序遍历中找到根节点，避免每次都做线性查找
// 可以使用哈希表来存储中序遍历中每个节点对应的下标
// 使用辅助哈希表和递归过程的栈空间，所以Time:(n),n为节点数量 Space:O(n)
func buildTree(preorder []int, inorder []int) *TreeNode {
    inPos := make(map[int]int) // 辅助hash表存储中序遍历节点的下标
    for i := 0; i < len(inorder); i++ {
        inPos[inorder[i]] = i // 中序遍历元素作为key下标作为value存储
    }
    return buildTreeHelper(preorder, 0, len(preorder)-1, 0, inPos)
}

// pre前序遍历序列，preStart开始下标，preEnd结束下标，中序遍历序列inStart开始下标，和hash表
func buildTreeHelper(pre []int, preStart, preEnd, inStart int, inPos map[int]int) *TreeNode {
    if preStart > preEnd { // 如果前序遍历序列的开始下标已经大于结束下标
        return nil // 说明本次构建的二叉树中已经没有节点，返回空树
    }
    // 否则拿出前序遍历序列开始的元素，构建一个树节点，即为当前树的根节点
    root := &TreeNode{Val: pre[preStart]}
    // 在hash表中查到当前根节点在中序遍历中的下标
    rootIdx := inPos[pre[preStart]]
    // 减去本次中序遍历的开始下标，即为左子树的节点个数
    leftLen := rootIdx - inStart
    // 使用节点个数在前序遍历中确定左右子树的边界
    // 递归的调用自己分别构建左右子树
    root.Left = buildTreeHelper(pre, preStart+1, preStart+leftLen, inStart, inPos)
    root.Right = buildTreeHelper(pre, preStart+leftLen+1, preEnd, rootIdx+1, inPos)
    return root // 最后返回这棵树即可
}
```

## LC209

```go
// 滑动窗口 时间复杂度: O(n) 空间复杂度: O(1)
func minSubArrayLen4(s int, nums []int) int {
  l, r := 0, -1        //nums[l...r]为我们的滑动窗口,如果[0,0]就包含了第一个元素,初始不包含任何元素
  sum := 0             // 合初始设置为0
  res := len(nums) + 1 // 最短数组长度设置为整个数组长度+1,设置为最大值

  for l < len(nums) { // 窗口的左边界在数组范围内,则循环继续
    if r+1 < len(nums) && sum < s {
      r++            // 窗口右边界扩大
      sum += nums[r] // 计算sum值
    } else { // r已经到头,或者sum>=s
      sum -= nums[l]
      l++ // 窗口左边界缩小
    }
    if sum >= s {
      res = min(res, r-l+1)
    }
  }
  if res == len(nums)+1 {
    return 0
  }
  return res
}

func min(a, b int) int {
  if a < b {
    return a
  }
  return b
}
```



## LC106

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func buildTree(inorder []int, postorder []int) *TreeNode {
    postorderLength := len(postorder)
    if postorderLength == 0 {
        return nil
    }

    rootVal := postorder[postorderLength - 1]

    // 找出 root 在中序遍历中的位置
    index := findIndex(inorder, rootVal)
    root := new(TreeNode)
    root.Val = rootVal
    root.Left = buildTree(inorder[:index], postorder[:index])
    root.Right = buildTree(inorder[index + 1:], postorder[index:postorderLength - 1])

    return root
}

func findIndex(array []int, num int) int {
    for i, a := range array {
        if a == num {
            return i
        }
    }
    return 0
}
```

## LC215

```go
// Using Min Heap to store the k largest elements
// Time Complexity: O(nlogk) Space Complexity: O(k)
// https://github.com/aQuaYi/LeetCode-in-Go/blob/master/Algorithms/0215.kth-largest-element-in-an-array/kth-largest-element-in-an-array.go
func findKthLargest(nums []int, k int) int {
  temp := highHeap(nums)
  h := &temp
  heap.Init(h)
  if k == 1 {
    return (*h)[0]
  }
  for i := 1; i < k; i++ {
    heap.Remove(h, 0)
  }
  return (*h)[0]
}

// highHeap 实现了Heap 的接口
type highHeap []int

func (h highHeap) Len() int            { return len(h) }
func (h highHeap) Less(i, j int) bool  { return h[i] > h[j] } // h[i] > h[j] 使得 h[0] == max(h...)
func (h highHeap) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
func (h *highHeap) Push(x interface{}) { *h = append(*h, x.(int)) } // Push 使用 *h，是因为增加了 h 的长度
func (h *highHeap) Pop() interface{} { // Pop 使用 *h，是因为减短了 h 的长度
  res := (*h)[len(*h)-1]
  *h = (*h)[:len(*h)-1]
  return res
}
```

