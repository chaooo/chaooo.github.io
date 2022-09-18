---
title: 「数据结构与算法」Trie 树 与 AC 自动机
date: 2022-07-06 20:25:00
tags: [算法, 数据结构]
categories: 数据结构
---

`Trie 树`，也叫“字典树”、“前缀树”。它是一种有序树形结构。是一种专门处理字符串匹配的数据结构，用来解决在一组字符串集合中快速查找某个字符串的问题。
`AC 自动机`以**`Trie 树`的结构**为基础，结合**`KMP`的思想**建立的，是一种用于解决多模式匹配问题的经典算法。
<!-- more -->

### 1. Trie 树
`Trie 树`的本质，就是利用字符串之间的公共前缀，将重复的前缀合并在一起。

举例说明一下。我们有`6`个字符串：`how，hi，her，hello，so，see`。构造出来`Trie 树`的结构就是下面这个图中的样子。

![](02_01.webp)

其中，根节点不包含任何信息。每个节点表示一个字符串中的字符，从根节点到红色节点的一条路径表示一个字符串（注意：红色节点表示一个单词的结尾，红色节点并不都是叶子节点）。

`Trie 树`构造的分解过程：

![](02_02.webp)
![](02_03.webp)

当我们在 `Trie 树`中查找一个字符串的时候，比如查找字符串“her”，从 `Trie 树`的根节点开始匹配。如图所示，绿色的路径就是在 `Trie 树`中匹配的路径。

![](02_04.webp)


#### 1.1 Trie 树 的实现
`Trie 树`是一个多叉树。假如这个字典只包括`26`个英文字母（暂且都定为小写），那么这个树的深度会由具体单词不一样而定。但是它的广度范围是可以提前确定好的。对于每个节点，广度最大为`26`。（因为每个节点的下一个字母（即后缀点）只可能是`26`个字母。）
每个节点存储为`26`个元素(`a-z`)的数组，下标`0-25`存储子节点数组的位置，子节点数组仍然为`26`个元素的数组。我们在数组中下标为`0`的位置，存储指向子节点`a`的指针，下标为`1`的位置存储指向子节点`b`的指针，以此类推，下标为`25`的位置，存储的是指向的子节点`z`的指针。如果某个字符的子节点不存在，我们就在对应的下标的位置存储`null`。

``` Java
class TrieNode {
  char data;
  TrieNode children[26];
}
```

当我们在 `Trie 树`中查找字符串的时候，我们就可以通过字符的`ASCII`码减去`a`的`ASCII`码，迅速找到匹配的子节点的指针。
比如，`d`的`ASCII`码减去`a`的`ASCII`码就是`3`，那子节点`d`的指针就存储在数组中下标为`3`的位置中。

``` Java
public class Trie {
  // 根节点存储无意义字符
  private TrieNode root = new TrieNode('/');
  // 往Trie树中插入一个字符串
  public void insert(char[] text) {
    TrieNode p = root;
    for (int i = 0; i < text.length; ++i) {
      int index = text[i] - 'a';
      if (p.children[index] == null) {
        TrieNode newNode = new TrieNode(text[i]);
        p.children[index] = newNode;
      }
      p = p.children[index];
    }
    p.isEndingChar = true;
  }
  // 在Trie树中查找一个字符串
  public boolean find(char[] pattern) {
    TrieNode p = root;
    for (int i = 0; i < pattern.length; ++i) {
      int index = pattern[i] - 'a';
      // 不存在pattern
      if (p.children[index] == null) {
        return false;
      }
      p = p.children[index];
    }
    // 不能完全匹配，只是前缀
    if (p.isEndingChar == false) return false;
    // 找到pattern
    else return true;
  }

  public class TrieNode {
    public char data;
    public TrieNode[] children = new TrieNode[26];
    public boolean isEndingChar = false;
    public TrieNode(char data) {
      this.data = data;
    }
  }
}
```

#### 1.2 Trie 树 的复杂度
构建`Trie 树`的过程，需要扫描所有的字符串，时间复杂度是 `O(n)`（`n` 表示所有字符串的长度和）。
但是一旦构建成功之后，后续的查询操作会非常高效，时间复杂度是 `O(k)`（`k`表示要查找的字符串的长度）。
所以`Trie 树`的查找时间复杂度`O(k)`是可以忽略不计的。

`Trie 树`用的是一种空间换时间的思路，用数组来存储一个节点的子节点的指针。如果字符串中包含从`a-z`这`26`个字符，那每个节点都要存储一个长度为`26`的数组，如果字符串中不仅包含小写字母，还包含大写字母、数字、甚至是中文，那需要的存储空间就更多了。所以，也就是说，在某些情况下，`Trie 树`不一定会节省存储空间。在重复的前缀并不多的情况下，`Trie 树`不但不能节省内存，还有可能会浪费更多的内存。
`Trie 树`尽管有可能很浪费内存，但是确实非常高效。那为了解决这个内存问题，我们可以稍微牺牲一点查询的效率，将每个节点中的数组换成其他数据结构，来存储一个节点的子节点指针。比如有序数组、跳表、散列表、红黑树等。

`Trie 树`可以应用于 搜索引擎的搜索关键词提示、输入法自动补全功能、IDE 代码编辑器自动补全功能、浏览器网址输入的自动补全功能等等。


### 2. AC 自动机
`AC 自动机`（Aho-Corasick Automaton）算法，是一种用于解决多模式匹配问题的经典算法。自动机是一个数学模型，自动机的结构就是一张**有向图**。通过将模式串预处理为确定有限状态自动机，扫描文本一遍就能结束。其复杂度为O(n)，即与模式串的数量和长度无关。
`AC 自动机`以**`Trie 树`的结构**为基础，结合**`KMP`的思想**建立的。
简单来说，建立一个`AC 自动机`有两个步骤：
1. 基础的`Trie 树`结构：将所有的模式串构成一棵`Trie 树`。
2. `KMP` 的思想：对`Trie 树`上所有的结点构造失配指针（相当于`KMP`中的失效函数`next`数组）。
然后就可以利用它进行**多模式匹配**了。

>- 单模式串匹配算法，是在一个模式串和一个主串之间进行匹配，也就是说，在一个主串中查找一个模式串。
>- 多模式串匹配算法，就是在多个模式串和一个主串之间做匹配，也就是说，**在一个主串中查找多个模式串**。

举个`AC自动机`的构建的例子：假设文本串是`shisherhis`，模式串有`4`个，分别为`{he,her,his,she}`，我们用模式串建一颗`Trie 树`。如下图：

![](02_05.png)

紫色虚线表示**失配指针**，失配指针的作用是在这个模式串失配后快速跳到下一个有可能成功匹配的模式串来匹配，即利用已知信息来加速匹配。如下图：

![](02_06.png)

其实，构建**失配指针**其实就是在找**最长的与当前失配模式串后缀(已匹配后缀)相同的与下一个模式串的前缀**，其实最长的话就可以保证每个有可能匹配的模式串都匹配到，假设当前节点模式串失配了，就跳最大的前缀，如果又失配，就又到最大前缀的最大前缀，直到没有为止，这样就可以保证全部考虑到。
画个图来理解：
![](02_07.png)

计算每个节点的失败指针这个过程看起来有些复杂。其实，如果我们把树中相同深度的节点放到同一层，那么某个节点的失败指针只有可能出现在它所在层的上层。
可以像`KMP 算法`那样，当我们要求某个节点的失败指针的时候，我们通过已经求得的、深度更小的那些节点的失败指针来推导。也就是说，我们可以逐层依次来求解每个节点的失败指针。所以，失败指针的构建过程，是一个按层遍历的过程（广度优先搜索(BFS)）。

> - `广度优先搜索（Breadth-First-Search）`，简称 `BFS`。直观地讲，它其实就是一种“地毯式”层层推进的搜索策略，即先查找离起始顶点最近的，然后是次近的，依次往外搜索。
> - `深度优先搜索（Depth-First-Search）`，简称 `DFS`。它会尽可能深地搜索分支，最直观的例子就是“走迷宫”，即走迷宫发现走不通的时候，你就回退到上一个岔路口，重新选择一条路继续走，直到最终找到出口。

### 3. AC 自动机的 Java 实现
``` Java
import java.util.*;
/**
 * AC 自动机的 Java 实现
 */
public class AC {
    /**
     * 本示例中的AC自动机只处理 英文类型的字符串；
     * ASCII码到目前为止共定义了128个字符；
     * 包括大小写字母、数字0-9、标点符号，以及在英文中使用的特殊控制字符。
     */
    private static final int ASCII = 128;
    /**
     * 根结点不存储任何字符信息
     */
    private final Node root;
    /**
     * 匹配结果集合，key表示模式串，value表示模式串在文本串出现的位置
     */
    private final HashMap<String, List<Integer>> result = new HashMap<>();

    /**
     * 内部静态类，用于表示AC自动机的每个结点
     */
    private static class Node{
        /** 该结点对应的字符 */
        public char data;
        /** 深度：根节点到这个节点所经历的路径（边数）*/
        public int depth = 0;
        /** ASCII == 128, 所以这里相当于128叉树 */
        public Node[] children = new Node[ASCII];
        /** 失配指针 */
        Node fail;
        /** 是否结尾字符 */
        public boolean isWord = false;
        /** 构造函数 */
        public Node() {}
        public Node(char data) {
            this.data = data;
        }
    }

    /**
     * 通过模式串构建 Trie树
     */
    private void buildTrieTree(String[] patterns){
        for (String pattern : patterns) {
            // 初始化匹配结果集合
            result.put(pattern, new LinkedList<Integer>());
            // 通过模式串构建 Trie树
            Node p = root;
            for (int i = 0; i < pattern.length(); ++i) {
                char ch = pattern.charAt(i);
                if (p.children[ch] == null) {
                    p.children[ch] = new Node(pattern.charAt(i));
                    p.children[ch].depth = p.depth + 1;
                }
                p = p.children[ch];
            }
            p.isWord = true;
        }
    }
    /**
     * 对 Trie树 上所有的结点构造失配指针
     */
    private void buildFailurePointer() {
        // LinkedList 实现了 Queue 接口，可作为队列使用(先进先出)
        Queue<Node> queue = new LinkedList<>();
        // 单独处理根结点下的第一层子结点
        for (Node x : root.children){
            if(x != null){
                // 根结点的所有孩子结点的fail都指向根结点
                x.fail = root;
                // 所有根结点的孩子结点入队列
                queue.add(x);
            }
        }
        while(!queue.isEmpty()){
            // 出队列结点的孩子结点的fail指向
            Node p = queue.remove();
            for (int i = 0; i < p.children.length; i++){
                if(p.children[i] != null){
                    // 孩子结点入队列
                    queue.add(p.children[i]);
                    // 从p.fail开始找起
                    Node f = p.fail;
                    while(true){
                        // 向上找到了根结点还没有找到(没有公共前缀)
                        if(f == null){
                            p.children[i].fail = root;
                            break;
                        }
                        // 有公共前缀
                        if(f.children[i] != null){
                            p.children[i].fail = f.children[i];
                            break;
                        } else{
                            // 继续向上寻找
                            f = f.fail;
                        }
                    }
                }
            }
        }
    }

    /**
     * 构造AC自动机
     * @param patterns 多模式串
     */
    public AC(String[] patterns) {
        root = new Node();
        buildTrieTree(patterns);
        buildFailurePointer();
    }

    /**
     * 匹配所有的模式串
     * @param text 文本串（主串）
     */
    public HashMap<String, List<Integer>> match(String text){
        Node p = root;
        int i = 0;
        while(i < text.length()){
            // 文本串中的字符
            char ch = text.charAt(i);
            // 在AC自动机中找文本串中的字符ch
            if(p.children[ch] != null){
                // 找到了，自动机进入下一状态
                p = p.children[ch];
                if(p.isWord){
                    // 模式串匹配成功，存入结果集result
                    String pattern = text.substring(i - p.depth + 1, i + 1);
                    result.get(pattern).add(i - p.depth + 1);
                }
                // 模式串的中间某部分字符可能正好包含另一个模式串
                if(p.fail != null && p.fail.isWord){
                    String pattern = text.substring(i - p.fail.depth + 1, i + 1);
                    result.get(pattern).add(i - p.fail.depth + 1);
                }
                // 文本串索引自增，指向下一个文本串中的字符
                i++;
            } else{
                // 没找到（p.children[ch] == null）,
                // 失配指针发挥作用的地
                p = p.fail;
                // 到根结点还未找到，说明文本串中以ch作为结束的字符片段不是任何目标字符串的前缀，
                // 状态机重置(从root开始重新匹配)，比较下一个字符
                if(p == null){
                    p = root;
                    i++;
                }
            }
        }
        // 文本串扫描完毕，返回结果集
        return result;
    }
}
```

AC自动机 测试
``` Java
@Test
void testAC() {
    String[] patterns = {"he","her","his","she"};
    String text = "shisherhis";
    // 构造AC自动机
    AC ac = new AC(patterns);
    // AC自动机匹配文本串
    HashMap<String, List<Integer>> result = ac.match(text);

    System.out.println("文本串“"+ text + "”的匹配结果：");
    for (Map.Entry<String, List<Integer>> entry : result.entrySet()){
        System.out.println(entry.getKey()+" : " + entry.getValue());
    }
}
```

输出结果：
```
文本串“shisherhis”的匹配结果：
she : [3]
his : [1, 7]
her : [4]
he : [4]
```
