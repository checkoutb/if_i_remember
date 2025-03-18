LT-Dis
2021年10月14日
9:34

==============================

=============================

LT24,  链表结点  按对互换

cpp,py,rb
    ListNode **pp = &head, *a, *b;
    while ((a = *pp) && (b = a->next)) {
        a->next = b->next;
        b->next = a;
        *pp = b;
        pp = &(a->next);
    }
    return head;

    pre, pre.next = self, head
    while pre.next and pre.next.next:
        a = pre.next
        b = a.next
        pre.next, b.next, a.next = b, a, b.next
        pre = a
    return self.next

My first line does set pre = self and then pre.next = head (which is equivalent to self.next = head, as pre and self are the same object).

。。右值  估计是  l-r  顺序计算，然后  左值也是  l-r  顺序计算。所以导致  pre.next  就是  self.next。  不过搞不清  self  代表了什么？  代表了一个结点？  为什么不是一个util类？

。。怎么知道  这个方法  是Node的  方法？just try？

    pre = dummy = ListNode.new 0
    pre.next = head
    while a = pre.next and b = a.next
        c = b.next
        ((pre.next = b).next = a).next = c
        pre = a
    end
    dummy.next

=================================================

LT0029,  不用  * / %  完成除法。

// if (A == INT_MIN && B == -1) return INT_MAX;
// int a = abs(A), b = abs(B), res = 0;
// for (int x = 31; x >= 0; x--)
//     if ((signed)((unsigned)a >> x) - b >= 0)
//         res += 1 << x, a -= b << x;
// return (A > 0) == (B > 0) ? res : -res;

O(32) == 0(1)

===================================

LT33，有序的distinct数组  全体循环移位后，进行搜索。

// [12, 13, 14, 15, 16, 17, 18, 19, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11] 可以当作：

//     [12, 13, 14, 15, 16, 17, 18, 19, inf, inf, inf, inf, inf, inf, inf, inf, inf, inf, inf

//     [-inf, -inf, -inf, -inf, -inf, -inf, -inf, -inf, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]

// while (lo < hi) {
//     int mid = (lo + hi) / 2;
//     double num = (nums[mid] < nums[0]) == (target < nums[0])
//                ? nums[mid]
//                : target < nums[0] ? -INFINITY : INFINITY;
//     if (num < target)
//         lo = mid + 1;
//     else if (num > target)
//         hi = mid;
//     else
//         return mid;
// }
// cpp的， 第一次见到这个 INFINITY

// while(lo<hi){
//     int mid=(lo+hi)/2;
//     if(A[mid]>A[hi]) lo=mid+1;
//     else hi=mid;
// }
// 计算出 最小值。 然后
// int mid=(lo+hi)/2;
// int realmid=(mid+rot)%n;
// if(A[realmid]==target)return realmid;
// if(A[realmid]<target)lo=mid+1;
// else hi=mid-1;

==============================

LT34，有序数组，搜索  第一次出现，最后一次出现。

// for low < high {
//     mid := (low + high)/2
//     if nums[mid] < target {
//         low = mid+1
//     } else {
//         high = mid
//     }
// }
//         find ans[0]
// high = len(nums) - 1
// for low < high {
//     mid := (low + high)/2 + 1
//     if nums[mid] <= target {
//         low = mid
//     } else {
//         high = mid - 1
//     }
// }
//          find ans[1]

// lower_bound(x), lower_bound(x+1)/upper_bound(x)

============================

LT0040
给与一个可能包含重复值的数组，  使用这些数字  来相加组成一个  target。  每个数字最多使用一次，并且  最终的  结果不能包含  相同的arr。

// for(int i = order;i<num.size();i++) // iterative component
// {
//     if(num[i]>target) return;

//     if(i&&num[i]==num[i-1]&&i>order) continue; // check duplicate combination

//     local.push_back(num[i]),

//     findCombination(res,i+1,target-num[i],local,num); // recursive componenet

//     local.pop_back();
// }
// 这种好像是， 第一次的时候 会 dfs， 后续相同值 不会 dfs， 但是 第一次的dfs 里面可以继续获得相同值。
// 这样，保证， 第一次的dfs 必然只有 1个 值，  dfs的内部 是2个 相同值， dfs再dfs的内部 是3个相同值。

=============================

LT0044
"asdasd"   和  "asd*?zcad"  是否匹配，  *代表任意长度的string(包括空)，?代表一个char

// const char* star=NULL;
// const char* ss=s;
// while (*s){

//     //advancing both pointers when (both characters match) or ('?' found in pattern)

//     //note that *p will not advance beyond its length
//     if ((*p=='?')||(*p==*s)){s++;p++;continue;}

//     // * found in pattern, track index of *, only advancing pattern pointer
//     if (*p=='*'){star=p++; ss=s;continue;}

//     //current characters didn't match, last pattern pointer was *, current pattern pointer is not *

//     //only advancing pattern pointer
//     if (star){ p = star+1; s=++ss;continue;}

//    //current pattern pointer is not star, last patter pointer was not *
//    //characters do not match
//     return false;
// }
// //check for remaining characters in pattern
// while (*p=='*'){p++;}
// return !*p;

==================================

LT0047
含有重复值的数组，  求  唯一的  permutation。

// for(int i = 0; i < nums.length; i++)
// {

//     if(visited[i] || i > 0 && nums[i] == nums[i - 1] && !visited[i - 1]) continue;

//     per.add(nums[i]);
//     visited[i] = true;
//     dfs(nums, visited, uniquePermutations, per);
//     per.remove(per.size() - 1);
//     visited[i] = false;
// }
// dfs, 先 sort nums
// 访问过， 或 和前面的相等且前面的没有访问过(且本元素没有访问过)  就 跳过。
// 就是 相同值，需要 一个个 取。  这样的话， 本次for循环 就不会 取到 重复的值。

    for k, v := range map2 {
        if k == lst {
            continue
        }
        v2 := v
        sz2 := len(arr)
        for v > 0 {     // map2[k] > 0
            v--
            arr = append(arr, k)
            map2[k]--
            dfsa47(sz1, arr, ans, map2)
        }
        map2[k] = v2
        arr = arr[0 : sz2]
    }
。。我的，如果要用这个值，就  一次性把  这个值  的剩余次数  全部  用完。  并且  下一次的值  不能和  本次相同。

==================================

LT0049，  分组为  同构(顺序不同)的string

strs.sort.group_by { |s| s.chars.sort }.values

strs.group_by { |s| s.chars.sort }.values.map(&:sort)

----- ruby - py ---

return [sorted(g) for _, g in itertools.groupby(sorted(strs, key=sorted), sorted)]

    groups = itertools.groupby(sorted(strs, key=sorted), sorted)
    return [sorted(members) for _, members in groups]

    groups = collections.defaultdict(list)
    for s in strs:
        groups[tuple(sorted(s))].append(s)
    return map(sorted, groups.values())

==================================

LT0067
二进制相加

// sum = (firstDigit ^ secondDigit ^ carry);

// carry = ((firstDigit | carry) & (secondDigit | carry) & (firstDigit | secondDigit));

// sb.append(sum);

==================================
LT0069
// 牛顿迭代法  求平方根

// long r = x;
// while (r*r > x)
//     r = (r + x/r) / 2;
// return r;

==================================
LT0070
爬楼梯，每次1步或2步，  多少种走法走到  第n阶台阶。
。。fibonacci
// int a = 1, b = 1;
// while (n--)
//     a = (b += a) - a;
// return a;

// def climb_stairs(n)
//     a = b = 1
//     n.times { a, b = b, a+b }
//     a
// end

======================================
LT0075
0,1,2组成的  重复  无序数组，   排序

// nums[i] = 2
// if n <= 1 {
//     nums[one] = 1
//     one++
// }
// if n == 0 {
//     nums[zero] = 0
//     zero++
// }

==================================

LT0080

// int i = 0;
// for (int n : nums)
//     if (i < 2 || n > nums[i-2])
//         nums[i++] = n;
// return i;

==================================

LT0081
shift  后的有序重复值数组，搜索

        if nums[left] < nums[mid]  {
            if target < nums[mid] && target >= nums[left] {
                right = mid - 1
            } else {
                left = mid + 1
            }
        } else if nums[left] == nums[mid] {
            left++
        } else {
            if target > nums[mid] && target <= nums[right] {
                left = mid + 1
            } else {
                right = mid - 1
            }
        }

1) everytime check if targe == nums[mid], if so, we find it.

2) otherwise, we check if the first half is in order (i.e. nums[left]<=nums[mid])

  and if so, go to step 3), otherwise, the second half is in order,   go to step 4)

3) check if target in the range of [left, mid-1] (i.e. nums[left]<=target < nums[mid]), if so, do search in the first half, i.e. right = mid-1; otherwise, search in the second half left = mid+1;

4)  check if target in the range of [mid+1, right] (i.e. nums[mid]<target <= nums[right]), if so, do search in the second half, i.e. left = mid+1; otherwise search in the first half right = mid-1;

        while(left<=right)
        {
            mid = (left + right) >> 1;
            if(nums[mid] == target) return true;

            // the only difference from the first one, trickly case, just updat left and right

            if( (nums[left] == nums[mid]) && (nums[right] == nums[mid]) ) {++left; --right;}

            else if(nums[left] <= nums[mid])
            {

                if( (nums[left]<=target) && (nums[mid] > target) ) right = mid-1;

                else left = mid + 1;
            }
            else
            {

                if((nums[mid] < target) &&  (nums[right] >= target) ) left = mid+1;

                else right = mid-1;
            }
        }
        return false;

==================================

LT0087

==================================

LT0089

// for (int i = 0; i < (1 << n); i++) {
//     ret.add(i ^ (i >> 1));
// }

Gray Code

G(i) = i^ (i/2).

==================================

LT0115

==================================

LT0119
// for (int j = 1; j <= rowIndex - 2; j++)
// {
//     result.add((int)(s = (rowIndex - j) * s / j));
// }
杨辉三角。的第  rowIndex  层。  哪里抄的？

==================================

LT0121

        int maxCur = 0, maxSoFar = 0;
        for(int i = 1; i < prices.length; i++) {
            maxCur = Math.max(0, maxCur += prices[i] - prices[i-1]);
            maxSoFar = Math.max(maxCur, maxSoFar);
        }
        return maxSoFar;

Kadane

https://leetcode.com/problems/best-time-to-buy-and-sell-stock/discuss/39038/Kadane's-Algorithm-Since-no-one-has-mentioned-about-this-so-far-%3A)-(In-case-if-interviewer-twists-the-input)

==================================

LT0137
// a, b := 0, 0
// for _, n := range nums {
//     a, b = ^b & (a ^ n), n & ^b & a | ^n & b & ^a
// }

==================================

LT141
链表找环

// while head:
//     if sys.getrefcount(head) > 4:
//         return True
//     head = head.next
// https://docs.python.org/3/library/sys.html#sys.getrefcount

==================================

LT168
A-1
Z-26
AZ-27

return n == 0 ? "" : convertToTitle((n - 1) / 26) + (char) ((n - 1) % 26 + 'A');

==================================

LT169
一个数出现的次数  > floor(len/2) .求这个数。

// public int majorityElement(int[] nums) {
//     int res = nums[0], count = 0;
//     for (int i = 0; i < nums.length; i++) {
//         if (count == 0) {
//             res = nums[i];
//         }
//         if (res == nums[i]) {
//             count++;
//         } else {
//             count--;
//         }
//     }
//     return res;
// }

// .....意会了。。 大约就是  不同的时候--， 由于某个数(A)的个数>一半，所以 它(A)能活下来， 其他的数(B) 都被 和它(B)不想等的数--，导致count变0,从而 从res上 被踢下来了。

多数投票算法(Boyer-Moore Algorithm)
基本思想是每次都从数组中找出不同的一对数，然后删除，最后留下的就是我们要找的数。

==================================

LT0189
循环向右旋转数组
// reverse(nums, 0, nums.length - k - 1);
// reverse(nums, nums.length - k, nums.length - 1);
// reverse(nums, 0, nums.length - 1);

// nums = [1,2,3,4,5,6,7] and k = 3, first we reverse [1,2,3,4], it becomes[4,3,2,1];

// then we reverse[5,6,7], it becomes[7,6,5],

// finally we reverse the array as a whole, it becomes[4,3,2,1,7,6,5] ---> [5,6,7,1,2,3,4].

    cnt := gcda189(sz1, k)
    for i := 0; i < cnt; i++ {
        t2 := nums[i]
        for j := (i + k) % sz1; j > i; j = (j + k) % sz1 {
            t2, nums[j] = nums[j], t2
        }
        nums[i] = t2
    }

void rotate(int nums[], int n, int k) {
    for (; k %= n; n -= k)
        for (int i = 0; i < k; i++)
            swap(*nums++, nums[n - k]);
}

==================================

LT191
获得二进制中1的个数
num = num & (num-1)     移除右侧所有的0.  。  不这个是  移除最右侧的1

LT231中
判断是不是  2的N次方
// func isPowerOfTwo(n int) bool {
//     if n < 1 {
//         return false
//     }
//     return n & (n - 1) == 0
// }

==================================

LT239

// For Example: A = [2,1,3,4,6,3,8,9,10,12,56], w=4

//     partition the array in blocks of size w=4. The last block may have less then w.

//     2, 1, 3, 4 | 6, 3, 8, 9 | 10, 12, 56|

//     Traverse the list from start to end and calculate max_so_far. Reset max after each block boundary (of w elements).

//     left_max[] = 2, 2, 3, 4 | 6, 6, 8, 9 | 10, 12, 56
//     Similarly calculate max in future by traversing from end to start.
//     right_max[] = 4, 4, 4, 4 | 9, 9, 9, 9 | 56, 56, 56

//     now, sliding max at each position i in current window, sliding-max(i) = max{right_max(i), left_max(i+w-1)}

//     sliding_max = 4, 6, 6, 8, 9, 10, 12, 56

    // max_left[0] = in[0];
    // max_right[in.length - 1] = in[in.length - 1];
    // for (int i = 1; i < in.length; i++) {

    //     max_left[i] = (i % w == 0) ? in[i] : Math.max(max_left[i - 1], in[i]);

    //     final int j = in.length - i - 1;

    //     max_right[j] = (j % w == 0) ? in[j] : Math.max(max_right[j + 1], in[j]);

    // }
    // final int[] sliding_max = new int[in.length - w + 1];
    // for (int i = 0, j = 0; i + w <= in.length; i++) {
    //     sliding_max[j++] = Math.max(max_right[i], max_left[i + w - 1]);
    // }
// too hard to understand....sad
// 感觉是  以这个点为最后一个元素 的范围的 最大值  和 以这个点为第一个元素的范围内的最大值。

但是为什么是以  w  为分组。
。应该是  和  i+w-1，  题目要求的w。

==================================

LT236

// TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
//     if (!root || root == p || root == q) return root;
//     TreeNode* left = lowestCommonAncestor(root->left, p, q);
//     TreeNode* right = lowestCommonAncestor(root->right, p, q);
//     return !left ? right : !right ? left : root;
// }

==================================

LT235
BST的最低公共父节点

// public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {

//     while ((root.val - p.val) * (root.val - q.val) > 0)
//         root = p.val < root.val ? root.left : root.right;
//     return root;
// }

// public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {

//     return (root.val - p.val) * (root.val - q.val) < 1 ? root :

//            lowestCommonAncestor(p.val < root.val ? root.left : root.right, p, q);

// }

==================================

LT234

class Solution {
public:
    ListNode* temp;
    bool isPalindrome(ListNode* head) {
        temp = head;
        return check(head);
    }

    bool check(ListNode* p) {
        if (NULL == p) return true;
        bool isPal = check(p->next) & (temp->val == p->val);
        temp = temp->next;
        return isPal;
    }
};
temp保存头，dfs到最后一个，对比。
temp保存第二个，dfs出栈一次，到倒数第二个，  对比
temp=temp.next，保存第三个，dfs出栈，到倒数第三个。对比

def isPalindrome(self, head):
    rev = None
    slow = fast = head
    while fast and fast.next:
        fast = fast.next.next
        rev, rev.next, slow = slow, rev, slow.next
    if fast:
        slow = slow.next
    while rev and rev.val == slow.val:
        slow = slow.next
        rev = rev.next
    return not rev

def isPalindrome(self, head):
    rev = None
    fast = head
    while fast and fast.next:
        fast = fast.next.next
        rev, rev.next, head = head, rev, head.next
    tail = head.next if fast else head
    isPali = True
    while rev:
        isPali = isPali and rev.val == tail.val
        head, head.next, rev = rev, head, rev.next
        tail = tail.next
    return isPali

==================================

LT232
2个stack  完成一个  queue

class Queue
    def initialize
        @in, @out = [], []
    end

    def push(x)
        @in << x
    end

    def pop
        peek
        @out.pop
    end

    def peek
        @out << @in.pop until @in.empty? if @out.empty?
        @out.last
    end

    def empty
        @in.empty? && @out.empty?
    end
end

class Queue {
    stack<int> input, output;
public:

    void push(int x) {
        input.push(x);
    }

    void pop(void) {
        peek();
        output.pop();
    }

    int peek(void) {
        if (output.empty())
            while (input.size())
                output.push(input.top()), input.pop();
        return output.top();
    }

    bool empty(void) {
        return input.empty() && output.empty();
    }
};

==================================

LT218 skyline

// https://briangordon.github.io/2014/08/the-skyline-problem.html

// Step 1:
    // Use a multimap to sort all boundary points.
    // For a start point of an interval, let the height be positive;
    // otherwise, let the height be negative. Time complexity: O(n log n)
// Step 2:

    // Use a multiset (rather than a heap/priority_queue) to maintain the current set of heights to be considered.

    // If a new start point is met, insert the height into the set, otherwise, delete the height.

    // The current max height is the back() element of the multiset.

    // For each point, the time complexity is O(log n). The overall time complexity is O(n log n).

// Step 3:
    // Delete the points with equal heights. Time: O(n)

// // Step 1:
// multimap<int, int> coords;
// for (const vector<int> & building : buildings) {
//         coords.emplace(building[0], building[2]);
//         coords.emplace(building[1], -building[2]);
// }

// // Step 2:
// multiset<int> heights = { 0 };
// map<int, int> corners;
// for (const pair<int, int> & p : coords) {
//         if (p.second > 0) {
//                 heights.insert(p.second);
//         }
//         else {
//                 heights.erase(heights.find(-p.second));
//         }
//         int cur_y = *heights.rbegin();
//         corners[p.first] = cur_y;
// }

// // Step 3:

// function<bool(pair<int, int> l, pair<int, int> r)> eq2nd = [](pair<int, int> l, pair<int, int> r){ return l.second == r.second;  };

// vector<pair<int, int>> results;

// unique_copy(corners.begin(), corners.end(), back_insert_iterator<vector<pair<int, int>>>(results), eq2nd);

// return results;

题目要找天际线的关键点，首先从左至右把可能出现关键点的位置扫描一遍，然后判断每个位置是否有关键点，并且确定关键点的具体坐标（也就是其高度）。

通过画图画出大部分可能的重叠情况，我们可以发现，所有的关键点可以分为两类：进入点和离开点。对于进入点来说，如果此刻进入的这栋楼是最高的，那他的起点就是一个关键点；如果他不是最高的，或者跟当前最高的楼一样高，说明该进入点被覆盖了，不需要添加关键点。对于离开点来说，如果他是最高的楼，那他的离开，一定会导致原本比他更低一点的楼露出，因此对应的关键点高度就是比他更低一点的楼的高度；如果他不是最高的，那他的离开就对天际线没有影响，因此不需要更新。

在代码实现时，首先我们需要把“进入点”和“离开点”都加入到一个队列中，并按照从左往右的顺序开始排序。这里的构思十分巧妙。首先考虑进入点，对于每栋楼的坐标(L,R,H)来说，我们先按照左下标来排序，即key为L。但是当两栋楼的左下标一样时，我们应该优先考虑更高的那栋楼。因此key为(L,-H)。然后再考虑离开点，离开点的右下标可以同样作为L与进入点一起排序，设想一种边界情况：当某个位置同时出现多栋楼的离开点和进入点时，此处是否有关键点以及相应关键点的高度取决于此时仍然存在的楼的高度，以及刚好出现的楼的高度，也就是说，进入点必须比离开点先考虑，而离开点之间的先后顺序并无影响，因为每栋楼的离开点所在的关键点不会由这栋楼的高度影响，另外一种情况是，如果某个位置只有离开点，没有进入点，且没有任何还存在的楼，那(R,0)也需要加入的集合里。为了区分进入点和离开点，我们把进入点表达为(L,-H,R)，把离开点表达为(R,0,None)，因此只有进入点的元组中有三个存在的数值。然后按照tuple[0]和tuple[1]来排序，以保证上述边界情况下，考虑点的优先顺序都是正确的。

接下来就是要遍历排序后的所有候选点。首先我们需要一个有序的数据结构，能够维护当前最高的一栋楼的高度和位置信息，这里的构思也非常巧妙。

为什么需要知道当前最高楼的高度？因为天际线一定是当前所有楼层最高的那条，如果在遍历过程中，当前的天际线与目前最高楼的高度不一样，说明发生了两种情况之一：一种是出现了一栋更高的楼，此时一定是经过了一个新的进入点，所以此时要加入一个新的天际线关键点，其高度就是当前最高楼的高度；另一种情况是，最高楼的高度变低了，说明一定是经过了一个离开点，有一栋最高楼在刚才离开了，天际线发生了变化，因此也要加入一个新的天际线关键点，其高度就是刚才最高楼离开之后，剩下楼层里最高的高度。如果目前已经没有任何大楼，说明到达了最低点。因此我们在初始化这个有序的结构时，需要加一个高度为0，横坐标位置无限大的点，用来完成这种“特殊情况”。总结来说，就是只要当前维护的最大楼高度不等于上一个添加的天际线关键点的高度，就需要增加一个新的关键点了。另外一个边界情况是，当目前还没有任何天际线时，我们如何判断当前最大楼高是否发生了变化呢？也就是在我们遍历第一栋楼时，我们应该直接加入这个进入点，因为这个进入点一定高于地平线，地平线就是上一个“天际线”。所以我们需要在初始化最终集合的时候，加入一个高度为0的元素，来解决这个问题。当然在最后返回的时候，还需要扔掉这个元素。除此之外也可以通过增加if逻辑来应对这种情况。

最后一个问题是，我们用什么数据结构来维护当前的最高楼层，以及如何在遍历过程中进行更新。首先我们思考一下，需要进行哪些操作：取出当前最大的楼层高度；删除已经离开的楼层高度；增加刚进入的楼层高度。何时需要增加楼层高度？当此时的点是进入点时。何时需要删除已经离开点楼层高度？当此时遍历的点已经超过了楼层的最右位置时。也就是说，我们需要存的数据，应该包含了楼的高度和最右位置。总结一下就是：每次当我们遍历到进入点时，把对应的楼高和右下标加入到这个有序结构中；每次当我们遍历到的点的位置已经超过或者等于当前最高楼的右下标时，说明这栋楼已经失效（为什么等于也不行：因为最开始分析过，楼的离开对这栋楼所构成的天际线是没有影响的，可以看作是左闭右开的区间），这个操作需要一直执行，直到当前的最高楼仍然在范围内为止。总结到这里，我们就可以开始选择合适的数据结构。如何能够快速的维护最大值、删除最大值以及增加一个新的元素？答案就是堆。python中heapq库可以方便地维护一个有序的list，通过heappop和heappush可以方便地增删元素。取最大值则只需要通过获取第一个元素（注意heapq默认是升序排列，因此我们仍存储-H来实现降序）。

==================================

LT0363

==================================

LT0365

2个容器，每个容器的  容量给定。  需要  获得  m升的水。  问是否可能。

Bézout's identity (also called Bézout's lemma) is a theorem in the elementary theory of numbers:

let a and b be nonzero integers and let d be their greatest common divisor. Then there exist integers x

and y such that ax+by=d

In addition, the greatest common divisor d is the smallest positive integer that can be written as ax + by

every integer of the form ax + by is a multiple of the greatest common divisor d.

裴蜀定理（或贝祖定理）得名于法国数学家艾蒂安·裴蜀，说明了对任何整数a、b和它们的最大公约数d，关于未知数x和y的线性不定方程（称为裴蜀等式）：若a,b是整数,且gcd(a,b)=d，那么对于任意的整数x,y,ax+by都一定是d的倍数，特别地，一定存在整数x,y，使ax+by=d成立。

它的一个重要推论是：a,b互质的充分必要条件是存在整数x,y使ax+by=1.

==================================

// sqrt(2147483647)   <    46341

==================================

LT0367.  给一个数，判断是否是  某个数的平方。

1 = 1
4 = 1 + 3
9 = 1 + 3 + 5
16 = 1 + 3 + 5 + 7
25 = 1 + 3 + 5 + 7 + 9
36 = 1 + 3 + 5 + 7 + 9 + 11
....
so 1+3+...+(2n-1) = (2n-1 + 1)n/2 = nn

牛顿方法  求平方根。
        long x = num;
        while (x * x > num) {
            x = (x + num / x) >> 1;
        }
        return x * x == num;

(a+b)/2 >= sqrt(a*b) only when a = b the left is equal to the right

==================================

==================================

LT0383

ransomNote  是否可以由  magazine  的char  组成。

    bool canConstruct(string ransomNote, string magazine) {
        std::multiset<char> mag(magazine.begin(), magazine.end());
        std::multiset<char> ransom(ransomNote.begin(), ransomNote.end());

        return std::includes(mag.begin(), mag.end(), ransom.begin(), ransom.end());

    }

def canConstruct(self, ransomNote, magazine):
    return not collections.Counter(ransomNote) - collections.Counter(magazine)

==================================

LT0385  shuffle array

// for (int i = 0;i < result.size();i++) {
//     int pos = rand()%(result.size()-i);
//     swap(result[i+pos], result[i]);
// }

    for i, _ := range this.arr {
        i2 := rand.Intn(len(this.arr))
        ans[i], ans[i2] = ans[i2], ans[i]
    }

这2种  有什么区别呢。  哪种更加  随机(平均)

==================================

LT0391   多个矩形  是否  正好组成一个  大矩形

遍历，获得  长宽  面积，计算  大矩形的面积  是否==  小矩形面积和。
遍历，查找是否  有重叠部分。

-------

每个小矩形的4个点  都只可能出现  2次  或4次。  除了  4个顶点。

==================================

LT0023

        auto mycomp = [](const ListNode* n1, const ListNode* n2) {
            return n1->val >= n2->val;
        };

        lists.erase(std::remove_if(begin(lists), end(lists), [](const ListNode* np) { return np == nullptr; }), end(lists));

        std::priority_queue<ListNode*, vector<ListNode*>, decltype(mycomp)> priq{ mycomp, lists };

==================================

// LT2598  BIT(Fenwick)
    //int bt[2002] = {}, n = 2001;
    //int prefix_sum(int i)
    //{
    //    int sum = 0;
    //    for (i = i + 1; i > 0; i -= i & (-i))
    //        sum += bt[i];
    //    return sum;
    //}
    //void add(int i, int val)
    //{
    //    for (i = i + 1; i <= n; i += i & (-i))
    //        bt[i] += val;
    //}

==================================

==================================

==================================

==================================

==================================

==================================

==================================

==================================

==================================

已使用 Microsoft OneNote 2016 创建。


```C++
sort(begin(vvi), end(vvi), [](vector<int>& v1, vector<int>& v2){
	return v1[2] < v2[2];
});

// --- another way

for (vector<int>& vi : vvi) {
	swap(vi[0], vi[2]);
}
sort(begin(vvi), end(vvi))
```


---

LT2810: 遇到i 就反转前面的子串
```C++
string finalString(const string &s) {
    string a, b;
    for (char ch : s)
        if (ch == 'i')
            swap(a, b);
        else
            a += ch;
    return string(rbegin(b), rend(b)) + a;
}
```