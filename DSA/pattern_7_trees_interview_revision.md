# Pattern 7: Trees — C++ Interview Revision

This file is made from our chat explanations. The goal is not to memorize code blindly, but to understand the **pattern trigger**, the **interview explanation**, and the **clean C++ solution**.

---

## TreeNode Structure Used by LeetCode

Most LeetCode tree questions already provide this structure:

```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};
```

---

# 1. Basic Tree Pattern Mindset

For most tree questions, ask:

> At each node, what information do I need from the left child and right child?

Tree problems mostly use:

## DFS / Recursion

Use DFS when the question talks about:

- depth / height
- diameter
- balanced tree
- same tree
- subtree
- path sum
- validate BST
- build tree
- serialize / deserialize

Basic DFS template:

```cpp
int dfs(TreeNode* root) {
    if (!root) return base_value;

    int left = dfs(root->left);
    int right = dfs(root->right);

    return something_using_left_right;
}
```

## BFS / Queue

Use BFS when the question talks about:

- level order
- right side view
- nodes grouped by depth

Basic BFS template:

```cpp
queue<TreeNode*> q;
q.push(root);

while (!q.empty()) {
    int size = q.size();

    for (int i = 0; i < size; i++) {
        TreeNode* node = q.front();
        q.pop();

        if (node->left) q.push(node->left);
        if (node->right) q.push(node->right);
    }
}
```

---

# Problem 43: Invert Binary Tree

## Pattern Trigger

Need to mirror every subtree.

At every node:

```text
swap left and right child
```

## Interview Explanation

I will solve this using recursion. For every node, I will swap its left and right child. Then I will recursively do the same for its left subtree and right subtree. If the node is null, I simply return null.

## Code

```cpp
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if (!root) return nullptr;

        swap(root->left, root->right);

        invertTree(root->left);
        invertTree(root->right);

        return root;
    }
};
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(h)
```

`n` is the number of nodes and `h` is the height of the tree.

## Key Interview Line

Because the recursion visits every node once and swaps left and right child in constant time, the time complexity is O(n).

---

# Problem 44: Maximum Depth of Binary Tree

## Pattern Trigger

Need height or maximum depth of a binary tree.

At every node:

```text
height = 1 + max(left height, right height)
```

## Interview Explanation

I will use DFS recursion. For a null node, depth is 0. For every non-null node, I calculate the depth of the left subtree and right subtree, then return 1 plus the maximum of both.

## Code

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) return 0;

        int leftDepth = maxDepth(root->left);
        int rightDepth = maxDepth(root->right);

        return 1 + max(leftDepth, rightDepth);
    }
};
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(h)
```

## Key Interview Line

The height of a tree is the current root node plus the maximum height from the left or right subtree.

---

# Problem 45: Diameter of Binary Tree

## Pattern Trigger

Need longest path between any two nodes.

At every node:

```text
possible diameter = left height + right height
```

## Important Point

Diameter is measured in **edges** on LeetCode.

## Interview Explanation

I will use DFS. For each node, I calculate the height of its left and right subtree. The longest path passing through that node is left height plus right height. I update a global answer with this value. Finally, the DFS returns 1 plus the maximum of left and right height to its parent.

## Code

```cpp
class Solution {
    int best = 0;

public:
    int diameterOfBinaryTree(TreeNode* root) {
        height(root);
        return best;
    }

private:
    int height(TreeNode* node) {
        if (!node) return 0;

        int left = height(node->left);
        int right = height(node->right);

        best = max(best, left + right);

        return 1 + max(left, right);
    }
};
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(h)
```

## Key Interview Line

For diameter, we use `left + right` because the path can pass through both sides of the current node. But we return `1 + max(left, right)` because the parent can continue through only one side.

---

# Problem 46: Balanced Binary Tree

## Pattern Trigger

Need every subtree's left and right heights to differ by at most 1.

## Main Idea

Return height normally.

Return `-1` if the subtree is unbalanced.

Why `-1`?

Because height is never negative, so `-1` can safely mean:

```text
this subtree is already unbalanced
```

## Interview Explanation

I will use DFS recursion. For every node, I calculate the height of the left and right subtree. If any subtree is already unbalanced, I return -1. If the height difference between left and right is greater than 1, I also return -1. Otherwise, I return the normal height of the current subtree.

## Code

```cpp
class Solution {
public:
    bool isBalanced(TreeNode* root) {
        return height(root) != -1;
    }

private:
    int height(TreeNode* node) {
        if (!node) return 0;

        int left = height(node->left);
        if (left == -1) return -1;

        int right = height(node->right);
        if (right == -1) return -1;

        if (abs(left - right) > 1) return -1;

        return 1 + max(left, right);
    }
};
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(h)
```

## Key Interview Line

We return `-1` as an early failure signal when any subtree is unbalanced.

---

# Problem 47: Same Tree

## Pattern Trigger

Need to compare two trees for structure and value equality.

## Interview Explanation

I will use DFS recursion on both trees at the same time. If both nodes are null, I return true. If only one node is null, the structure is different, so I return false. If both nodes exist, I compare their values and recursively compare their left and right subtrees.

## Clean Code

```cpp
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        if (!p || !q) return p == q;

        return p->val == q->val &&
               isSameTree(p->left, q->left) &&
               isSameTree(p->right, q->right);
    }
};
```

## Beginner-Friendly Code

```cpp
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        if (p == NULL && q == NULL) return true;

        if (p == NULL || q == NULL) return false;

        if (p->val != q->val) return false;

        bool leftSame = isSameTree(p->left, q->left);
        bool rightSame = isSameTree(p->right, q->right);

        return leftSame && rightSame;
    }
};
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(h)
```

## Key Interview Line

Same Tree checks both value equality and structure equality.

---

# Problem 48: Subtree of Another Tree

## Pattern Trigger

Need to check whether one tree exists inside another tree.

Think:

```text
Same Tree + DFS search
```

## Interview Explanation

I will traverse the main tree using DFS. At every node, I will check whether the tree starting from that node is exactly the same as `subRoot` using a helper function similar to Same Tree. If it matches, I return true. Otherwise, I recursively search in the left and right subtree.

## Code

```cpp
class Solution {
public:
    bool isSubtree(TreeNode* root, TreeNode* subRoot) {
        if (!root) return false;

        if (same(root, subRoot)) return true;

        return isSubtree(root->left, subRoot) ||
               isSubtree(root->right, subRoot);
    }

private:
    bool same(TreeNode* a, TreeNode* b) {
        if (!a || !b) return a == b;

        return a->val == b->val &&
               same(a->left, b->left) &&
               same(a->right, b->right);
    }
};
```

## Complexity

```text
Time Complexity: O(m * n) worst case
Space Complexity: O(h)
```

Where:

```text
m = number of nodes in root
n = number of nodes in subRoot
```

## Key Interview Line

Subtree problem is DFS search plus Same Tree check.

---

# Problem 49: Lowest Common Ancestor of a BST

## Pattern Trigger

Question gives a BST and asks for LCA.

Use BST property:

```text
left < root < right
```

## Main Idea

At current root:

1. If both `p` and `q` are smaller, move left.
2. If both `p` and `q` are greater, move right.
3. Otherwise, they split around current node, so current node is LCA.

## Interview Explanation

Since this is a BST, I can use the ordering property. If both `p` and `q` are smaller than the current node, I move left. If both are greater, I move right. The moment they split, or one of them equals the current node, the current node is the lowest common ancestor.

## Code

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        while (root) {
            if (p->val < root->val && q->val < root->val) {
                root = root->left;
            }
            else if (p->val > root->val && q->val > root->val) {
                root = root->right;
            }
            else {
                return root;
            }
        }

        return nullptr;
    }
};
```

## Complexity

```text
Time Complexity: O(h)
Space Complexity: O(1)
```

Balanced BST:

```text
O(log n)
```

Skewed BST:

```text
O(n)
```

## Key Interview Line

We do not need to check every node because BST ordering tells us which side contains both nodes.

---

# Problem 50: Binary Tree Level Order Traversal

## Pattern Trigger

Question says:

```text
level order
level by level
nodes grouped by depth
```

So use BFS with queue.

## Important Trick

Before processing a level:

```cpp
int size = q.size();
```

This tells us how many nodes are in the current level.

## Interview Explanation

I will use BFS with a queue. Since we need nodes level by level, I process the tree level-wise. For every level, I first store the current queue size because that represents the number of nodes in the current level. Then I pop exactly that many nodes, store their values, and push their children for the next level.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        if (!root) return {};

        vector<vector<int>> ans;
        queue<TreeNode*> q;

        q.push(root);

        while (!q.empty()) {
            int size = q.size();
            vector<int> level;

            for (int i = 0; i < size; i++) {
                TreeNode* node = q.front();
                q.pop();

                level.push_back(node->val);

                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
            }

            ans.push_back(level);
        }

        return ans;
    }
};
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(w)
```

`w` is the maximum width of the tree.

## Key Interview Line

Queue size before the loop gives the number of nodes in the current level.

---

# Problem 51: Binary Tree Right Side View

## Pattern Trigger

Need visible node from the right side at each level.

Think:

```text
Level order traversal + last node of every level
```

## Interview Explanation

I will use BFS level order traversal. Since the right side view contains the last node of every level, I will process each level using the current queue size. While processing that level, when I reach the last node, I add it to the answer. Then I continue pushing children for the next level.

## Code

```cpp
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        if (!root) return {};

        vector<int> ans;
        queue<TreeNode*> q;

        q.push(root);

        while (!q.empty()) {
            int size = q.size();

            for (int i = 0; i < size; i++) {
                TreeNode* node = q.front();
                q.pop();

                if (i == size - 1) {
                    ans.push_back(node->val);
                }

                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
            }
        }

        return ans;
    }
};
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(w)
```

## Key Interview Line

We check `i == size - 1` because the last node processed at each level is visible from the right side.

---

# Problem 52: Count Good Nodes in Binary Tree

## Pattern Trigger

Need to compare the current node with the maximum value seen from root to that node.

Think:

```text
DFS + carry max value in path
```

## Main Idea

Pass one variable:

```cpp
maxSoFar
```

At each node:

```cpp
if (node->val >= maxSoFar) good node
```

Then update:

```cpp
maxSoFar = max(maxSoFar, node->val);
```

## Interview Explanation

I will use DFS and carry the maximum value seen from the root to the current node. If the current node value is greater than or equal to this maximum, then it is a good node. Then I update the maximum value and continue DFS for the left and right subtrees.

## Code

```cpp
class Solution {
public:
    int goodNodes(TreeNode* root) {
        return dfs(root, INT_MIN);
    }

private:
    int dfs(TreeNode* node, int maxSoFar) {
        if (!node) return 0;

        int good = 0;

        if (node->val >= maxSoFar) {
            good = 1;
        }

        maxSoFar = max(maxSoFar, node->val);

        int leftGood = dfs(node->left, maxSoFar);
        int rightGood = dfs(node->right, maxSoFar);

        return good + leftGood + rightGood;
    }
};
```

## Short Code

```cpp
class Solution {
public:
    int goodNodes(TreeNode* root) {
        return dfs(root, INT_MIN);
    }

private:
    int dfs(TreeNode* node, int best) {
        if (!node) return 0;

        int good = node->val >= best;

        best = max(best, node->val);

        return good + dfs(node->left, best) + dfs(node->right, best);
    }
};
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(h)
```

## Key Interview Line

This problem needs the maximum value in the current root-to-node path, not the maximum of the whole tree.

---

# Problem 53: Validate Binary Search Tree

## Pattern Trigger

Need to validate a BST.

Think:

```text
DFS with lower and upper bounds
```

## Why Bounds Are Needed

Only checking direct children is not enough.

Example:

```text
        5
       / \
      1   7
         / \
        4   8
```

Node `4` is smaller than `7`, so locally it looks okay.

But `4` is in the right subtree of `5`, so it must be greater than `5`.

Since `4 < 5`, this is invalid.

## Interview Explanation

I will use DFS with a valid range for every node. Initially, the range is negative infinity to positive infinity. For every node, its value must be strictly greater than the lower bound and strictly smaller than the upper bound. For the left child, the upper bound becomes the current node value. For the right child, the lower bound becomes the current node value.

## Code

```cpp
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        return valid(root, LLONG_MIN, LLONG_MAX);
    }

private:
    bool valid(TreeNode* node, long long low, long long high) {
        if (!node) return true;

        if (node->val <= low || node->val >= high) {
            return false;
        }

        return valid(node->left, low, node->val) &&
               valid(node->right, node->val, high);
    }
};
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(h)
```

## Key Interview Line

We use `low` and `high` because every node must satisfy the range created by all its ancestors, not just its direct parent.

## Important Detail

Use `long long` bounds because node values may be `INT_MIN` or `INT_MAX`.

---

# Problem 54: Kth Smallest Element in a BST

## Pattern Trigger

Question says:

```text
BST + kth smallest
```

Think:

```text
Inorder traversal
```

Because inorder traversal of BST gives sorted values.

## Interview Explanation

Since this is a BST, inorder traversal gives values in sorted increasing order. So I will perform inorder traversal and decrement `k` whenever I visit a node. When `k` becomes zero, the current node is the kth smallest element.

## Recursive Code

```cpp
class Solution {
    int count = 0;
    int answer = -1;

public:
    int kthSmallest(TreeNode* root, int k) {
        inorder(root, k);
        return answer;
    }

private:
    void inorder(TreeNode* node, int k) {
        if (!node) return;

        inorder(node->left, k);

        count++;
        if (count == k) {
            answer = node->val;
            return;
        }

        inorder(node->right, k);
    }
};
```

## Iterative Code

```cpp
class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        stack<TreeNode*> st;

        while (root || !st.empty()) {
            while (root) {
                st.push(root);
                root = root->left;
            }

            root = st.top();
            st.pop();

            k--;
            if (k == 0) return root->val;

            root = root->right;
        }

        return -1;
    }
};
```

## Complexity

```text
Time Complexity: O(h + k)
Worst Case: O(n)
Space Complexity: O(h)
```

## Key Interview Line

Inorder traversal of a BST gives sorted order because it visits left subtree first, then root, then right subtree.

---

# Problem 55: Construct Binary Tree from Preorder and Inorder

## Pattern Trigger

Need to build tree from preorder and inorder.

Remember:

```text
Preorder picks the root.
Inorder divides the tree.
```

## Traversal Meaning

Preorder:

```text
root -> left -> right
```

Inorder:

```text
left -> root -> right
```

## Interview Explanation

I will use preorder to identify the root because preorder always visits root first. Then I will find that root in inorder. Everything on the left side of the root in inorder belongs to the left subtree, and everything on the right side belongs to the right subtree. I will repeat this recursively for both subtrees.

## Code

```cpp
class Solution {
    unordered_map<int, int> inIndex;
    int preIndex = 0;

public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        for (int i = 0; i < inorder.size(); i++) {
            inIndex[inorder[i]] = i;
        }

        return build(preorder, 0, inorder.size() - 1);
    }

private:
    TreeNode* build(vector<int>& preorder, int left, int right) {
        if (left > right) return nullptr;

        int rootVal = preorder[preIndex];
        preIndex++;

        TreeNode* root = new TreeNode(rootVal);

        int mid = inIndex[rootVal];

        root->left = build(preorder, left, mid - 1);
        root->right = build(preorder, mid + 1, right);

        return root;
    }
};
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(n)
```

## Key Interview Line

Preorder gives the root, and inorder tells which nodes belong to the left and right subtree.

## Why Use Hash Map?

Without a hashmap, finding root index in inorder takes O(n) each time.

With hashmap:

```text
root index lookup = O(1)
```

---

# Problem 56: Binary Tree Maximum Path Sum

## Pattern Trigger

Need best path that can start and end anywhere.

Think:

```text
DFS + global maximum
```

## Main Idea

At every node, two things are different:

## 1. Answer candidate at current node

The best path through current node can use both sides:

```cpp
node->val + left + right
```

## 2. Return value to parent

To the parent, we can return only one side:

```cpp
node->val + max(left, right)
```

Because a path going upward cannot split into both left and right.

## Why Ignore Negative Paths?

Use:

```cpp
int left = max(0, gain(node->left));
int right = max(0, gain(node->right));
```

Negative paths reduce the sum, so we ignore them.

## Interview Explanation

I will use DFS. For every node, I calculate the maximum gain from the left and right subtree. If any gain is negative, I ignore it by taking max with 0. The maximum path through the current node is node value plus left gain plus right gain, so I update a global answer. But while returning to the parent, I can only return one path, so I return node value plus the maximum of left and right gain.

## Code

```cpp
class Solution {
    int best = INT_MIN;

public:
    int maxPathSum(TreeNode* root) {
        gain(root);
        return best;
    }

private:
    int gain(TreeNode* node) {
        if (!node) return 0;

        int left = max(0, gain(node->left));
        int right = max(0, gain(node->right));

        best = max(best, node->val + left + right);

        return node->val + max(left, right);
    }
};
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(h)
```

## Key Interview Line

We update the answer using both sides, but return only one side to the parent.

---

# Problem 57: Serialize and Deserialize Binary Tree

## Pattern Trigger

Need to convert a tree into a string and rebuild the exact same tree.

Think:

```text
Preorder traversal + null markers
```

## Why Null Markers Are Needed

Without null markers, these two trees may look the same:

```text
Tree 1:        Tree 2:

   1              1
  /                \
 2                  2
```

Both may look like:

```text
1,2
```

But with null markers:

```text
Tree 1: 1,2,#,#,#,
Tree 2: 1,#,2,#,#,
```

Now the structure is clear.

## Interview Explanation

I will use preorder traversal for serialization. For every node, I store its value, and for every null child, I store a special marker like `#`. This null marker is important because it preserves the exact structure of the tree. During deserialization, I read tokens in the same preorder order. If the token is `#`, I return null. Otherwise, I create a node and recursively build its left and right children.

## Code

```cpp
class Codec {
public:
    string serialize(TreeNode* root) {
        string result;
        write(root, result);
        return result;
    }

    TreeNode* deserialize(string data) {
        stringstream ss(data);
        return read(ss);
    }

private:
    void write(TreeNode* node, string& result) {
        if (!node) {
            result += "#,";
            return;
        }

        result += to_string(node->val) + ",";

        write(node->left, result);
        write(node->right, result);
    }

    TreeNode* read(stringstream& ss) {
        string token;
        getline(ss, token, ',');

        if (token == "#") {
            return nullptr;
        }

        TreeNode* node = new TreeNode(stoi(token));

        node->left = read(ss);
        node->right = read(ss);

        return node;
    }
};
```

## Understanding the `read()` Function

Example serialized string:

```text
1,2,#,#,3,#,#,
```

The function reads tokens one by one.

### Step 1: Read one token

```cpp
string token;
getline(ss, token, ',');
```

This reads one value until comma.

### Step 2: If token is `#`, return null

```cpp
if (token == "#") {
    return nullptr;
}
```

This means there is no node here.

### Step 3: If token is a number, create node

```cpp
TreeNode* node = new TreeNode(stoi(token));
```

### Step 4: Build left subtree

```cpp
node->left = read(ss);
```

### Step 5: Build right subtree

```cpp
node->right = read(ss);
```

### Step 6: Return current node

```cpp
return node;
```

## Complexity

```text
Time Complexity: O(n)
Space Complexity: O(n)
```

## Key Interview Line

Serialization needs both values and structure. Null markers preserve the structure.

---

# Final Quick Revision Table

| Problem | Main Pattern | Key Logic |
|---|---|---|
| Invert Binary Tree | DFS | Swap left and right |
| Maximum Depth | DFS | `1 + max(left, right)` |
| Diameter | DFS + global answer | `best = max(best, left + right)` |
| Balanced Tree | DFS height | Return `-1` if unbalanced |
| Same Tree | DFS on two trees | Compare structure and values |
| Subtree of Another Tree | DFS + Same Tree | Try matching at every node |
| LCA of BST | BST property | Move left/right until split |
| Level Order | BFS | Queue + level size |
| Right Side View | BFS | Last node of every level |
| Good Nodes | DFS with path max | Carry `maxSoFar` |
| Validate BST | DFS with bounds | `low < node < high` |
| Kth Smallest BST | Inorder | BST inorder is sorted |
| Build Tree | Preorder + Inorder | Preorder root, inorder split |
| Max Path Sum | DFS + global answer | Return one side, update with both sides |
| Serialize/Deserialize | Preorder + `#` | Store values and null structure |

---

# Final Interview Advice

For every tree question, do not panic. First identify whether it is:

```text
DFS recursion
BFS queue
BST property
Inorder traversal
Global answer with helper function
```

Then explain the idea before writing code. Interviewers care about your thinking, not only the final code.
