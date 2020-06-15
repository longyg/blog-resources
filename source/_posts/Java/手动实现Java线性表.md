---
title: 手动实现Java线性表
cover: false
top: false
date: 2019-11-10 16:36:00
group: java
permalink: java-list-impl
categories: Java后端
tags:
	- Java
	- 线性表
keywords:
	- Java实现线性表
	- 多种数据结构实现线性表
summary: 本文介绍线性表通用的基本功能和接口定义，并用Java语言采用多种底层数据结构从零实现线性表。
---



### 一、 线性表接口定义

一般情况下，一个线性表需要具有如下基本功能：

-   添加或插入元素
-   删除元素
-   查找元素
-   返回线性表中元素个数
-   判断线性表是否为空
-   清空线性表

因此，我们首先定义如下接口：
```java
package longyg.javabasic.collection;
 
public interface List<E> {
    /**
     * 往线性表中添加元素
     * @param element
     */
    void add(E element);
 
    /**
     * 在线性表的指定索引处插入元素
     * @param element
     * @param index
     */
    void insert(E element, int index);
 
    /**
     * 从线性表中删除元素
     * @param element
     * @return 返回删除的元素
     */
    E remove(E element);
 
    /**
     * 删除指定索引处的元素
     * @param index
     * @return 返回删除的元素
     */
    E delete(int index);
 
    /**
     * 从线性表种查找元素
     * @param element
     * @return 返回元素在线性表中的索引
     */
    int locate(E element);
 
    /**
     * 获取指定索引处的元素
     * @param index
     * @return 返回指定索引处的元素
     */
    E get(int index);
 
    /**
     * 获取线性表中的元素个数
     * @return
     */
    int size();
 
    /**
     * 判断线性表是否为空
     * @return
     */
    boolean isEmpty();
 
    /**
     * 清空线性表
     */
    void clear();
}
```

### 二、线性表实现

#### 实现一：基于数组的顺序存储结构

最常见的线性表实现是基于数组的顺序存储结构。
```java
package longyg.javabasic.collection;
 
import java.util.Arrays;
 
/**
 * 基于数组的顺序存储结构实现的线性表
 * @param <E>
 */
public class SequenceList<E> implements List<E> {
    private int capacity;
 
    private final static int DEFAULT_SIZE = 16;
 
    private Object[] data;
 
    private int size = 0;
 
    public SequenceList() {
        capacity = DEFAULT_SIZE;
        data = new Object[capacity];
    }
 
    private void ensureCapacity(int minCapacity) {
        if (minCapacity > capacity) {
            while (minCapacity > capacity) {
                capacity <<= 1;
            }
            data = Arrays.copyOf(data, capacity);
//            Object[] oldData = data;
//            data = new Object[capacity];
//            for (int i = 0; i < size; i++) {
//                data[i] = oldData[i];
//            }
        }
    }
 
    public void add(E element) {
        insert(element, size);
    }
 
    @Override
    public void insert(E element, int index) {
        if (index < 0 || index > size) {
            throw new IndexOutOfBoundsException("input index is out of array bounds");
        }
 
        ensureCapacity(size + 1);
        System.arraycopy(data, index, data, index + 1, size - index);
        data[index] = element;
        size++;
    }
 
    public E remove(E element) {
        int index = indexFor(element);
        if (index == -1) {
            return null;
        }
        return delete(index);
    }
 
    @Override
    public E delete(int index) {
        if (index < 0 || index > size - 1) {
            throw new IndexOutOfBoundsException("input index is out of array bounds");
        }
 
        E oldValue = (E) data[index];
 
        int numMoved = size - index - 1;
        if (numMoved > 0) {
            System.arraycopy(data, index + 1, data, index, numMoved);
        }
        data[--size] = null;
        return oldValue;
    }
 
    @Override
    public int locate(E element) {
        for (int i = 0; i < size; i++) {
            if (data[i].equals(element)) {
                return i;
            }
        }
        return -1;
    }
 
    private int indexFor(E element) {
        for (int i = 0; i < size; i++) {
            if (data[i].equals(element)) {
                return i;
            }
        }
        return -1;
    }
 
    public E get(int index) {
        if (index < 0 || index > size - 1) {
            throw new IndexOutOfBoundsException("input index is out of array bounds");
        }
        return (E) data[index];
    }
 
    public int size() {
        return size;
    }
 
    public boolean isEmpty() {
        return size == 0;
    }
 
    public void clear() {
//        for (int i = 0; i < size; i++) {
//            data[i] = null;
//        }
        Arrays.fill(data, null);
        size = 0;
    }
 
    @Override
    public String toString() {
        if (isEmpty()) {
            return "[]";
        }
        StringBuilder sb = new StringBuilder("[");
        for (int i = 0; i < size; i++) {
            sb.append(data[i]).append(",");
        }
        sb.deleteCharAt(sb.length() - 1);
        sb.append("]");
        return sb.toString();
    }
}
```

测试代码：
```java
    public static void main(String[] args) {
        SequenceList<String> list = new SequenceList<>();
        list.add("long");
        list.add("yong");
        list.add("gang");
        list.add("ni");
        list.add("hao");
        list.insert("good", 2);
 
        System.out.println(list);
        assert list.get(3).equals("gang");
        assert !list.isEmpty();
        assert list.size() == 6;
        assert list.locate("yong") == 1;
 
        assert list.remove("test") == null;
        assert list.remove("gang").equals("gang");
        assert list.delete(2).equals("good");
        System.out.println(list);
        assert !list.isEmpty();
        assert list.size() == 4;
 
        list.clear();
        assert list.remove("hehe") == null;
        assert list.locate("hehe") == -1;
        System.out.println(list);
        assert list.isEmpty();
        assert list.size() == 0;
 
        list.add("test");
        System.out.println(list);
        assert list.locate("test") == 0;
        assert !list.isEmpty();
        assert list.size() == 1;
    }
```

#### 实现二：基于单链表的链式存储结构

我们也可以基于单向链表的链式存储结构来实现线性表。让每个元素都保留指向下一个元素的引用，从而构成一个链表。
```java
package longyg.javabasic.collection;
 
/**
 * 基于单链表的链式存储结构实现的线性表
 * @param <E>
 */
public class LinkedSeqList<E> implements List<E> {
    private Node first;
    private Node last;
 
    private int size = 0;
 
    private class Node {
        E data;
        Node next;
 
        public Node() {}
 
        public Node(E data, Node next) {
            this.data = data;
            this.next = next;
        }
    }
 
    public LinkedSeqList() {
        first = null;
        last = null;
    }
 
    // 尾插法
    public void add(E element) {
        if (first == null) {
            first = new Node(element, null);
            last = first;
        } else {
            Node newNode = new Node(element, null);
            last.next = newNode;
            last = newNode;
        }
        size++;
    }
 
    @Override
    public void insert(E element, int index) {
        if (index < 0 || index > size) {
            throw new IndexOutOfBoundsException("input index is out of array bounds");
        }
        if (index == 0) {
            Node newNode = new Node(element, first);
            first = newNode;
            size++;
        } else {
            Node preNode = getNodeByIndex(index - 1);
            if (null != preNode) {
                Node current = preNode.next;
                Node newNode = new Node(element, current);
                preNode.next = newNode;
                size++;
            }
        }
    }
 
    public E remove(E element) {
        if (isEmpty()) return null;
 
        Node toDel = null;
        if (first.data.equals(element)) {
            toDel = first;
            first = first.next;
        } else {
            Node preNode = first;
            Node curNode = preNode.next;
            for (int i = 0; i < size; i++) {
                if (curNode == null) break;
                if (curNode.data.equals(element)) {
                    toDel = curNode;
                    preNode.next = toDel.next;
                    toDel.next = null;
                    break;
                } else {
                    preNode = curNode;
                    curNode = preNode.next;
                }
            }
        }
        if (toDel != null) {
            size--;
            return toDel.data;
        }
        return null;
    }
 
    @Override
    public E delete(int index) {
        if (index < 0 || index > size -1) {
            throw new IndexOutOfBoundsException("input index is out of bounds");
        }
 
        Node toDel = null;
        if (index == 0) {
            toDel = first;
            first = first.next;
        } else {
            Node preNode = getNodeByIndex(index - 1);
            toDel = preNode.next;
            preNode.next = toDel.next;
            toDel.next = null;
        }
        size--;
        return toDel.data;
    }
 
    @Override
    public int locate(E element) {
        Node curr = first;
        for (int i = 0; i < size; i++) {
            if (curr == null) break;
            if (curr.data.equals(element)) {
                return i;
            } else {
                curr = curr.next;
            }
        }
        return -1;
    }
 
    public E get(int index) {
        Node node = getNodeByIndex(index);
        if (node != null) {
            return node.data;
        }
        return null;
    }
 
    private Node getNodeByIndex(int index) {
        if (index < 0 || index > size -1) {
            throw new IndexOutOfBoundsException("input index is out of bounds");
        }
        Node currNode = first;
        for (int i = 0; i < size; i++) {
            if (currNode == null) break;
            if (i == index) {
                return currNode;
            } else {
                currNode = currNode.next;
            }
        }
        return null;
    }
 
    public int size() {
        return size;
    }
 
    public boolean isEmpty() {
        return size == 0;
    }
 
    public void clear() {
        first = null;
        last = null;
        size = 0;
    }
 
    @Override
    public String toString() {
        if (isEmpty()) {
            return "[]";
        }
        StringBuilder sb = new StringBuilder("[");
        Node curr = first;
        while (curr != null) {
            sb.append(curr.data).append(",");
            curr = curr.next;
        }
        sb.deleteCharAt(sb.length() - 1);
        sb.append("]");
        return sb.toString();
    }
}
```

#### 实现三：基于双向链表的链式存储结构

我们也可以基于双向链表实现线性表。每个元素不仅保留指向下一个元素的引用，也保留了指向前一个元素的引用，从而构成一个双向的链表。
```java
package longyg.javabasic.collection;
 
/**
 * 基于双向链表的链式存储结构实现的线性表
 * @param <E>
 */
public class DuLinkedList<E> implements List<E> {
    private Node<E> header;
    private Node<E> tail;
    private int size = 0;
 
    private class Node<E> {
        private E data;
        private Node<E> pre;
        private Node<E> next;
 
        public Node(E data, Node<E> pre, Node<E> next) {
            this.data = data;
            this.pre = pre;
            this.next = next;
        }
    }
 
    public DuLinkedList() {
        header = null;
        tail = null;
    }
 
    // 尾插法
    public void add(E element) {
        Node<E> l = tail;
        Node<E> newNode = new Node(element, l, null);
        tail = newNode;
        if (l == null) {
            header = newNode;
        } else {
            l.next = newNode;
        }
        size++;
//        if (header == null) {
//            header = new Node(element, null, null);
//            tail = header;
//        } else {
//            Node newNode = new Node(element, tail, null);
//            tail.next = newNode;
//            tail = newNode;
//        }
//        size++;
    }
 
    @Override
    public void insert(E element, int index) {
        if (index < 0 || index > size) {
            throw new IndexOutOfBoundsException("input index is out of array bounds");
        }
 
        if (index == 0) {
            Node newNode = new Node(element, null, header);
            header = newNode;
            if (tail == null) {
                tail = header;
            }
        } else {
            Node preNode = getNodeByIndex(index - 1);
            Node next = preNode.next;
            Node newNode = new Node(element, preNode, next);
            preNode.next = newNode;
            next.pre = newNode;
        }
        size++;
    }
 
    public E remove(E element) {
        if (isEmpty()) return null;
 
        Node<E> toDel = null;
        if (header.data.equals(element)) {
            toDel = header;
            header = header.next;
        } else {
            Node preNode = header;
            Node curNode = preNode.next;
            for (int i = 0; i < size; i++) {
                if (curNode == null) break;
                if (curNode.data.equals(element)) {
                    toDel = curNode;
                    preNode.next = toDel.next;
                    toDel.next.pre = preNode;
                    toDel.next = null;
                    toDel.pre = null;
                    break;
                } else {
                    preNode = curNode;
                    curNode = preNode.next;
                }
            }
        }
        if (toDel != null) {
            size--;
            return toDel.data;
        }
        return null;
    }
 
    @Override
    public E delete(int index) {
        if (index < 0 || index > size -1) {
            throw new IndexOutOfBoundsException("input index is out of bounds");
        }
 
        Node<E> toDel = null;
        if (index == 0) {
            toDel = header;
            header = header.next;
            header.pre = null;
        } else {
            Node preNode = getNodeByIndex(index - 1);
            toDel = preNode.next;
            preNode.next = toDel.next;
            if (toDel.next != null) {
                toDel.next.pre = preNode;
            }
            toDel.next = null;
            toDel.pre = null;
        }
        size--;
        return toDel.data;
    }
 
    @Override
    public int locate(E element) {
        Node curr = header;
        for (int i = 0; i < size; i++) {
            if (curr == null) break;
            if (curr.data.equals(element)) {
                return i;
            } else {
                curr = curr.next;
            }
        }
        return -1;
    }
 
    private Node getNodeByIndex(int index) {
        if (index < 0 || index > size -1) {
            throw new IndexOutOfBoundsException("input index is out of bounds");
        }
 
        if (index <= size / 2) {
            // 从头节点正向搜索
            Node cur = header;
            for (int i = 0; i <= size / 2; i++) {
                if (cur == null) break;
                if (i == index) {
                    return cur;
                } else {
                    cur = cur.next;
                }
            }
        } else {
            // 从尾节点反向搜索
            Node cur = tail;
            for (int i = size - 1; i > size / 2; i--) {
                if (cur == null) break;
                if (i == index) {
                    return cur;
                } else {
                    cur = cur.pre;
                }
            }
        }
        return null;
    }
 
    public E get(int index) {
        Node<E> node = getNodeByIndex(index);
        if (node != null) {
            return node.data;
        }
        return null;
    }
 
    public boolean isEmpty() {
        return size == 0;
    }
 
    public int size() {
        return size;
    }
 
    public void clear() {
        header = null;
        tail = null;
        size = 0;
    }
 
    @Override
    public String toString() {
        if (isEmpty()) {
            return "[]";
        }
        StringBuilder sb = new StringBuilder("[");
        Node curr = header;
        while (curr != null) {
            sb.append(curr.data).append(",");
            curr = curr.next;
        }
        sb.deleteCharAt(sb.length() - 1);
        sb.append("]");
        return sb.toString();
    }
}
```