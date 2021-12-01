### [剑指]() Offer 09. 用两个栈实现队列

常规的想法是用一个栈作辅助栈，然后倒来倒去

但是我们注意到队列**一头只负责进，另一头只负责出**，因此我们可以用两个栈分别承担两个功能

1. s1负责进队列，每次push都进入s1
2. s2负责出队列，每次pop都从s2出
   如果s2为空，则一次性将s1中所有元素倒入s2（出队的顺序是正好的，可以全部倒入），如果两个栈都为空，则删除失败返回-1

```c++
class CQueue {
private:
    // s1代表队尾，s2代表队头
    stack<int> s1, s2;

public:
    CQueue() {

    }
    
    void appendTail(int value) {
        s1.push(value);
    }
    
    void mov(stack<int>&from, stack<int>& to){
        while (!from.empty()){
            int temp = from.top();
            from.pop();
            to.push(temp);
        }
    }

    int deleteHead() {
        if (s2.empty()){
            if (s1.empty()) return -1;
            mov(s1, s2);
        }
        int temp = s2.top();
        s2.pop();
        return temp;
    }
};
```



### 299. 猜数字游戏

> 写出一个秘密数字，并请朋友猜这个数字是多少。朋友每猜测一次，你就会给他一个包含下述信息的提示：
>
> - 猜测数字中有多少位属于数字和确切位置都猜对了（称为 "Bulls", 公牛）
>
> - 有多少位属于数字猜对了但是位置不对（称为 "Cows", 奶牛），也就是说，这次猜测中有多少位非公牛数字可以通过重新排列转换成公牛数字。
>
> 给你一个秘密数字 secret 和朋友猜测的数字 guess ，请你返回对朋友这次猜测的提示。
>
> 提示的格式为 "xAyB" ，x 是公牛个数， y 是奶牛个数，A 表示公牛，B 表示奶牛。

- bulls很好找，只要遍历一遍找到位置相同的即可，然后我们把bulls设为字母，避免之后找cows的时候重复计算
- cows的寻找方法利用了unordered_map进行遍历，相当于找两个集合的交集

```c++
class Solution {
public:
    string getHint(string secret, string guess) {
        size_t len = secret.size();
        unordered_map<char, int> mp;
        int bulls = 0, cows = 0;
        for (int i = 0; i < len; ++i){
            if (secret[i] == guess[i]){
                ++bulls;
                secret[i] = guess[i] = 'a';
            } else {
                ++mp[secret[i]];
            }
        }
        for (auto& t : guess){
            if (t == 'a') continue;
            if (mp[t] > 0) ++cows, --mp[t];
        }
        return to_string(bulls) + "A" + to_string(cows) + "B";
    }
};
```



### 剑指 Offer 07. 重建二叉树

> 根据前序遍历和中序遍历重建二叉树

**根据自己总结**：二叉树题通常三种方法

1. 用数组二叉树，每个叶子标号
2. 用`void dfs();`函数遍历，采用遍历的思想删除或者更改结点
3. 用`TreeNode* build();`函数，返回值是TreeNode*的通常是建立二叉树，内部结构一般为`root->left = build(root->left, ...);`和`root->right = build(root->right, ...);`

前序遍历的特点是在某树所有节点中，根节点一定在第一个；中序遍历的特点是根节点把左右子树区分开来。因此我们想到在前序遍历中获得根节点，再在中序遍历中找到其位置，最后对中序遍历左右两块各自调用build函数建立左右子树
需要注意的是递归时pos（前序遍历下标）的确定（确定右子树root时要根据左子树的节点数量进行右移）

```c++
class Solution {
private:
    size_t len;

public:
    // 根据先序找到每一段的根节点，可以在中序分出左右子树
    TreeNode* build(TreeNode* root, vector<int>& preorder, vector<int>& inorder, int left, int right, int pos){
        if (pos == len || left > right) return root;
        int newRoot = preorder[pos];
        int newPos = find(inorder.begin(), inorder.end(), newRoot) - inorder.begin();

        root = new TreeNode(newRoot);

        root->left = build(root->left, preorder, inorder, left, newPos - 1, pos + 1);
        root->right = build(root->right, preorder, inorder, newPos + 1, right, pos + newPos - left + 1);

        return root;
    }

    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        len = preorder.size();
        TreeNode* root;
        if (!len) return root;
        return build(root, preorder, inorder, 0, len - 1, 0);
    }
};
```



### 剑指 Offer 11. 旋转数组的最小数字

> 把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  

O(n)的算法很容易想到，但是只能超过40%的人，所以得二分，判断边界点和中间点的大小关系

如果将数组的点在坐标轴上画出来，一定是如下所示：

<img src="https://assets.leetcode-cn.com/solution-static/jianzhi_11/1.png" alt="fig1" style="zoom:20%;" />

因此我们考虑三种情况：

1. `nums[ri] > nums[mid]`如果**右边的比中间的大**，那么**最小值一定在左边**，但是**中间可能是最小值**
2. `nums[ri] < nums[mid]`如果**右边的比中间的小**，那么**最小值一定在右边**，且**中间不是最小值**
3. `nums[ri] == nums[mid]`如果**右边值和中间相等**，那么有两种情况，一是这一串都相等，二是右边先降后升，**左右无法确定**。但是可以确定**最右端点不是最小值（如果这一串都相等，少一个也不影响；如果右边先降后升，降的一定比它小）**

```c++
class Solution {
public:
    int minArray(vector<int>& numbers) {
        int le = 0, ri = numbers.size() - 1;
        while (le < ri){
            int mid = (ri - le) / 2 + le;
            // 如果右边的比中间的大，那么最小值一定在左边，但是中间可能是最小值
            if (numbers[ri] > numbers[mid]){
                ri = mid;
            // 如果右边的比中间的小，那么最小值一定在右边，且中间不是最小值
            } else if (numbers[ri] < numbers[mid]) {
                le = mid + 1;
            // 如果右边值和中间相等，那么有两种情况，一是这一串都相等，二是右边先降后升，左右无法确定
            // 但是可以确定**最右端点不是最小值**
            } else {
                --ri;
            }
        }
        return numbers[ri];
    }
};
```



### 剑指 Offer 12. 矩阵中的路径

> 给定一个 m x n 二维字符网格 board 和一个字符串单词 word 。如果 word 存在于网格中，返回 true ；否则，返回 false 。
>
> 单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

进行的是在较大表中的一次查询，因此不采用前缀树（会超时）

采用dfs的方法，dfs也可以分为两种

1. 修改原表：每次比对完后将字符更改为'\0'，这样就不需要vis数组，临界条件也比较简单，可以放在dfs函数第一行（如果最终表要返回原来状态，可以用值传递）
2. 不修改原表：需要用到vis数组判断一个数是否已经被比对过，但这样需要把i和j跑出边界的条件移到`vis[i][j]`之前，否则可能会发生溢出；或者是在调用函数之前判断，但是这样递归终止条件也要进行相应改变

```c++
class Solution {
private:
    int rows, cols;
    size_t len;
    vector<vector<int>> dirs = {
        {-1, 0}, {1, 0}, {0, 1}, {0, -1}
    };
public:
    vector<vector<int>> vis;
    bool dfs(vector<vector<char>>& board, int i, int j, string word, int pos){
        char s = board[i][j];
        if (pos == len - 1 && word[pos] == s) return 1;
        if (s != word[pos]) return 0;
        
        vis[i][j] = 1;
        bool flag = 0;
        for (auto& dir : dirs){
            int ii = i + dir[0], jj = j + dir[1];
            if (ii < 0 || jj < 0 || ii == rows || jj == cols) continue;
            if (!vis[ii][jj] && dfs(board, ii, jj, word, pos + 1)){
                flag = 1; break;
            }
        }
        vis[i][j] = 0;
        return flag;
    }

    bool exist(vector<vector<char>>& board, string word) {
        rows = board.size();
        cols = board[0].size();
        len = word.size();
        vis.resize(rows, vector<int>(cols));
        for (int i = 0; i < rows; ++i){
            for (int j = 0; j < cols; ++j){
                if (board[i][j] == word[0])
                    if (dfs(board, i, j, word, 0)) return 1;
            }
        }
        return 0;
    }
};
```



### 剑指 Offer 13. 机器人的运动范围

> 地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

一个简单的搜索问题，要注意的是，机器人不需要往里走（往回走），因为往外走会变小的唯一情况是个位数为9的变成了0，而这种情况要么是由里面的点往外扩张出来的，要么是机器人走不到的（不连续），因此只要往外走即可

另外：判断条件放在递归式前面而不是递归终止的地方运行会快很多

```c++
class Solution {
private:
    int rows, cols, limit;
    vector<vector<int>> vis;
    vector<vector<int>> dirs = {
        {1, 0}, {0, 1}
    };
public:
    int ok(int i, int j){
        int sum = 0;
        while (i){
            sum += i % 10;
            i /= 10;
        }
        while (j){
            sum += j % 10;
            j /= 10;
        }
        if (sum > limit) return 0;
        return 1;
    }

    int dfs(int i, int j){
        if (i < 0 || j < 0 || i == rows || j == cols) return 0;
        if (vis[i][j]) return 0;
        int sum = 1;
        vis[i][j] = 1;
        for (auto& dir : dirs){
            if (ok(i + dir[0], j + dir[1])) sum += dfs(i + dir[0], j + dir[1]);
        }
        return sum;
    }

    int movingCount(int m, int n, int k) {
        rows = m;
        cols = n;
        vis.resize(m, vector<int>(n));
        limit = k;
        return dfs(0, 0);
    }
};
```



### 剑指 Offer 16. 数值的整数次方

> 实现 pow(x, n)，即计算 x 的 n 次幂函数。不得使用库函数，同时不需要考虑大数问题。

x的n次幂，**x取double，n取long long**。虽然题目中n的限制是int范围，但是取最小负数的绝对值是溢出的，计算负数次幂就会炸

另外，可以特判三种情况`n == 0`、`x == 1`和`x == -1`

```c++
class Solution {
public:
    double myPow(double x, int nn) {
        if (!nn) return 1;
        long long n = nn;
        int flag = n > 0 ? 1 : 0;
        double res = 1;
        n = fabs(n);
        while (n){
            if (n & 1){
                res *= x;
                --n;
            } else {
                x *= x;
                n /= 2;
            }
        }
        return flag ? res : 1 / res;
    }
};
```



### 剑指 Offer 26. 树的子结构

> 输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)
>
> B是A的子结构， 即 A中有出现和B相同的结构和节点值。

总卡一个样例，换个写法卡一个，再换个卡另一个，要考虑的情况挺多的（因为存在重复数字），一开始有点难考虑全，下面列出几种情况

- 普通的子结构
- B是A中的一部分，例如A的右子树还有东西，而B已经没有了，用`if (!A && !B || A && !B) return 1;`这句代码解决
- 数字与B的根数字相等，但是不构成子结构；后面存在一个重复数字，能构成子结构，用`int fix = 0`解决（fix==1即要求这个根为子结构的根）

```c++
class Solution {
public:
    bool func(TreeNode* A, TreeNode* B, int fix = 0){
        if (!A && !B || A && !B) return 1;
        if (B && !A) return 0;
        bool sub = func(A->left, B) || func(A->right, B);
        if (A->val != B->val){
            if (fix) return 0;
            return sub;
        } else {
            return func(A->left, B->left, 1) && func(A->right, B->right, 1) || sub;
        }
    }

    bool isSubStructure(TreeNode* A, TreeNode* B) {
        if (!B) return 0;
        return func(A, B);
    }
};
```



### 剑指 Offer 27. 二叉树的镜像

> 请完成一个函数，输入一个二叉树，该函数输出它的镜像。

- **空间换时间**：定义新的root，在root上进行赋值，这样就不用执行交换了
- **时间换空间**：通过交换操作在原二叉树上进行

```c++
// 时间换空间
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root) {
        if (!root) return root;
        TreeNode* temp = root->left;
        root->left = root->right;
        root->right = temp;
        root->left = mirrorTree(root->left);
        root->right = mirrorTree(root->right);
        return root;
    }
};

// 空间换时间
class Solution {
public:
    TreeNode* create(TreeNode* root, TreeNode* newRoot){
        if (!root) return root;
        newRoot = new TreeNode(root->val);
        newRoot->right = create(root->left, newRoot->right);
        newRoot->left = create(root->right, newRoot->left);
        return newRoot;
    }

    TreeNode* mirrorTree(TreeNode* root) {
        if (!root) return root;
        TreeNode* newRoot = new TreeNode(root->val);
        return create(root, newRoot);
    }
};
```



### 剑指 Offer 29. 顺时针打印矩阵

> 输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。

模拟，但是模拟也容易出错，因为定义上下左右四个变量太多了，而且最后结束的条件不好判断，因此最后应该用pos计数矩阵中所有元素的数量，但是这样要在循环体内增加三个判断语句，暂未找到好方法

```c++
class Solution {
private:
    int urow, lcol, drow, rcol, pos, n;
    vector<int> res;
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        drow = matrix.size() - 1;
        if (drow == -1) return res;
        rcol = matrix[0].size() - 1;
        if (rcol == -1) return res;
        urow = lcol = pos = 0;
        n = (drow + 1) * (rcol + 1);
        res.resize(n);
        
        while (pos < n){
            for (int i = lcol; i <= rcol; ++i) res[pos++] = matrix[urow][i];
            if (pos == n) break;
            ++urow;
            for (int i = urow; i <= drow; ++i) res[pos++] = matrix[i][rcol];
            if (pos == n) break;
            --rcol;
            for (int i = rcol; i >= lcol; --i) res[pos++] = matrix[drow][i];
            if (pos == n) break;
            --drow;
            for (int i = drow; i >= urow; --i) res[pos++] = matrix[i][lcol];
            ++lcol;
        }
        return res;
    }
};
```



### 剑指 Offer 59 - II. 队列的最大值

> 请定义一个队列并实现函数 max_value 得到队列里的最大值，要求函数max_value、push_back 和 pop_front 的均摊时间复杂度都是O(1)。
>
> 若队列为空，pop_front 和 max_value 需要返回 -1

实际上是一个**队列**加上一个**双端队列**完成。
队列用于保存所有元素；双端队列用于保存可能的最大值

双端队列维护的是一个单减序列，因为**位于较大值左边的较小值是没有意义的**，类似于滑动窗口

```c++
class MaxQueue {
private:
    int* head;
    int* tail;
    // 维护单减序列（位于较大值左边的较小值是没有意义的）
    int* maxHead;
    int* maxTail;
public:
    MaxQueue() {
        head = tail = new int[10000];
        maxHead = maxTail = new int[10000];
    }
    
    int max_value() {
        if (maxHead == maxTail) return -1;
        return *maxHead;
    }
    
    void push_back(int value) {
        *tail++ = value;
        while (maxTail != maxHead && value > *(maxTail - 1)){
            maxTail--;
        }
        *maxTail++ = value;
    }
    
    int pop_front() {
        if (head == tail) return -1;
        if (*head == *maxHead) maxHead++;
        return *head++;
    }
};
```



### 剑指 Offer 30. 包含min函数的栈

> 定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)

与上一题类似，分析这种**局部max**或者**局部min**值时，最重要的是判断下面一句话：

**位于较X值X边的较X值无意义，所以维护单X序列**

一旦搞清楚这句话，题目就好做了

```c++
class MinStack {
private:
    // 位于较小值右边的较大值无意义，所以维护单减序列
    stack<int> s, mini;

public:
    MinStack() {

    }
    
    void push(int x) {
        s.push(x);
        if (mini.empty() || x <= mini.top()){
            mini.push(x);
        }
    }
    
    void pop() {
        int v = s.top();
        if (v == mini.top()){
            mini.pop();
        }
        s.pop();
    }
    
    int top() {
        return s.top();
    }
    
    int min() {
        return mini.top();
    }
};
```



### 剑指 Offer 31. 栈的压入、弹出序列

> 输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如，序列 {1,2,3,4,5} 是某栈的压栈序列，序列 {4,5,3,2,1} 是该压栈序列对应的一个弹出序列，但 {4,3,5,1,2} 就不可能是该压栈序列的弹出序列。

模拟便能得到答案，需要注意的一点是：当元素全部push进栈后，如果执行一遍pop操作仍未全弹出，那么就可以得出这个popped无法实现的结论

***// TODO 能不能用数学方法？***

```c++
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
        int pos = 0;
        stack<int> s;
        for (int num : pushed){
            s.push(num);
            while (!s.empty() && s.top() == popped[pos]){
                s.pop(); ++pos;
            }
        }
        return s.empty();
    }
};
```



### 剑指 Offer 32 - II. 从上到下打印二叉树 II

> 从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，**每一层打印到一行**。

1. 首先想到的是增加一个辅助队列，每一层将下一层要计数的结点push进辅助队列中，再把辅助队列赋值给原队列，得到结果，但是费时费空间
2. 用**计数**的方法，注意到每次q的大小都是该层的结点数量，所以每层都只需要q.size()次循环即可，这样就只需要一个队列了

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        if (!root) return res;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()){
            int count = q.size();
            vector<int> level(count);
            for (int i = 0; i < count; ++i){
                TreeNode* t = q.front();
                level[i] = t->val;
                if (t->left) q.push(t->left);
                if (t->right) q.push(t->right);
                q.pop();
            }
            res.emplace_back(level);
        }
        return res;
    }
};
```



### 剑指 Offer 32 - III. 从上到下打印二叉树 III

> 请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

使用**双端队列**，每次从前往后遍历时使用push_back，从后往前遍历时使用push_front，否则队头队尾数据可能会被push进来的数据替换，导致输出错误

**误区**：从右到左打印不能简单的反转每次push的顺序，而是整个层队列的反转！

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        if (!root) return res;
        deque<TreeNode*> q;
        q.push_back(root);
        bool flag = 1;
        while (!q.empty()){
            int len = q.size();
            vector<int> level(len);
            if (flag){
                for (int i = 0; i < len; ++i){
                    TreeNode* t = q.front();
                    level[i] = t->val;
                    if (t->left) q.push_back(t->left);
                    if (t->right) q.push_back(t->right);
                    q.pop_front();
                }
            } else {
                for (int i = 0; i < len; ++i){
                    TreeNode* t = q.back();
                    level[i] = t->val;
                    if (t->right) q.push_front(t->right);
                    if (t->left) q.push_front(t->left);
                    q.pop_back();
                }
            }
            flag ^= 1;
            res.emplace_back(level);
        }
        return res;
    }
};
```



### *剑指 Offer 33. 二叉搜索树的后序遍历序列

> 输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 `true`，否则返回 `false`。假设输入的数组的任意两个数字都互不相同。

1. 判断是否是后序遍历：使用递归的方法，找到左右子树分界点**（第一个小于根节点的点往左就是左子树，往右除根外就是右子树）**，分别进行递归
2. 判断是否是搜索二叉树：用mini和maxi递归框定每一个结点的取值范围，**左子树为【mini，该根节点大小】，右子树为【该根节点大小，maxi】**

```c++
class Solution {
public:
    bool isPost(vector<int>& postorder, int low, int high, int mini = INT_MIN, int maxi = INT_MAX){
        if (low >= high) return 1;
        for (int i = high; i >= low; --i){
            if (postorder[i] < mini || postorder[i] > maxi) return 0;
            if (postorder[i] < postorder[high]){
                return isPost(postorder, low, i, mini, postorder[high]) && 
                       isPost(postorder, i + 1, high - 1, postorder[high], maxi);
            }
        }
        return isPost(postorder, low, high - 1, postorder[high], maxi);
    }

    bool verifyPostorder(vector<int>& postorder) {
        size_t len = postorder.size();
        if (!len) return 1;
        return isPost(postorder, 0, len - 1);
    }
};
```



### 剑指 Offer 34. 二叉树中和为某一值的路径

> 给你二叉树的根节点 `root` 和一个整数目标和 `targetSum` ，找出所有 **从根节点到叶子节点** 路径总和等于给定目标和的路径。

简单的回溯，需要注意题目要求的终止条件是遇到叶子节点且满足target

```c++
class Solution {
public:
    vector<vector<int>> res;
    void dfs(TreeNode* root, int target, vector<int>& path){
        if (!root) return;
        path.emplace_back(root->val);
        if (target == root->val && !root->left && !root->right){
            res.emplace_back(path);
        }
        dfs(root->left, target - root->val, path);
        dfs(root->right, target - root->val, path);
        path.pop_back();
    }

    vector<vector<int>> pathSum(TreeNode* root, int target) {
        vector<int> path;
        dfs(root, target, path);
        return res;
    }
};
```



### 剑指 Offer 35. 复杂链表的复制

> 请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

难点在于如何确定新链表和原链表的结点对应关系

可以用一个哈希表创造一一对应，若表中没有，则创建新结点，表中有则用表中的，最后递归求解

```c++
class Solution {
public:
    // 原链表和新链表的结点对应关系
    unordered_map<Node*, Node*> mp;
    Node* copyRandomList(Node* head) {
        if (!head) return head;

        if (!mp.count(head)){
            Node* temp = new Node(head->val);
            mp[head] = temp;
            temp->next = copyRandomList(head->next);
            temp->random = copyRandomList(head->random);
        }
        return mp[head];
    }
};
```



### *剑指 Offer 36. 二叉搜索树与双向链表

> 输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

中序遍历，把应该输出值的语句换成连接链表的语句，pre和当前root连

注意：不能连right！因为right可能已经改位置了，并不是连续的右子树

```c++
class Solution {
public:
    Node* pre = nullptr;
    Node* first = nullptr;
    void dfs(Node* root){
        if (!root) return;

        dfs(root->left);
        // pre和first只有一开始全为空，first也只需要初始化一次
        if (pre) pre->right = root;
        else first = root;
        // 把pre和root连起来
        root->left = pre;
        // 更新pre
        pre = root;

        dfs(root->right);
        return;
    }

    Node* treeToDoublyList(Node* root) {
        if (!root) return root;
        dfs(root);
        first->left = pre;
        pre->right = first;
        return first;
    }
};
```



### 剑指 Offer 38. 字符串的排列

> 输入一个字符串，打印出该字符串中字符的所有排列。
>
> 你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。

解法为回溯，需要注意的是字符串中的**重复元素**，需要保证重复元素**按顺序**计数，这样就不会得到重复结果

1. 对数组排序

2. 设一个vis数组

3. 判断是否重复`i > 0 && s[i] == s[i - 1]`

4. 判断是否按顺序`&& vis[i - 1] == 0`表示前面重复的还没有取，因此是不合理的

注意：

- 由于存在循环，`vis[i] = 1`这句话要放在循环中
- 顺序不对时只需要使用continue，而不是return，因为后面仍然有可能按顺序取前面的，**这是循环，不是单向的！**
- 每次都是再循环中添加temp的值，所以i == pos时不需要特殊考虑

当然，C++里有一个库函数叫做`next_permutation();`，可以直接用这个

```c++
class Solution {
public:
    size_t len;
    int vis[8] = {0};
    vector<string> res;
    string temp;
    void dfs(string& s, int pos){
        if (pos == len){
            res.emplace_back(temp);
            return;
        }
        for (int i = 0; i < len; ++i){
            // 相同的要保证顺序
            if (vis[i] || i > 0 && s[i] == s[i - 1] && vis[i - 1] == 0) continue;

            temp += s[i];
            vis[i] = 1;
            dfs(s, pos + 1);
            vis[i] = 0;
            temp = temp.substr(0, temp.size() - 1);
        }
    }

    vector<string> permutation(string s) {
        len = s.size();
        sort(s.begin(), s.end());
        dfs(s, 0);
        return res;
    }
};
```



### 剑指 Offer 39. 数组中出现次数超过一半的数字

> 数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。
>
> 你可以假设数组是非空的，并且给定的数组总是存在多数元素。

摩尔投票法，两个不同的数字可以相互抵消，最后剩下的就是出现过半的数字

**如果改为超过三分之一呢？**

那就是每三个不同的数进行一次抵消

```c++
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        size_t len = nums.size();
        int fre[2] = {0};
        fre[0] = nums[0];
        fre[1] = 1;
        for (int i = 1; i < len; ++i){
            if (nums[i] == fre[0]){
                fre[1]++;
            } else {
                if (fre[1] == 0){
                    fre[0] = nums[i];
                    fre[1]++;
                } else {
                    fre[1]--;
                }
            }
        }
        return fre[0];
    }
};
```



### 剑指 Offer 40. 最小的k个数

> 输入整数数组 `arr` ，找出其中最小的 `k` 个数。例如，输入4、5、1、6、2、7、3、8这8个数字，则最小的4个数字是1、2、3、4。

**快排还是重要啊！**

1. partition中，`--high`后赋值是`arr[low] = arr[high]`，因为要把low的覆盖
2. 注意pivitloc和k的关系，一定要用`pivotloc - low + 1`和`k`作比较
3. 排序时pivotloc位置的数不需要参与

```c++
class Solution {
public:
    size_t len;

    int partition(vector<int>&arr, int low, int high){
        int temp = arr[low];
        while (low < high){
            while (low < high && arr[high] >= temp) --high;
            arr[low] = arr[high];
            while (low < high && arr[low] <= temp) ++low;
            arr[high] = arr[low];
        }
        arr[low] = temp;
        return low;
    }

    void qsort(vector<int>& arr, int k, int low, int high){
        if (low < high){
            int pivotloc = partition(arr, low, high);
            int num = pivotloc - low + 1;
            if (num < k){
                qsort(arr, k - num, pivotloc + 1, high);
            } else if (num > k){
                qsort(arr, k, low, pivotloc - 1);
            } else return;
        }
    }

    vector<int> getLeastNumbers(vector<int>& arr, int k) {
        len = arr.size();
        qsort(arr, k, 0, len - 1);
        vector<int> res(arr.begin(), arr.begin() + k);
        return res;
    }
};
```

