---
title: Java-DataStructure
author: Jiny
date: 2020-11-28 14:30:00 +0800
categories: [Lang, Java]
tags: [java, basics]
toc: false
---

# Java DataStructure
___

## Stack & Queue

**ListNode**

```java
package base;

public class ListNode {
    public int data;
    public ListNode next;

    public ListNode(int input) {
        data = input;
        next = null;
    }

    @Override
    public String toString() {
        return String.valueOf(data);
    }
}
```

**LinkedList**
```java
package base;

public class LinkedList{

    private ListNode head;
    private ListNode tail;
    private int size;

    LinkedList(int input) {
        head = new ListNode(input);
        tail = head;
        size = 1;
    }

    void add(int data) {
        ListNode node = head;
        tail.next = new ListNode(data);
        tail = tail.next;
        size++;
    }

    void addTo(int data, int position) {
        ListNode node = head;
        ListNode addNode = new ListNode(data);

        if(position == 0) {
            addNode.next = node;
            head = addNode;
            return;
        }

        for(int i=0; i<position - 1; i++) {
            node = node.next;
        }

        if(node.next == null) {
            tail = addNode;
        }

        addNode.next = node.next;
        node.next = addNode;
        size++;
        return;
    }

    public boolean contains(ListNode head, ListNode checkNode) {
        ListNode node = head;
        while(node != null) {
            if(node.data == checkNode.data) {
                return true;
            }
            node = node.next;
        }
        return false;
    }

    public void remove() {
        removeAt(size-1);
    }

    public void removeAt(int position) {
        ListNode node = head;
        if(position == 0) {
            head = head.next;
        } else {
            for(int i=0; i<position-1; i++) {
                node = node.next;
            }

            if(node.next.next == null) {
                tail = node;
            }

            node.next = node.next.next;
        }
        size--;
    }

    int getHead() {
        return head.data;
    }

    int getTail() {
        return tail.data;
    }

    int getSize() {
        return this.size;
    }

}

```

**Stack(LinkedList)**

```java
package base;

public class Stack extends LinkedList {
    private int top;
    private int stackSize;

    Stack(int input) {
        super(input);
        top = super.getHead();
    }

    void push(int data) {
        super.add(data);
        top = super.getTail();
    }

    int peek() {
        return super.getTail();
    }

    int pop() {
        int result = super.getTail();
        super.remove();
        top = super.getTail();
        return result;
    }

    int size() {
        return super.getSize();
    }

}

```

**Stack(Array)**

```java
package base;

public class StackArr {
    private int top;
    private int size;
    private int arr[];

    StackArr(int input, int arrSize) {
        arr = new int[arrSize];
        size = 1;
        top = input;
    }

    public void push(int data) {
        arr[size] = data;
        size++;
    }

    public int pop() {
        int result = arr[size-1];
        size--;
        return result;
    }

    public int peek() {
        return arr[size-1];
    }
}
```

**Queue(LinkedList)**

```java
package base;

public class Queue extends LinkedList {
    private int head;
    private int stackSize;

    Queue(int input) {
        super(input);
        head = super.getHead();
    }

    void push(int data) {
        super.add(data);
        head = super.getHead();
    }

    int peek() {
        return super.getHead();
    }

    int pop() {
        int result = super.getHead();
        super.removeAt(0);
        head = super.getHead();
        return result;
    }

    int size() {
        return super.getSize();
    }
}
```

**Queue(Array)**

```java
package base;

public class QueueArr {
    private int head;
    private int size;
    private int start;
    private int arr[];

    QueueArr(int input, int arrSize) {
        arr = new int[arrSize];
        size = 1;
        start = arrSize;
        head = input;
    }

    public void push(int data) {
        arr[start-1] = data;
        start--;
        size++;
    }

    public int pop() {
        int result = arr[start-1];
        start++;
        size--;
        return result;
    }

    public int peek() {
        return arr[start-1];
    }
}

```
