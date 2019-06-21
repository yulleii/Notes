# 3.数组中重复的数字

## 题目描述

在一个长度为 n 的数组里的所有数字都在 0 到 n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字是重复的，也不知道每个数字重复几次。请找出数组中任意一个重复的数字。

```
Input:
{2, 3, 1, 0, 2, 5}

Output:
2
```

## 解题思路

思路一：

要求时间复杂度 O(N)，空间复杂度 O(1)。因此不能使用排序的方法，也不能使用额外的标记数组。

对于这种数组元素在 [0, n-1] 范围内的问题，可以将值为 i 的元素调整到第 i 个位置上进行求解。

以 (2, 3, 1, 0, 2, 5) 为例，遍历到位置 4 时，该位置上的数为 2，但是第 2 个位置上已经有一个 2 的值了，因此可以知道 2 重复：

```java
public boolean duplicate(int[] nums, int length, int[] duplication) {
    if (nums == null || length <= 0)
        return false;
    for (int i = 0; i < length; i++) {
        while (nums[i] != i) {
            if (nums[i] == nums[nums[i]]) {
                duplication[0] = nums[i];
                return true;
            }
            swap(nums, i, nums[i]);
        }
    }
    return false;
}

private void swap(int[] nums, int i, int j) {
    int t = nums[i];
    nums[i] = nums[j];
    nums[j] = t;
}
```

~~~java
public boolean duplicate(int[] nums, int length, int[] duplication) {
    if(nums==null || length<=0){
        return false;
    }
    boolean[] flags=new boolean[length];
}
~~~

思路二：快慢指针

# 4.二维数组的查找

## 题目描述

给定一个二维数组，其每一行从左到右递增排序，从上到下也是递增排序。给定一个数，判断这个数是否在该二维数组中。

```
Consider the following matrix:
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]

Given target = 5, return true.
Given target = 20, return false.
```

## 解题思路

要求时间复杂度 O(M + N)，空间复杂度 O(1)。其中 M 为行数，N 为 列数。

该二维数组中的一个数，小于它的数一定在其左边，大于它的数一定在其下边。因此，从右上角开始查找，就可以根据 target 和当前元素的大小关系来缩小查找区间，当前元素的查找区间为左下角的所有元素。

```java
public boolean Find(int target, int[][] matrix) {
    if (matrix == null || matrix.length == 0 || matrix[0].length == 0)
        return false;
    int rows = matrix.length, cols = matrix[0].length;
    int r = 0, c = cols - 1; // 从右上角开始
    while (r <= rows - 1 && c >= 0) {
        if (target == matrix[r][c])
            return true;
        else if (target > matrix[r][c])
            r++;
        else
            c--;
    }
    return false;
}
```

# 5.替换空格

## 题目描述

将一个字符串中的空格替换成 "%20"。

```
Input:
"A B"

Output:
"A%20B"
```

## 解题思路

低效的思路：通过遍历字符串数组来替换对应位置的值，之后的值整理向后偏移。整体时间O(n2)

高效思路：首先计算出替换后的长度，然后通过两个指针分别指向原数组的尾部和替换后数组的尾部，从后向前遍历遇到空格，替换内容否则填充字符。

~~~java
public String replaceSpace(StringBuffer str) {
        if(str==null)
            return null;
        if(str.length()<=0)
            return str.toString();
    	int P1=str.length()-1;
        for(int i=0;i<=P1;i++){
            if(str.charAt(i)==' ')
                str.append("  ");
        }
        int P2=str.length()-1;
        while(P1>=0 && P2>P1){
            char temp=str.charAt(P1--);
            if(temp==' '){
                str.setCharAt(P2--,'0');
                str.setCharAt(P2--,'2');
                str.setCharAt(P2--,'%');
            }
            else{
                str.setCharAt(P2--,temp);
            }
        }
        return str.toString();
    }
~~~

举一反三：合并两个数组

# 6.从尾到头打印链表

在面试中，如果我们打算**修改输入的数组**，最好先问面试官是不是允许修改

## 题目描述

从尾到头反过来打印出每个结点的值。

~~~java
 public class ListNode {
      int val;
       ListNode next = null;

      ListNode(int val) {
           this.val = val;
       }
    }
~~~

## 解题思路

### 使用递归

要逆序打印链表 1->2->3（3,2,1)，可以先逆序打印链表 2->3(3,2)，最后再打印第一个节点 1。而链表 2->3 可以看成一个新的链表，要逆序打印该链表可以继续使用求解函数，也就是在求解函数中调用自己，这就是递归函数。

~~~java
public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
    ArrayList<Integer>array=new ArrayList<>();
    if(listNode!=null){
        array.addAll(printListFromTailToHead(listNode.next));
        array.add(listNode.val);
    }
    return array;
}
~~~

### 使用头插法

使用头插法可以得到一个逆序的链表。

头结点和第一个节点的区别：

- 头结点是在头插法中使用的一个额外节点，这个节点不存储值；
- 第一个节点就是链表的第一个真正存储值的节点

~~~java
public ArrayList<Integer> printListFromTailToHead(ListNode listNode){
    ListNode head=new ListNode(-1);
    while(listNode!=null){
        ListNode memo=listNode.next;
        listNode.next=head.next;
        head.next=listNode;
      	listNode=memo; 
        }
	ArrayList<Integer>array=new ArrayList<>();
	head=head.next;
	while(head!=null){
        array.add(head.val);
        head=head.next;
    }
	return array;
}
~~~

### 使用栈

栈具有后进先出的特点，在遍历链表时将值按顺序放入栈中，最后出栈的顺序即为逆序。

```java
public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
    Stack<Integer> stack = new Stack<>();
    while (listNode != null) {
        stack.add(listNode.val);
        listNode = listNode.next;
    }
    ArrayList<Integer> ret = new ArrayList<>();
    while (!stack.isEmpty())
        ret.add(stack.pop());
    return ret;
}
```

## 拓展：关于链表的基本操作

~~~java
public class Node {
    Node next=null;
    int data;
    public Node(int data){this.data=data;}
}
~~~

向链表插入数据

~~~java
 public void addNode(int d){
        Node newNode=new Node(d);
        if(head==null){
            head=newNode;
            return;
        }
        Node tmp=head;
        while(tmp.next!=null){
            tmp=tmp.next;
        }
        tmp.next=newNode;
    }
~~~

删除第index个节点，成功返回true否则false

~~~java
    public boolean deleteNode(int index){
        //注意特殊情况
        if(index<1 || index>length())
            return false;
        if(index==1){
            head=head.next;
            return true;
        }
        int i=1;
        Node preNode=head;
        Node curNode=preNode.next;
        while(curNode!=null){
            if(i==index){
                preNode.next=curNode.next;
                return true;
            }
            preNode=curNode;
            curNode=curNode.next;
            i++;
        }
        return true;
    }
~~~

删除链表的重复元素

1. 需要额外的空间hashTable/set
2. 双重循环遍历

```java
public void deleteDuplicate(){
	//Set<Integer>set=new HashSet<>();
    Hashtable<Integer,Integer> hashtable=new Hashtable<>();
    Node tmp=head;
    Node pre=null;
    while(tmp!=null){
        if(hashtable.containsKey(tmp.data))
            pre.next=tmp.next;
        else{
            hashtable.put(tmp.data,1);
            pre=tmp;
        }
        tmp=tmp.next;
    }
}
```

```java
public void deleteDuplicate2(){
    Node p=head;
    while(p!=null){
        Node q=p;
        while(q.next!=null){
            if(p.data==q.next.data){
                q.next=q.next.next;
            }else
                q=q.next;
        }
        p=p.next;
    }
}
```

链表的排序：使用冒泡法

```java
public Node orderList(){
    Node nextNode=null;
    Node curNode=head;
    int tmp=0;
    while(curNode!=null){
        nextNode=curNode.next;
        while(nextNode!=null){
            if(curNode.data>nextNode.data){
                tmp=curNode.data;
                curNode.data=nextNode.data;
                nextNode.data=tmp;
            }
            nextNode=nextNode.next;
        }
        curNode=curNode.next;
    }
    return head;
}
```

# 18.1在O(1)时间内删除链表的结点

## 解题思路

如果该节点不是尾节点，那么可以直接将下一个节点的值赋给该节点，然后令该节点指向下下个节点，在删除下一个节点，时间复杂度为O(1)

如果是尾节点，就需要先遍历链表，找到节点的前一个节点，然后让前一个节点指向null，时间复杂度为O(N)

综上，如果进行 N 次操作，那么大约需要操作节点的次数为 N-1+N=2N-1，其中 N-1 表示 N-1 个不是尾节点的每个节点以 O(1) 的时间复杂度操作节点的总次数，N 表示 1 个尾节点以 O(N) 的时间复杂度操作节点的总次数。(2N-1)/N ~ 2，因此该算法的平均时间复杂度为 O(1)。

```java
public ListNode deleteNode(ListNode head, ListNode tobeDelete) {
    if (head == null || tobeDelete == null)
        return null;
    if (tobeDelete.next != null) {
        // 要删除的节点不是尾节点
        ListNode next = tobeDelete.next;
        tobeDelete.val = next.val;
        tobeDelete.next = next.next;
    } else {
        if (head == tobeDelete)
             // 只有一个节点
            head = null;
        else {
            ListNode cur = head;
            while (cur.next != tobeDelete)
                cur = cur.next;
            cur.next = null;
        }
    }
    return head;
}
```

# 18.2 删除链表中重复的结点

## 题目描述

在一个**排序的链表**中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。 例如，链表1->2->3->3->4->4->5 处理后为 1->2->5

## 解题思路

使用递归的思想来解答此题

```java
//删除链表中重复的结点：解法一递归思想
public static ListNode deleteDuplication(ListNode pHead) {
    if (pHead == null || pHead.next == null)
        return pHead;
    ListNode next = pHead.next;
    if (pHead.val == next.val) {
        while (next != null && pHead.val == next.val)
            next = next.next;
        return deleteDuplication(next);
    } else {
        pHead.next = deleteDuplication(pHead.next);
        return pHead;
    }
}
```

```java
//删除链表中重复的结点：解法二
    public static ListNode deleteDuplication1(ListNode pHead)
    {
        if(pHead==null)
            return null;
        ListNode preNode=null;
        ListNode pNode=pHead;
        while(pNode!=null){
            ListNode nextNode=pNode.next;
            boolean needDelete=false;
            if(nextNode!=null && nextNode.val==pNode.val)
                needDelete=true;
            if(!needDelete){
                preNode=pNode;
                pNode=pNode.next;
            }else{
                int value=pNode.val;
                ListNode toBeDel=pNode;
                while(toBeDel!=null && toBeDel.val==value){
                    toBeDel=toBeDel.next;
                }
                if(preNode==null){
                    pHead=toBeDel;
                }else{
                    preNode.next=toBeDel;
                }
                pNode=toBeDel;
            }
        }
        return pHead;

    }
```

# 21.调整数组顺序使奇数位于偶数前面

## 题目描述

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变

## 解题思路

如果**不考虑相对位置**的关系，使用双指针的思路。时间复杂度O(n)，空间复杂度O(1)

```java
public static void reOrderArray1(int []array){
    if(array==null || array.length==0)
        return;
    int length=array.length;
    int head=0,tail=length-1;
    while(head<tail){
        while(head<tail && array[head]%2==1)
            head++;
        while(head<tail && array[tail]%2==0)
            tail--;
        if(head<tail){
            int tmp=array[head];
            array[head]=array[tail];
            array[tail]=tmp;
        }
    }
}
```

如果**考虑**相对位置关系，可以使用新的数组来存放。空间复杂度O(n)，时间复杂度O(n)

```java
public static void reOrderArray2(int []array){
    int oddCnt=0;
    for(int x:array){
        if(!isEven(x))
            oddCnt++;
    }
    int []copy=array.clone();
    int i=0,j=oddCnt;
    for(int num:copy){
        if(isEven(num))
            array[j++]=num;
        else
            array[i++]=num;
    }
}
private  static boolean isEven(int number){
    return number%2==0;
}
```

也可以使用冒泡思想，每次都当前偶数上浮到当前最右边。时间复杂度：O(n*n)，空间复杂度O(1）

```java
public static void reOrderArray3(int []array){
    int N=array.length;
    for(int i=N-1;i>0;i--){
        for(int j=0;j<i;j++){
            if(isEven(array[j])&&!isEven(array[j+1])){
                swap(array,j,j+1);
            }
        }
    }
}
private static void  swap(int[]nums,int i,int j){
    int t=nums[i];
    nums[i]=nums[j];
    nums[j]=t;
}
private  static boolean isEven(int number){
    return number%2==0;
}
```

# 24.反转链表

## 解题思路

递归

~~~java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    ListNode next = head.next;
    ListNode newHead = reverseList(next);
    next.next = head;
    head.next = null;
    return newHead;
}
~~~

迭代 使用头插法

```java
public ListNode ReverseList(ListNode head) {
    ListNode newList = new ListNode(-1);
    while (head != null) {
        ListNode next = head.next;
        head.next = newList.next;
        newList.next = head;
        head = next;
    }
    return newList.next;
}
```