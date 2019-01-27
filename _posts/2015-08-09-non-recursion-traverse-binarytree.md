---
title: 三种遍历二叉树的非递归方式
tags: [algorithm]
---

传统遍历二叉树的方式用递归实现起来非常简单，因为递归程序内部本质上是用栈来实现的，所以非递归遍历二叉树可以采用栈去模拟实现。

先序遍历（根左右）：

```java
private static void preOrderTraverse(Node root) {
    Stack<Node> stack = new Stack<>();
    if (root != null) {
        Node p = root;
        while (p != null || !stack.empty()) {
            if (p != null) { // 遍历左子树
                System.out.print(p.val + " "); // 入栈时输出该节点
                stack.push(p);
                p = p.left;
            } else { // 遍历右子树并退栈
                p = stack.pop().right;
            }
        }
    }
}
```

中序遍历（左根右）：

```java
private static void inOrderTraverse(Node root) {
    Stack<Node> stack = new Stack<>();
    if (root != null) {
        Node p = root;
        while (p != null || !stack.empty()) {
            if (p != null) { // 遍历左子树
                stack.push(p);
                p = p.left;
            } else { // 遍历右子树并退栈
                System.out.print(stack.peek().val + " "); // 出栈时输出该节点
                p = stack.pop().right;
            }
        }
    }
}
```

后序遍历（左右根）：

```java
private static void postOrderTraverse(Node root) {
    Stack<Node> stack = new Stack<>();
    int[] tag = new int[8]; // 存储节点的右孩子访问标记（0表示未访问，1表示已访问，size应大于等于节点总数）
    if (root != null) {
        Node p = root;
        while (p != null || !stack.empty()) {
            if (p != null) { // 遍历左子树
                stack.push(p);
                tag[stack.size() - 1] = 0;
                p = p.left;
            } else if (tag[stack.size() - 1] == 0) { // 遍历右子树
                p = stack.peek().right;
                tag[stack.size() - 1] = 1;
            } else { // 输出该节点并退栈
                System.out.print(stack.pop().val + " ");
            }
        }
    }
}
```

