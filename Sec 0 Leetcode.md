# Blind No.14: 269. Alien Dictionary

**Difficulty:** Hard
**Topics:** Graph, Topological Sort
**Companies:**

**Description:**
There is a new alien language that uses the English alphabet. However, the order of the letters is unknown to you.
You are given a list of strings `words` from the alien language's dictionary. Now it is claimed that the strings in `words` are sorted lexicographically by the rules of this new language.

* If this claim is incorrect, and the given arrangement of strings in `words` cannot correspond to any order of letters, return `""`.
* Otherwise, return a string of the unique letters in the new alien language sorted in lexicographically increasing order by the new language's rules. If there are multiple solutions, return any of them.

---

## Solution

```java
class Solution {
    /**
     * 返回一个合法的外星字母顺序，如果不存在则返回空串 ""
     */
    public String alienOrder(String[] words) {
        int N = words.length;
        // 记录每个字符是否在 words 中出现过
        boolean[] present = new boolean[26];
        // 邻接表：graph[u] 存储 u -> v 的所有边
        List<Set<Integer>> graph = new ArrayList<>();
        // 入度数组：inDegree[v] 表示有多少字符先于 v
        int[] inDegree = new int[26];
        
        // 1. 初始化图结构
        for (int i = 0; i < 26; i++) {
            graph.add(new HashSet<>());
        }
        // 2. 扫描所有单词，标记出现过的字符
        for (String w : words) {
            for (char c : w.toCharArray()) {
                present[c - 'a'] = true;
            }
        }
        
        // 3. 构建约束边：根据相邻单词第一个不同字符对
        for (int i = 0; i < N - 1; i++) {
            String cur = words[i], next = words[i + 1];
            int len = Math.min(cur.length(), next.length());
            // 特殊情况：如果 cur 比 next 长且 next 是 cur 的前缀，则无效
            if (cur.length() > next.length() && cur.startsWith(next)) {
                return "";
            }
            for (int j = 0; j < len; j++) {
                int u = cur.charAt(j) - 'a';
                int v = next.charAt(j) - 'a';
                if (u != v) {
                    if (!graph.get(u).contains(v)) {
                        graph.get(u).add(v);
                        inDegree[v]++;
                    }
                    break;
                }
            }
        }
        
        // 4. 拓扑排序初始化：入度为 0 且出现过的字符入队
        Deque<Integer> queue = new LinkedList<>();
        for (int i = 0; i < 26; i++) {
            if (present[i] && inDegree[i] == 0) {
                queue.addLast(i);
            }
        }
        
        // 5. 依次出队，构建结果字符串
        StringBuilder sb = new StringBuilder();
        while (!queue.isEmpty()) {
            int u = queue.removeFirst();
            sb.append((char) (u + 'a'));
            for (int v : graph.get(u)) {
                inDegree[v]--;
                if (inDegree[v] == 0) {
                    queue.addLast(v);
                }
            }
        }
        
        // 6. 检查环路：若有剩余入度，则无效
        for (int i = 0; i < 26; i++) {
            if (present[i] && inDegree[i] > 0) {
                return "";
            }
        }
        return sb.toString();
    }
}
```

**Thought:** Topological Sort

---

# Blind No.15: 271. Encode and Decode Strings

**Difficulty:** Medium
**Topics:** Array Design, String Parsing
**Companies:**

**Description:**
Design an algorithm to encode a list of strings to a single string. The encoded string is then sent over the network and is decoded back to the original list of strings.

---

## Solution

```java
public class Codec {
    /**
     * 将字符串列表编码为一个单一字符串。
     * 采用「长度 + 分隔符 # + 原字符串」方式，
     * 解码时能够准确截取每个原始字符串。
     */
    public String encode(List<String> strs) {
        StringBuilder sb = new StringBuilder();
        for (String s : strs) {
            sb.append(s.length());  // 写入长度
            sb.append('#');         // 写入分隔符
            sb.append(s);           // 写入内容
        }
        return sb.toString();
    }

    /**
     * 将编码后的字符串还原成原始字符串列表。
     * 读取长度前缀，跳过分隔符，截取对应长度子串。
     */
    public List<String> decode(String s) {
        List<String> res = new ArrayList<>();
        int i = 0;
        while (i < s.length()) {
            int j = i;
            // 找到下一个分隔符的位置
            while (s.charAt(j) != '#') j++;
            int len = Integer.parseInt(s.substring(i, j));
            // 截取长度为 len 的原始字符串
            String str = s.substring(j + 1, j + 1 + len);
            res.add(str);
            i = j + 1 + len;
        }
        return res;
    }
}
```

**Thought:** Array Design, for loop

---

# Blind No.16: 19. Remove Nth Node From End of List

**Difficulty:** Medium
**Topics:** Linked List, Two Pointers
**Companies:**

**Description:**
Given the head of a linked list, remove the nᵗʰ node from the end of the list and return its head.

---

## Solution

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    /**
     * 删除链表倒数第 n 个节点，返回新的头结点
     */
    public ListNode removeNthFromEnd(ListNode head, int n) {
        // 1. 创建哑结点，简化删除头节点的情况
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        
        // 2. 初始化双指针，均指向 dummy
        ListNode first = dummy;
        ListNode second = dummy;
        
        // 3. 先让 first 前进 n+1 步，保持与 second 相隔 n 个节点
        for (int i = 0; i < n + 1; i++) {
            first = first.next;
        }
        
        // 4. 同步移动，直到 first 到末尾，second 指向待删节点的前驱
        while (first != null) {
            first = first.next;
            second = second.next;
        }
        
        // 5. 跳过倒数第 n 个节点
        second.next = second.next.next;
        
        // 6. 返回实际头结点
        return dummy.next;
    }
}
```

**Thought:** Linked List, Two Pointers

