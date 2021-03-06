### 算法篇

#### 开篇

+ <a href="https://labuladong.github.io/algo/1/2/">参考链接</a>

#### 数据结构

+ 组成
  + 一切数据结构的底层都是由数组加链表构成的
  + 数组
    + 由于是紧凑连续存储,可以随机访问，通过索引快速找到对应元素，而且相对节约存储空间。但正因为连续存储，内存空间必须一次性分配够，所以说数组如果要扩容，需要重新分配一块更大的空间，再把数据全部复制过去，时间复杂度 O(N)；而且你如果想在数组中间进行插入和删除，每次必须搬移后面的所有数据以保持连续，时间复杂度 O(N)。
  + 链表
    + 因为元素不连续，而是靠指针指向下一个元素的位置，所以不存在数组的扩容问题；如果知道某一元素的前驱和后驱，操作指针即可删除该元素或者插入新元素，时间复杂度 O(1)。但是正因为存储空间不连续，你无法根据一个索引算出对应元素的地址，所以不能随机访问；而且由于每个元素必须存储指向前后元素位置的指针，会消耗相对更多的储存空间。

#### 数据结构基本操作

+ 概述

  + 数据结构的操作本质上有两种（遍历+访问），更具体一点：增删查改

+ 遍历+访问方式

  + 线性遍历以迭代为主
  + 非线性遍历以递归为主

+ 示例

  + 线性

    + 数组

      ```java
      void traverse(int[] arr) {
          for (int i = 0; i < arr.length; i++) {
              // 迭代访问 arr[i]
          }
      }
      ```

  + 非线性

    + 链表

      ```java
      /* 基本的单链表节点 */
      class ListNode {
          int val;
          ListNode next;
      }
      
      void traverse(ListNode head) {
          for (ListNode p = head; p != null; p = p.next) {
              // 迭代访问 p.val
          }
      }
      
      void traverse(ListNode head) {
          // 递归访问 head.val
          traverse(head.next);
      }
      ```

    + 二叉树

      ```java
      /* 基本的二叉树节点 */
      class TreeNode {
          int val;
          TreeNode left, right;
      }
      
      void traverse(TreeNode root) {
          traverse(root.left);
          traverse(root.right);
      }
      ```

    + N叉树

      ```java
      /* 基本的 N 叉树节点 */
      class TreeNode {
          int val;
          TreeNode[] children;
      }
      
      void traverse(TreeNode root) {
          for (TreeNode child : root.children)
              traverse(child);
      }
      ```

#### 数据结构中的算法

+ 概述
  + 数据结构中算法本质是穷举
  + 穷举的注意点
    + 如何穷举？即**无遗漏**地穷举所有可能解。
    + 如何聪明地穷举？即**避免所有冗余的计算**，消耗尽可能少的资源求出答案。
+ 数组单链表之类的算法
  + 双指针
  + 滑动窗口
  + 前缀和和与差分
+ 二叉树算法
  + 主要思路
    + 第一类是遍历一遍二叉树得出答案
    + 第二类是通过分解问题计算出答案
    + 这两类思路分别对应着 [回溯算法核心框架]和 [动态规划核心框架。

