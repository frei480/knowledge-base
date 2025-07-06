---
tags:
  - csharp
  - python
---
**Complete guide** to the three binary tree traversal methods:
- **in-order**,
- **pre-order**,
- **post-order**
with both **recursive and iterative** implementations in **Python** and **C#**.

---
# 🔁 1. **In-Order Traversal (Left → Root → Right)**

## ✅ Recursive

### **Python**
```python
def in_order_recursive(node):
    if node:
        in_order_recursive(node.left)
        print(node.val)
        in_order_recursive(node.right)
```

### **C#**
```csharp
void InOrderRecursive(TreeNode node)
{
    if (node != null)
    {
        InOrderRecursive(node.Left);
        Console.WriteLine(node.Val);
        InOrderRecursive(node.Right);
    }
}
```

## 🔁 Iterative
### **Python**
```python
def in_order_iterative(root):
    stack = []
    current = root
    while stack or current:
        while current:
            stack.append(current)
            current = current.left
        current = stack.pop()
        print(current.val)
        current = current.right
```

### **C#**
```csharp
void InOrderIterative(TreeNode root)
{
    Stack<TreeNode> stack = new Stack<TreeNode>();
    TreeNode current = root;
    while (stack.Count > 0 || current != null)
    {
        while (current != null)
        {
            stack.Push(current);
            current = current.Left;
        }
        current = stack.Pop();
        Console.WriteLine(current.Val);
        current = current.Right;
    }
}
```

---
# 🟢 2. **Pre-Order Traversal (Root → Left → Right)**

## ✅ Recursive

### **Python**
```python
def pre_order_recursive(node):
    if node:
        print(node.val)
        pre_order_recursive(node.left)
        pre_order_recursive(node.right)
```

### **C#**

```csharp
void PreOrderRecursive(TreeNode node)
{
    if (node != null)
    {
        Console.WriteLine(node.Val);
        PreOrderRecursive(node.Left);
        PreOrderRecursive(node.Right);
    }
}
```

## 🔁 Iterative

### **Python**

```python
def pre_order_iterative(root):
    if not root:
        return
    stack = [root]
    while stack:
        node = stack.pop()
        print(node.val)
        if node.right:
            stack.append(node.right)
        if node.left:
            stack.append(node.left)
```

### **C#**

```csharp
void PreOrderIterative(TreeNode root)
{
    if (root == null) return;

    Stack<TreeNode> stack = new Stack<TreeNode>();
    stack.Push(root);

    while (stack.Count > 0)
    {
        TreeNode node = stack.Pop();
        Console.WriteLine(node.Val);

        if (node.Right != null) stack.Push(node.Right);
        if (node.Left != null) stack.Push(node.Left);
    }
}
```

---

# 🔴 3. **Post-Order Traversal (Left → Right → Root)**

## ✅ Recursive

### **Python**

```python
def post_order_recursive(node):
    if node:
        post_order_recursive(node.left)
        post_order_recursive(node.right)
        print(node.val)
```

### **C#**

```csharp
void PostOrderRecursive(TreeNode node)
{
    if (node != null)
    {
        PostOrderRecursive(node.Left);
        PostOrderRecursive(node.Right);
        Console.WriteLine(node.Val);
    }
}
```

## 🔁 Iterative

### **Python**

```python
def post_order_iterative(root):
    if not root:
        return
    stack1 = [root]
    stack2 = []
    while stack1:
        node = stack1.pop()
        stack2.append(node)
        if node.left:
            stack1.append(node.left)
        if node.right:
            stack1.append(node.right)
    while stack2:
        print(stack2.pop().val)
```

### **C#**

```csharp
void PostOrderIterative(TreeNode root)
{
    if (root == null) return;

    Stack<TreeNode> stack1 = new Stack<TreeNode>();
    Stack<TreeNode> stack2 = new Stack<TreeNode>();

    stack1.Push(root);
    while (stack1.Count > 0)
    {
        TreeNode node = stack1.Pop();
        stack2.Push(node);

        if (node.Left != null) stack1.Push(node.Left);
        if (node.Right != null) stack1.Push(node.Right);
    }

    while (stack2.Count > 0)
    {
        Console.WriteLine(stack2.Pop().Val);
    }
}
```

---

# 🌳 Binary Tree Node Definitions

## **Python**

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

## **C#**

```csharp
public class TreeNode
{
    public int Val;
    public TreeNode Left;
    public TreeNode Right;

    public TreeNode(int val = 0, TreeNode left = null, TreeNode right = null)
    {
        Val = val;
        Left = left;
        Right = right;
    }
}
```

---

Let me know if you'd like **unit tests**, **visual diagrams**, or **level-order traversal** (BFS) too!