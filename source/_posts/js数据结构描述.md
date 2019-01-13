---
title: js数据结构描述
date: 2019-01-13 20:00:00
tags: 数据结构
---

来自于《数据结构与算法JavaScript描述》读书整理，对前端工程师该书是比较好的数据结构与算法入门书。
<!-- more -->

# 数据结构

这部分介绍了以下数据结构，以及其在JavaScript中的实现方式和简单应用：数组，列表，栈，队列，链表，字典，散列，集合，二叉树和二叉查找树，图。

## 列表
列表。列表是一组**有序**的数据，每个列表中的数据项称为元素。列表的抽象数据类型定义：

方法 | 定义
--- | ---
length | 返回列表的元素个数
append | 在列表的末尾添加新元素
find | 在列表中查找元素
remove | 从列表中删除元素
clear | 清空列表中的所有元素
toString | 返回列表的字符串形式
insert | 在现有元素后面插入新元素
currPos | 返回列表的当前位置
front | 将列表的当前位置移动到第一个元素
end | 将列表的当前位置移动到最后一个元素
prev | 将列表的当前位置前移一位
next | 将列表的当前位置后移一位
hasPrev | 判断是否有前一位
hasNext | 判断是否有后一位
moveTo | 将当前位置移动到指定位置

<details>
    <summary>查看实现代码</summary>

    ```js
    class List {
        constructor() {
            this._dataStore = [];
            this._listSize = 0;
            this._pos = 0;
        }
        length() {
            return this._listSize;
        }
        append(el) {
            this._dataStore[this._listSize++] = el;
        }
        find(el) {
            return this._dataStore.findIndex(item => item == el);
        }
        remove(el) {
            const foundAt = this.find(el);
            if (foundAt > -1) {
                this._dataStore.splice(foundAt, 1);
                --this._listSize;
            }
        }
        clear() {
            this._dataStore = [];
            this._pos = this._listSize = 0;
        }
        toString() {
            return this._dataStore.join();
        }
        insert(el, after) {
            const foundAt = this.find(after);
            if (foundAt < 0) return false;
            this._dataStore.splice(foundAt, 0, el);
            this._listSize++;
            return true;
        }
        currPos() {
            return this._pos;
        }
        front() {
            this._pos = 0;
            return this.currPos();
        }
        end() {
            this._pos = this._listSize - 1;
            return this.currPos();
        }
        hasNext() {
            return this._pos < this._listSize - 1;
        }
        hasPrev() {
            return this._pos > 0;
        }
        prev() {
            if(this.hasPrev()) --this._pos;
            return this.currPos();
        }
        next() {
            if(this.hasNext()) ++this._pos;
            return this.currPos();
        }
        moveTo(pos) {
            if (0 <= pos && pos < this._listSize) {
                this._pos = pos;
                return this.currPos();
            } else {
                return -1;
            }

        }
    }
    ```
</details>

## 栈
栈。栈是一种特殊的列表，栈内的元素只能通过列表的一端访问，这一段称为栈顶。栈被称为一种**后入先出**的数据结构。

方法 | 定义
--- | ---
length | 返回栈内元素个数
push | 元素入栈
pop | 元素出栈
peek | 返回栈顶元素
empty | 返回栈内是否为空
clear | 清空栈内元素

<details>
    <summary>查看实现代码</summary>

    ```js
    class Stack {
        constructor() {
            this._dataStroe = [];
            this._top = 0;
        }
        length() {
            return this._top;
        }
        push(el) {
            this._dataStroe[this._top++] = el;
        }
        pop() {
            --this._top;
            return this._dataStroe.pop();
        }
        peek() {
            return this._dataStroe[this._top - 1];
        }
        clear() {
            this._dataStroe = [];
            this._top = 0;
        }
    }
    ```
</details>

## 队列
队列。队列是一种列表，不同的是队列只能在队尾插入元素，在对首删除元素，队列用于存储按顺序排列的数据，先进先出。在列表的实际使用中有时还会遇到需要插队的情况，这种数据结构叫做优先队列，出队伍的时候优先权最高的元素先出。

方法 | 定义
--- | ---
length | 返回队列元素的个数
toString | 显示队列内所有元素
enqueue | 入队
dequeue | 出队
front | 读取对首元素
end | 读取对尾元素
empty | 判断队伍是否为空

<details>
    <summary>查看实现代码</summary>

    ```js
    class Queue {
        constructor() {
            this._dataStore = [];
        }
        length() {
            return this._dataStore.length;
        }
        toString() {
            return this._dataStore.join();
        }
        enqueue(el) {
            this._dataStore.push(el);
        }
        dequeue() {
            return this._dataStore.shift();
        }
        front() {
            return this._dataStore[0];
        }
        end() {
            return this._dataStore[this._dataStore.length - 1];
        }
        empty() {
            return !this._dataStore.length;
        }
        toString() {
            return this._dataStore.join();
        }
    }
    ```
</details>

## 链表
链表。链表是由一组节点组成，每个节点都使用一个对象的引用指向它的后继，指向另一个节点的引用叫做链。需要拿链表和数组做一个对比，在很多编程语言中数组有很大的局限，数组被实现成的长度是固定的，这样当数组被填满后再舔新元素会很麻烦，而且操作数组也不像js一样有能直接使用的api。在js中数组的最大问题是被实现成了对象，相对其他编程语言，js的数组效率更低。当你觉得js中的数组在实际中使用很慢的时候，就可以考虑使用链表，除了对数据的随机访问，链表几乎可以使用在任何可以使用一维数组的情况中。

方法 | 定义
--- | ---
head | 头节点
find | 寻找节点
findPrev | 寻找上一个节点
insert | 插入节点
remove | 删除节点
display | 显示链表中的元素

<details>
    <summary>查看实现代码</summary>

    ```js
    function Node(el) {
        this.element = el;
        this.next = null;
    }

    class LList {
        constructor() {
            this._head = new Node('head');
        }
        find(item) {
            let currentNode = this._head;
            while(currentNode.element != item) {
                currentNode = currentNode.next;
            }
            return currentNode;
        }
        insert(newEl, item) {
            const newNode = new Node(newEl);
            const currentNode = this.find(item);
            newNode.next = currentNode.next;
            currentNode.next = newNode;
        }
        findPrev(item) {
            let currentNode = this._head;
            while(!(currentNode.next == null) && (currentNode.next.element != item)) {
                currentNode = currentNode.next;
            }
            return currentNode;
        }
        remove(item) {
            const prevNode = this.findPrev(item);
            if (!(prevNode.next == null)) {
                prevNode.next = prevNode.next.next;
            }
        }
        display() {
            let currentNode = this._head;
            const arr = [];
            arr.push(currentNode.element);
            while(currentNode.next != null) {
                currentNode = currentNode.next;
                arr.push(currentNode.element);
            }
            return arr;
        }
    }
    ```
</details>

## 字典
字典。字典是一种以键-值对形式存储数据的数据结构

方法 | 定义
--- | ---
add | 增加字典中的元素
find | 寻找字典中的元素
remove | 移除字典中的元素
showAll | 展示字典中的所有元素

<details>
    <summary>查看实现代码</summary>

    ```js
    class Dictionary {
        constructor() {
            this._dataStore = [];
        }
        add(key, value) {
            this._dataStore[key] = value;
        }
        find(key) {
            return this._dataStore[key];
        }
        remove(key) {
            delete this._dataStore[key];
        }
        showAll() {
            return this._dataStore;
        }
    }
    ```
</details>

## 散列表
散列。散列是一种常用的数据存储技术，散列后的数据可以快速地插入或者使用。散列使用的数据结构叫做散列表。在js中用数组来实现散列表，数组的长度是预先设定的，理想状况下散列函数会将每个键值映射成一个唯一的数组索引，并且将键均匀地映射到数组中。但是即便散列函数很高效，还是会存在将两个键值映射成同一个值得可能，这种现象叫做碰撞。实现一个散列表首先要确定的是数组的长度和散列函数。

## 集合
集合是一种包含不同元素的数据结构。集合中的元素称为成员，集合中两个最重要的特性是： 首先，集合中的成员是无序的；其次，集合中不允许相同的成员存在。js中可以通过Set对象实现集合

## 二叉查找树
树。树是一种非线性的数据结构，以分层形式存储数据，数被用来存储具有层级关系的数据。一棵树中最上面的节点称为**根节点**，如果一个节点下面连接多个节点，那么这个节点称为**父节点**，它下面的节点叫做**子节点**，没有任何子节点的节点称为**叶子节点**。树的层数叫做数的**深度**，从一个节点到另一个节点的一组边叫做**路径**，以某种方式访问树中所有的节点叫做树的**遍历**。**二叉树**是一种特殊的树，它的子节点树不超过两个，二叉查找树是一种特殊的二叉树，相对较小的值保存在左节点中，较大的值保存在右节点中。

方法 | 定义
--- | ---
insert | 给二叉查找树增加节点
find | 寻找节点
inOrder | 给返回排序后的节点
insert | 插入节点
getMin | 获取最小节点
getMax | 获取最大节点

<details>
    <summary>查看实现代码</summary>

    ```js
    function Node (data, left, right) {
        this.data  = data;
        this.left = left;
        this.right = right;
        this.show = () => this.data;
    }

    class BST {
        constructor() {
            this._root = null;
        }
        insert(data) {
            const n = new Node(data, null, null);
            if (this._root == null) {
                this._root = n;
            } else {
                let current = this._root;
                let parent;
                while(true) {
                    parent = current;
                    if (data < current.data) {
                        current = current.left;
                        if (current == null) {
                            parent.left = n;
                            break;
                        }
                    } else {
                        current = current.right;
                        if (current == null) {
                            parent.right = n;
                            break;
                        }
                    }
                }
            }
        }
        find(data) {
            let current = this._root;
            while(current != null) {
                if (current.data == data) {
                    return current;
                } else if (data < current.data) {
                    current = current.left;
                } else {
                    current = current.right;
                }
            }
            return null;
        }
        inOrder (node) {
            if (node != null) {
                this.inOrder(node.left);
                console.log(node.show());
                this.inOrder(node.right);
            }
        }
        getMin() {
            let current = this._root;
            while(!(current.left == null)) {
                current = current.left;
            }
            return current.data;
        }
        getMax() {
            let current = this._root;
            while(!(current.right == null)) {
                current = current.right;
            }
            return current.data;
        }
    }
    ```
</details>

## 图
图。图是由边的集合及顶点的集合组成。顶点有权重也称为成本，边由顶点对定义，如果边有方向，图就叫有向图，反之叫做无向图。图中一系列的顶点构成路径，路径中所有的顶点都由边连接。路径的长度用路径中第一个顶点到最后一个顶点之间边的数量表示。由指向自身的顶点组成的路径称为环，环的长度为0。圈是至少有一条边的路径，且路径的第一个顶点和最后一个顶点相同。只要是没有重复边或重复顶点的圈，就是一个简单圈，反之称为平凡圈。如果两个顶点之间有连接，那么这两个顶点就是强连通的。

