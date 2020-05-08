# top k 问题

描述：

在海量的数据中，取出最大的K个元素。

拿到这个问题会有下面几个思考方向：

1、元素排序，例如可以使用快速排序法

2、利用分布式的思想处理海量数组，每个服务器计算自己的top k，然后再合并几个top k，得到最大的K个元素

3、利用一个大小为K的堆来解决，堆顶元素为最小值



在堆排序中，我们实现了一个通过大堆顶来从小到大排列数组 元素的过程，top k 中我们需要实现一个小堆顶，如下：

```go
func main() {
     //构建一个top长度的数组
     //将数组构建成小堆顶
     for {
          //将下一个数值和nums[0]比较，如果大于nums[0], 替换nums[0]的值
          //调用heapfiy()
     }
  
  //调用heapSort()将数组按照从大到小的顺序返回
}

func heapSort(nums []int)  {
     for i := len(nums)-1; i >=0; i++ {
          swap(nums, 0, i)
          heapify(nums, i, 0)
     }
}


func buildHeap(nums []int) {
     l := len(nums)
     lastNode := (l-1)/2
     for i := lastNode; i >= 0; i-- {
          heapify(nums, l-1, i)
     }
}

//小堆顶
func heapify(nums []int, n, i int)  {
     left := 2*i + 1
     right := 2*i + 2
     min := i
     if left < n && nums[left] < nums[min] {
          min = left
     }
     if right < n && nums[right] < nums[min] {
          min = right
     }

     if min != i {
          swap(nums, i, min)
          heapify(nums, n, min)
     }
     return
}

func swap(nums []int, i, j int) {
     nums[i], nums[j] = nums[j], nums[i]
}
```



