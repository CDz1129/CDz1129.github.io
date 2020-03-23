---
layout:     post
title:      ARTS_WEEK16
subtitle:   算法、数据库、面试题、spring、Linux
date:       2020-03-22
author:     CDz
catalog: true
tags:
    - ARTS
---


这段时间还是主要在算法与高频的面试题目环节下文章。

但是这周有个问题非常严重，视频没有再有时间去看，还有极客专栏虽然看了很多课程，但是真正回想起来却没有很多印象。

所以得改变一下自己的看专栏策略，每篇专栏看完之后要写下对这篇专栏的**描述和疑问**，当做一个笔记。因为有些文章很多基础不够牢固，不知道要记一些什么笔记，而自己又不想笔记全是一篇文章中的抄写。

记笔记是一个需要去探索研究的。

目前想到的方案：

- 印象笔记
	- 模板——**描述与疑问**
- notability
	- 画图、演算
- things3
	- 练习项纪录
- 周结：
	- 思维导图xmind
	- ARTS
- 课结
	- 已思维导图为框架结构
	- 印象笔记的内容为主
	- notability、things3内容为辅
## 1. although
### 1.1. 每日一练
#### 1.1.1. 字符串压缩

```
/**
 * 面试题 01.06. 字符串压缩
 * 字符串压缩。利用字符重复出现的次数，编写一种方法，实现基本的字符串压缩功能。比如，字符串aabcccccaaa会变为a2b1c5a3。若“压缩”后的字符串没有变短，则返回原先的字符串。你可以假设字符串中只包含大小写英文字母（a至z）。
 * <p>
 * 示例1:
 * <p>
 * 输入："aabcccccaaa"
 * 输出："a2b1c5a3"
 * 示例2:
 * <p>
 * 输入："abbccd"
 * 输出："abbccd"
 * 解释："abbccd"压缩后为"a1b2c2d1"，比原字符串长度更长。
 * 提示：
 * <p>
 * 字符串长度在[0, 50000]范围内。
 */
```

```java
public String compressString(String S) {

        if (S.length()<=1){
            return S;
        }

        char[] chars = S.toCharArray();

        StringBuilder sb = new StringBuilder();
        char temp = chars[0];
        Integer tempCount = 1;
        for (int i = 1; i < chars.length; i++) {
            if (chars[i]!=temp){
                sb.append(temp)
                        .append(tempCount);
                temp = chars[i];
                tempCount = 1;
            }else {
                tempCount++;
            }

            if (i==chars.length-1){
                sb.append(temp).append(tempCount);
            }
        }
        if (sb.length()>=S.length()){
            return S;
        }
        return sb.toString();
    }
```

#### 1.1.2. 最小的k个数

```
/**
 * 面试题40. 最小的k个数
 * 输入整数数组 arr ，找出其中最小的 k 个数。例如，输入4、5、1、6、2、7、3、8这8个数字，则最小的4个数字是1、2、3、4。
 * <p>
 * <p>
 * <p>
 * 示例 1：
 * <p>
 * 输入：arr = [3,2,1], k = 2
 * 输出：[1,2] 或者 [2,1]
 * 示例 2：
 * <p>
 * 输入：arr = [0,1,2,1], k = 1
 * 输出：[0]
 * <p>
 * <p>
 * 限制：
 * <p>
 * 0 <= k <= arr.length <= 10000
 * 0 <= arr[i] <= 10000
 */
```

分析：

与第K大的数解法类似，都是求出一个集合中最大或最小的几个数字。

解题有两种大方向：

- 所有数字读入内存
	- 快排思想partition分割
	- 全部排序
- 不读入内存中
	- 堆排序，一个一个读取放入到堆中

##### 1.1.2.1. 全排序

```java
//排序O(nlogn)
    public int[] getLeastNumbers(int[] arr, int k) {
        Arrays.sort(arr);
        int[] ints = new int[k];
        for (int i = 0; i < k; i++) {
            ints[i] = arr[i];
        }
        return ints;
    }
```

##### 1.1.2.2. 快排思想

```java
//快速排序
    public int[] getLeastNumbers2(int[] arr, int k) {
        if (k == 0 || arr.length == 0) return new int[0];
        return quicksort(arr, 0, arr.length - 1, k - 1);
    }

	private int[] quicksort(int[] arr, int lo, int hi, int k) {

        int j = partition(arr, lo, hi);

        if (j == k) {
            return Arrays.copyOf(arr, k + 1);
        }

        return j > k ? quicksort(arr, lo, j - 1, k) : quicksort(arr, j + 1, hi, k);
    }

```

###### 1.1.2.2.1. Partition方法——其实就是荷兰旗问题

```
	/**
     * 荷兰旗问题
     *
     * 假设
     *  p0 m p2
     *  这三者区域 m=v
     *  p0区域所有值都小于 v
     *  p2区域所有值都大于 v
     *
     * @param arr
     * @param lo
     * @param hi
     * @return
     *
	*/
```

```java

    private int partition(int[] arr, int lo, int hi) {

        int v = arr[lo];

        int p0 = lo;
        int curr = lo;
        int p2 = hi;

        while (p2>=curr){
            if (arr[curr]>v){
                swap(arr,curr,p2);
                p2--;
            }else if (arr[curr]<v){
                swap(arr,curr,p0);
                p0++;
                curr++;
            }else {
                curr++;
            }
        }
        //最后要将最 左边的元素交换到 小于v元素的最大下标上
        swap(arr,p0,lo);
        return p0;
    }
```

##### 1.1.2.3. 堆排序思想

```java
//堆排序O(nlogn)
    //好处 在如果是输入流的情况下
    //可以很好的控制内存使用
    public int[] getLeastNumbers1(int[] arr, int k) {
        PriorityQueue<Integer> queue = new PriorityQueue<>(k);
        for (int i : arr) {
            queue.add(i);
        }
        int[] ints = new int[k];
        for (int i = 0; i < k; i++) {
            ints[i] = queue.poll();
        }
        return ints;
    }
```

#### 1.1.3. 矩形重叠

```
/**
 * Created with IntelliJ IDEA.
 * Description:
 * User: CDz
 * Date: 2020-03-18
 * Time: 下午4:35
 * <p>
 * 836. 矩形重叠
 * 矩形以列表 [x1, y1, x2, y2] 的形式表示，其中 (x1, y1) 为左下角的坐标，(x2, y2) 是右上角的坐标。
 * <p>
 * 如果相交的面积为正，则称两矩形重叠。需要明确的是，只在角或边接触的两个矩形不构成重叠。
 * <p>
 * 给出两个矩形，判断它们是否重叠并返回结果。
 * <p>
 * <p>
 * <p>
 * 示例 1：
 * <p>
 * 输入：rec1 = [0,0,2,2], rec2 = [1,1,3,3]
 * 输出：true
 * 示例 2：
 * <p>
 * 输入：rec1 = [0,0,1,1], rec2 = [1,0,2,1]
 * 输出：false
 * <p>
 * <p>
 * 提示：
 * <p>
 * 两个矩形 rec1 和 rec2 都以含有四个整数的列表的形式给出。
 * 矩形中的所有坐标都处于 -10^9 和 10^9 之间。
 * x 轴默认指向右，y 轴默认指向上。
 * 你可以仅考虑矩形是正放的情况。
 */
```

分析：
1. 两个矩形相交，确定一个矩形的位置，必须满足另一个不在其四周外部。
2. 两个矩形横线或者竖线有交点。数学定义：
	1. 在坐标轴上表示就是为，竖线上，两个矩形的竖线上顶点的最小值要大于，下顶点的最大值
	2. 横线上，两右顶点的最小值大于，两左顶点的最大值

```java
/**
     * 分析：
     * rec1 = [0,0,2,2], rec2 = [1,1,3,3]
     * <p>
     * 左右：0~2 1~3
     * 上下：0~2 1~3
     * <p>
     * rec1 = [0,0,1,1], rec2 = [1,0,2,1]
     * <p>
     * 左右：0~1 1~2
     * 上下：0~1 0~1
     *
     * @param rec1
     * @param rec2
     * @return
     */
    public boolean isRectangleOverlap(int[] rec1, int[] rec2) {

        /**
         * 如果相交必须满足
         *
         * 矩形rec1 在矩形 rec2 左边
         * 矩形rec1 在矩形 rec2 右侧
         * 矩形rec1 在矩形 rec2 上边
         * 矩形rec1 在矩形 rec2 下边
         *
         */

        return !(rec1[2] <= rec2[0] ||//左边
                rec1[0] >= rec2[2] ||//右侧
                rec1[3] <= rec2[1] ||//上边
                rec1[1] >= rec2[3]); //下边
    }

    /**
     * 两线段交际
     *
     * rec1的横线与rec2的横线相交
     *
     * rec1的竖线与rec2的竖线相交
     * @param rec1
     * @param rec2
     * @return
     */
    public boolean isRectangleOverlap1(int[] rec1, int[] rec2) {

        //根据数学知识我们可以知道，
        // 当 min(rec1[2], rec2[2]) > max(rec1[0], rec2[0]) 时，这两条线段有交集。

        return Math.min(rec1[2],rec2[2])>Math.max(rec1[0],rec2[0])&&
                Math.min(rec1[3],rec2[3])>Math.max(rec1[1],rec2[1]);

    }
```

#### 1.1.4. 最长回文串

```
/**
 * Created with IntelliJ IDEA.
 * Description:
 * User: CDz
 * Date: 2020-03-19
 * Time: 上午10:28
 * 409. 最长回文串
 * 给定一个包含大写字母和小写字母的字符串，找到通过这些字母构造成的最长的回文串。
 * <p>
 * 在构造过程中，请注意区分大小写。比如 "Aa" 不能当做一个回文字符串。
 * <p>
 * 注意:
 * 假设字符串的长度不会超过 1010。
 * <p>
 * 示例 1:
 * <p>
 * 输入:
 * "abccccdd"
 * <p>
 * 输出:
 * 7
 * <p>
 * 解释:
 * 我们可以构造的最长的回文串是"dccaccd", 它的长度是 7。
 */
```

分析:
回文字符串的性质，双数，或者中间一个为单数，其他都成对出现。

- `abba -> ab|ba`
- `ababa -> ab(a)ba`

注意点：
ASCII编码表中，`a~z A~Z`不止有52个。** 大小写字母之间有好几个特殊字符**。

```java
/**
     * 回文串的特殊性质
     *
     * 对称性，对称引出特性是：
     *
     * 如果有奇数个字符：只有一种为奇数个
     * 其他都为偶数
     *
     * 例如：
     * abba -> ab|ba
     * ababa -> ab(a)ba
     *
     * 那么问题就可以变换一下，
     * 统计字符串内有多少 偶数字符串，如果奇数就减一。
     *
     * @param s
     * @return
     */
    public int longestPalindrome(String s) {

        //统计每个字符数量
        //刚开始不知道为什么要58的数组，大小写字母只有52个，后来查阅了一下ASCII编码表，原来大小写字母之间有好几个特殊字符
        int[] counts = new int[58];
        char[] chars = s.toCharArray();
        for (char aChar : chars) {
            //a~z A~Z 52个字符串中
            //A为最小 char
            counts[aChar-'A']++;
        }

        int ans = 0;
        for (int count : counts) {
            ans+= count - (count&1);
        }
        return ans<s.length()?ans+1:ans;
    }
```

#### 1.1.5. 水壶问题

```
/**
 * 365. 水壶问题
 * 有两个容量分别为 x升 和 y升 的水壶以及无限多的水。请判断能否通过使用这两个水壶，从而可以得到恰好 z升 的水？
 * <p>
 * 如果可以，最后请用以上水壶中的一或两个来盛放取得的 z升 水。
 * <p>
 * 你允许：
 * <p>
 * 装满任意一个水壶
 * 清空任意一个水壶
 * 从一个水壶向另外一个水壶倒水，直到装满或者倒空
 * 示例 1: (From the famous "Die Hard" example)
 * <p>
 * 输入: x = 3, y = 5, z = 4
 * 输出: True
 * 示例 2:
 * <p>
 * 输入: x = 2, y = 6, z = 5
 * 输出: False
 */
```

分析：  
方法一：

状态转换过程：

1. 装满其中一桶水
	1. A倒满，B不动
	2. A不动，B倒满
2. 倒掉其中一桶水
	1. A倒空，B不动
	2. A不动，B倒空
3. 将一桶水倒入另一桶水中
	1. A倒空，B没满
	2. A没倒空，B满
	3. B倒空，A没满
	4. B没倒空，A满

方法二：

数学方法，题目可知其状态 「变化」的过程肯定是 x y 的整数次，那么就需要满足一个公式  `ax+by = z`，数学知识可以得到，只有当a与b的最大公约数能除尽z才会有解，既：`gcd(a,b)%z==0`

```java
//方法一：	
//广度搜索 BFS
    public boolean canMeasureWater(int x, int y, int z) {

        if (x + y == z) return true;
        if (x + y < z) return false;

        //初始状态
        State state = new State(0, 0);

        //创建栈 模拟方法栈
        LinkedList<State> queue = new LinkedList<>();
        queue.push(state);

        //创建一个set集合 缓存结果 避免重复计算
        HashSet<State> statesSet = new HashSet<>();

        //方法栈遍历
        while (!queue.isEmpty()) {
            //取出栈头
            State poll = queue.poll();
            //将栈头信息拿出
            int x1 = poll.getX();
            int y1 = poll.getY();
            if (x1 == z || y1 == z || x1 + y1 == z) return true;
            //获取从此状态下，所有的下一步状态集合

            List<State> nextList = getNextStateList(x1, y1, x, y);
            //判断下一步状态是否计算过

            for (State next : nextList) {
                if (!statesSet.contains(next)) {
                    //没有放入调用栈和set集合
                    queue.push(next);
                    statesSet.add(next);
                }
            }
        }

        return false;
    }

	private List<State> getNextStateList(int currX, int currY, int x, int y) {

        ArrayList<State> list = new ArrayList<>();

        //1. 倒掉x
        State state1 = new State(0, currY);
        //2. 倒掉y
        State state2 = new State(currX, 0);
        //3. 装满x
        State state3 = new State(x, currY);
        //4. 装满y
        State state4 = new State(currX, y);

        //5. 将当前currX倒入currY中 直到倒满
        //分两种情况
        //X有剩余 Y满
        State state5 = new State(currX - (y - currY), y);
        //X倒完 Y没有满
        State state6 = new State(0, currX + currY);
        //6. 当前currY倒入currX中 直到倒满
        //分两种情况
        //Y有剩余 X满
        State state7 = new State(x, currY - (x - currX));
        //Y倒完 X没有满
        State state8 = new State(currX + currY, 0);

        //没有满时才需要加水
        if (currX < x) {
            list.add(state3);
        }
        if (currY < y) {
            list.add(state4);
        }
        //不为空时才能倒空
        if (currX != 0) {
            list.add(state1);
        }
        if (currY != 0) {
            list.add(state2);
        }

        //有剩余才能倒
        if (currX - (y - currY) > 0) {
            list.add(state5);
        }
        if (currY - (x - currX) > 0) {
            list.add(state7);
        }

        //能倒过去才能倒
        if (currX + currY < y) {
            list.add(state6);
        }
        if (currX + currY < x) {
            list.add(state8);
        }
        return list;
    }

    private class State {
        private int x;
        private int y;

        public State(int x, int y) {
            this.x = x;
            this.y = y;
        }

        public int getX() {
            return x;
        }

        public int getY() {
            return y;
        }


        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            State state = (State) o;
            return x == state.x &&
                    y == state.y;
        }

        @Override
        public int hashCode() {
            return Objects.hash(x, y);
        }
    }
```

```java
	//方法二：
    //数学方法
    public boolean canMeasureWater1(int x, int y, int z) {
        /**
         * 如果要判断满足条件
         * 由题意可以知道一点
         * 其状态 「变化」的过程肯定是 x y 的整数次
         * 那么就需要满足一个公式  ax+by = z
         *
         */
        if (x + y == z) return true;
        if (x + y < z) return false;
        if (x == 0 || y == 0) return z == 0;
        return z%gcd(x,y)==0;
    }

	//最大公约数
    //辗转相除
    private int gcd(int a, int b) {

        int c = a % b;

        return c == 0 ? b : gcd(b, c);


    }
```

#### 1.1.6. 使数组唯一的最小增量

```
/**
 * 945. 使数组唯一的最小增量
 * 给定整数数组 A，每次 move 操作将会选择任意 A[i]，并将其递增 1。
 * <p>
 * 返回使 A 中的每个值都是唯一的最少操作次数。
 * <p>
 * 示例 1:
 * <p>
 * 输入：[1,2,2]
 * 输出：1
 * 解释：经过一次 move 操作，数组将变为 [1, 2, 3]。
 * 示例 2:
 * <p>
 * 输入：[3,2,1,2,1,7]
 * 输出：6
 * 解释：经过 6 次 move 操作，数组将变为 [3, 4, 1, 2, 5, 7]。
 * 可以看出 5 次或 5 次以下的 move 操作是不能让数组的每个值唯一的。
 * 提示：
 * <p>
 * 0 <= A.length <= 40000
 * 0 <= A[i] < 40000
 */
```

分析：

得看懂题目意思：*给一个数组，将里面的元素，通过每次自增1，将这个数组的元素，全部变成唯一的，不重复。求出最小的自增次数*

方法一：排序，然后计算前后元素的关系，让前后关系变为唯一。

方法二：桶计数，计数每个数的次数。数量大于二的，只留一个在原地，其他的向前累加，所以全部都move 1。最后需要用到求和公式`1+2+3+...+n=n*(n+1)/2`。

```java
//方法一：排序
public int minIncrementForUnique(int[] A) {
        /**
         * 题目的意思是，
         * 给一个数组，将里面的元素，通过每次自增1，将这个数组的元素，全部变成唯一的，不重复
         *
         * 求出最小的自增次数
         *
         * 解题思路，找到元素之间的关系，然后将其变为不通的元素
         *
         */
        //1. 排序
        Arrays.sort(A);

        int res = 0;

        //遍历排序后的所有元素
        //找到两两之间的关系
        //因为已经排序过了，所以，只需要找到前后两个元素的关系
        //然后进行比较，只需要比前一个数字大一位就可以
        for (int i = 1; i < A.length; i++) {
            int pre = A[i-1];
            int curr= A[i];
            if (pre>=curr){
                res += pre-curr+1;
                A[i] = pre+1;
            }
        }

        return res;
    }
```

```
//桶统计方法

    /**
     * 思路，先统计A中的数字，出现的次数
     * 并且桶统计时天然排序好了
     *
     * 这时，需要做的是将桶位上如果大于1的数
     * 变成1 并且其他的数字向后移动，以此类推
     *
     * 当到了最后一个后一位（因为我们全部都向后移动了）
     * 如果大于1 那么需要再向后移动，比如说有cont[max+1]=4
     * 那么将后面的 max+1+1 max+1+2 max+1+3 max+1+4 这四个下标都填满
     *
     * 后面相加的数就是移动的位数
     *
     * 那么其实就是 1+2+3+4=(4+1)*4/2 求和公式
     *
     * 如果cont[max+1]= n
     * 即为 (n+1)*n/2
     * @param A
     * @return
     */
```

```java
//方法二
public int minIncrementForUnique1(int[] A) {

        //有题目可知 0 <= A[i] < 40000
        //所以我们可以创建一个 40000容量的数组
        int[] countor = new int[40000];

        //记录A数组中最大的那个数字，方便后面计算
        int max = -1;
        for (int i : A) {
            countor[i]++;
            max = Math.max(max,i);
        }

        int ans = 0;
        for (int i = 0; i <= max; i++) {
            if (countor[i]>=2){
                //计数超过2
                //将超过数量的数字全部向前移动一位
                //也就是需要move count次
                int count = countor[i]-1;
                countor[i+1]+=count;
                ans+=count;
            }
        }

        //处理最后一个
            //max+1位只留下一个
            //剩下count个数，计算所有的move次数
            int count = countor[max+1]-1;
            ans+=(count+1)*count/2;

        return ans;
    }
```


### 1.2. 二叉树练习
#### 1.2.1. 最长同值路径

```
/**
 * Created with IntelliJ IDEA.
 * Description:
 * User: CDz
 * Date: 2020-03-18
 * Time: 下午5:49
 * <p>
 * 687. 最长同值路径
 * 给定一个二叉树，找到最长的路径，这个路径中的每个节点具有相同值。 这条路径可以经过也可以不经过根节点。
 * <p>
 * 注意：两个节点之间的路径长度由它们之间的边数表示。
 * <p>
 * 示例 1:
 * <p>
 * 输入:
 * <p>
 * 5
 * / \
 * 4   5
 * / \   \
 * 1   1   5
 * 输出:
 * <p>
 * 2
 * 示例 2:
 * <p>
 * 输入:
 * <p>
 * 1
 * / \
 * 4   5
 * / \   \
 * 4   4   5
 * 输出:
 * <p>
 * 2
 * 注意: 给定的二叉树不超过10000个结点。 树的高度不超过1000。
 */
```

```java
int ans = 0;
    public int longestUnivaluePath(TreeNode root) {
        arrowLength(root);
        return ans;
    }

    private int arrowLength(TreeNode root) {

        if (root==null)return 0;
        int left = arrowLength(root.left);
        int right = arrowLength(root.right);
        int arrowLeft = 0;
        int arrowRight = 0;

        if (root.left!=null&&root.left.val==root.val){
            arrowLeft+=left+1;
        }
        if (root.right!=null&&root.right.val==root.val){
            arrowRight+=right+1;
        }
        ans = Math.max(ans,arrowLeft+arrowRight);

        return Math.max(arrowLeft,arrowRight);
    }
```

#### 1.2.2. 左叶子之和

```
/**
 * 404. 左叶子之和
 * 计算给定二叉树的所有左叶子之和。
 * <p>
 * 示例：
 * <p>
 * 3
 * / \
 * 9  20
 * /  \
 * 15   7
 * <p>
 * 在这个二叉树中，有两个左叶子，分别是 9 和 15，所以返回 24
 * <p>
 * <p>
 * 读题，所有左叶子节点之和，并不是**左子节点**
 */
```

```java
public int sumOfLeftLeaves(TreeNode root) {

        return sumOfLeftLeavesHelper(root, false);
    }

    private int sumOfLeftLeavesHelper(TreeNode root, boolean isLeft) {

        if (root == null) return 0;

        int leave = 0;
        if (isLeft && root.left == null && root.right == null) {
            leave = root.val;
        }
        int leftCount = sumOfLeftLeavesHelper(root.left, true);
        int rightCount = sumOfLeftLeavesHelper(root.right, false);
        return leave+leftCount+rightCount;
    }
```

## 2. review

### 2.1. [ SQLite vs MySQL vs PostgreSQL ][1]

文章对比了SQLite 、MySQL、postgreSQL。

各个特点：

- SQLite:
	- 简小，开箱即用
	- 适合APP
	- 没有用户管理，既没有权限管理
- MySQL
	- 最流行
	- 安全的权限管理
	- 速度客观
	- 不严格按照SQL规范
	- 进展缓慢
	- web网页开发
	- 预期未来增长
- PostgreSQL
	- 新型的SQL
		- 自称最先进的开源关系型数据库
	- 完全符合SQL规范
	- 开源
	- 扩展性
	- 数据量大的情况
	- 复杂的操作
		- 可以利用多个CPU查询计划
	- 消耗内存
		- 每次一个连接都会fork线程，大概10MB


## 3. tip
### 3.1. 德雷福斯模型
专业技术能力成长模型。

成长模型分为五个阶段：

1. 新手
2. 高级新手
3. 精通者
4. 胜人者
5. 专家

从新手到高级新手是一个自然进升的过程，也有一些人可以从新手直接进升为精通者。但是可怕的是，只有在这一个阶段是自然晋升。

而后都需要不断的刻意练习，才有可能进入下一个阶段。

正确的认识自己，确实现在还在高级新手的阶段。可怕的是，在我工作了这几年之后，总觉得很多东西都可以做，给的任务都可以完成。就觉得自己是一个能力强的人，当看到这个模型的时候，我严重的意识到，我只是一个高级新手。

## 4. share
### 4.1. 面试题
#### 4.1.1. SpringMVC 访问controller是否新建线程？
SpringMVC 访问controller是否新建线程

首先springMVC的controller默认是单例的，也就是说在任何时候只有一个实例。

而controller的执行是通过Tomcat的线程池来调用的，虽然请求仍然可以提交（通过线程池）

但是调用方法中，堵塞的时间是累加的。

也就是说，如果一个controller固定耗时为1秒，那么同时发出的两个请求，第一个处理1秒时，第二个请求需要堵塞等待最终处理时间会被累加1秒。
#### 4.1.2. Linux 权限的概念

```bash
chmod 777 file
```
是什么意思？

这里有两层意思：

1. 权限
2. 给那些人权限

Linux的权限逻辑是，1️⃣给那些人权限，2️⃣给的这些人能有什么样的操作权限

操作权限：

* r:读，简写数字4
* w:写，简写数字2
* x:执行，简写数字1
任意两种权限可以相加：比如给一个读写权限rw:4+2=6

而我们看到是三个数字组成，这是一种省略写法，其实是：

```bash
chmod u:7 g:7 o:7 file
```

* U:文件所有者
* G:文件所有者所在组
* O:other
所以如果我们设置一个文件权限为：

```bash
chmod 600 file
```

意思为：

* 此文件创建者的权限为rw（读写权限r=4,w=2）
* 此文件创建者所在组没有任何权限
* 其他人没有任何权限

#### 4.1.3. 静态代码什么时候初始化?

静态代码块或者是静态代码，是在类初始化的时候加载。在JVM层面

静态变量或者是静态代码块的作用是：
给类初始化使用的。

构造代码作用：
给对象初始化使用的。
#### 4.1.4. 判断两个链表是否相交

https://blog.csdn.net/fengxinlinux/article/details/78885764

1. 暴力破解
2. 相交一定最后一个节点相同
	1. 先判断两个链表长度
	2. 长的len1短的len2
	3. 长链表从表头先向后移动(len1-len2)
	4. 然后两链表一起移动
	5. 比较节点找到相同
3. hash方法
	1. 先遍历一个加入hash表中
	2. 再遍历第二个链表
	3. 每次判断是否存在hash表
4. 使用两个栈
	1. 同时将两个链表加入栈中
	2. Pop出两个栈中的元素进行对比
5. 人为构成环
	1. 将一个链表的尾部与第二个链表的头部相结合
	2. 再通过快慢指针的方式判断链表是否有环

[1]:	https://www.digitalocean.com/community/tutorials/sqlite-vs-mysql-vs-postgresql-a-comparison-of-relational-database-management-systems#sqlite
