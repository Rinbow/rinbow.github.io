---
title: 赫夫曼编码
tags: [algorithm]
---

### 赫夫曼树

赫夫曼树又叫做最优二叉树，特点是**带权路径最短**（某个节点的带权路径指的是该节点到根节点的路径长度乘以节点的权值）。

构造赫夫曼树的步骤如下：

1. 将所有节点看作只有一个根节点的二叉树，进行排序；
2. 选出两个根节点权值最小的树作为左右子树构造一个新的二叉树，新二叉树的根节点的权值为左右子树的权值之和；
3. 从所有节点中删除第 2 步中的两个树，并加入新构造的树；
4. 重复第 2 步和第 3 步，直到只剩下一个树。

赫夫曼树的特点：

1. 权值越大的节点，距离根节点越近。
2. 树中没有度为 1 的节点，这类树又叫做正则（严格）二叉树。

### 赫夫曼编码

赫夫曼树的一个重要应用就是赫夫曼编码，即要将传送的文字转换为二进制的字符串，要求转换后的字符串越短越好，而且可解码，因此引入了前缀编码的概念（前缀编码是指任一字符的编码都不是另一个字符编码的前缀），这保证了字符串在解码过程中不会产生歧义。

为了同时满足前缀编码和长度最短两个要求，引入了赫夫曼编码的概念，**赫夫曼编码就是长度最短的前缀编码**。

示例代码：

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class HuffmanEncode {

    public static void main(String[] args) {
        List<Node> nodes = new ArrayList<>();
        nodes.add(new Node("A", 0.4));
        nodes.add(new Node("B", 0.3));
        nodes.add(new Node("C", 0.1));
        nodes.add(new Node("D", 0.2));
        Node root = constructHuffmanTree(nodes);
        printTree(root, "");
    }

    /**
     * 构造赫夫曼树
     *
     * @param nodes
     * @return
     */
    private static Node constructHuffmanTree(List<Node> nodes) {
        while (nodes.size() > 1) {
            Collections.sort(nodes);
            Node left = nodes.get(0);
            Node right = nodes.get(1);
            Node parent = new Node(null, left.weight + right.weight);
            parent.left = left;
            parent.right = right;
            nodes.remove(left);
            nodes.remove(right);
            nodes.add(parent);
        }
        return nodes.get(0);
    }

    /**
     * 打印结果
     *
     * @param node
     * @param suffix
     */
    private static void printTree(Node node, String suffix) {
        if (node == null) return;
        else if (node.left == null && node.right == null) {
            System.out.println(node.data + ":" + suffix);
        } else {
            printTree(node.left, suffix + "0");
            printTree(node.right, suffix + "1");
        }
    }
}

/**
 * 赫夫曼树节点
 */
class Node implements Comparable<Node> {
    String data;
    double weight;

    Node left;
    Node right;

    public Node(String data, double weight) {
        this.data = data;
        this.weight = weight;
    }

    @Override
    public String toString() {
        return "data:" + this.data + ",weight:" + this.weight;
    }

    @Override
    public int compareTo(Node o) {
        if (this.weight < o.weight) {
            return -1;
        } else if (this.weight > o.weight) {
            return 1;
        }
        return 0;
    }
}
```

输出结果：

```
A:0
B:10
C:110
D:111
```

---

### 参考资料

- 数据结构高分笔记2015版